# Design Doc #

## Requirements ##

> Implement a prototype job worker service that provides an API to run arbitrary Linux processes.

## Design ##

This project will have client app and server daemon app. Server app will open API and receive the commands from client side then run the command as the client wish.

Client side is a command line app for users run the commands through terminal. Users should have the ability of `start, stop, get status, and stream output of a job`.

### Server side ###

Server side application should listen the port `9090` for gRPC requests. 

#### runtime ####

This project will use [tokio 1.x](https://docs.rs/tokio/latest/tokio/) as the runtime for supporting async features.

Tokio runtime will handle each the function of each requests. As the "backend" after gRPC api.

#### API ####

This project server side will open gRPC API for other service/cli/etc.. It uses [tonic](https://docs.rs/tonic/latest/tonic/) as the gRPC lib for code generating and server running.

gRPC API schema includes the fields of gRPC request and response for client and server side. 

```grpc
proto Request {
    command ;; start/stop/get status/stream output/etc. commands
    client authorization info {
        ...
    }
}

proto Response {
    command_status ;; success/failed/etc. for the API call
    job_statuc {
        job_status ;; status of job
        ...
    }
}

```

#### Authorization ####

Use mTLS as the authentication method for making sure the identity of client. After the transport layer (mTLS) authorization, we also need a way to know which client it is. In this project, I might hard code some `client id` of clients, keep the publickey-client relations inside server side (which should keep in DB in real world). 

Potential packages can support the mTLS:

+ rustls
+ native-ssl
+ openssl

#### cgroups ####

After jobs run, their pid should add to specific cgroup.

#### Logs ####

Every jobs should keep their logs in file for client to check. When the client need the streaming outputs of the job, server should write down the log to the file and send to client side at the same time. 

### Client side ###

#### CLI interface ####

CLI is a single terminal application call the gRPC API on server. It has ability of print out the response from server and streaming printing. 

## Tests ##

### How to run ###

As the design upper, there are one CA crt, one server pem (or p12), and at least two client pem (or p12). Run commands below can generate them through the script:

> cd ca
> sh gen.sh

After the all keys generated, use `cargo run -- --bin server` to start the server. 

Then, in other terminal, `cargo run -- --bin client` to run the CLI app.
