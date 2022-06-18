# Design Doc #

## Requirements ##

> Implement a prototype job worker service that provides an API to run arbitrary Linux processes.

## Design ##

This project will have client app and server daemon app. Server app will open API and receive the commands from client side then run the command as the client wish.

Client side is a command line app for users run the commands through terminal. Users should have the ability of `start, stop, get status, and stream output of a job`.

### Server side ###

Server side application should listen the port `9090` for gRPC requests. 

#### Server library ####

After receive gRPC request, if it is `start`, core library will call `Command::new` in `std::process` module for make a new processing. Core library will keep the info of this processing (pid/stdin/stdout/stderr/etc.) for future use, like stopping it or get the stdout/stderr. 

The `Command::new` actually generate the child processing of server application processing. The lifecycle of the jobs spawned by server will from they start to the earlier time point of receive stop of this job or server graceful stop. 

#### Runtime ####

This project will use [tokio 1.x](https://docs.rs/tokio/latest/tokio/) as the runtime for supporting async features.

Tokio runtime will handle each the function of each requests. As the "backend" after gRPC api.

#### API ####

This project server side will open gRPC API for other service/cli/etc.. It uses [tonic](https://docs.rs/tonic/latest/tonic/) as the gRPC lib for code generating and server running.

gRPC API schema includes the fields of gRPC request and response for client and server side. 

```grpc
message Request {
    string command; // start/stop/get status/stream output/etc. commands
    id client; 
    Authorization authorization; 
}

message Authorization {
    ...
}

message Response {
    int command_status; // success/failed/etc. for the API call
    int job_status // job status
    ...
}

service App {
    rpc StartJob(Request) returns (Response)
    rpc StopJob(Request) returns (Response)
    rpc JobStatus(Request) returns (Response)
}

```

#### Authorization ####

Use mTLS as the authentication method for making sure the identity of client. After the transport layer (mTLS) authorization, we also need a way to know which client it is. In this project, I might hard code some `client id` of clients, keep the publickey-client relations inside server side (which should keep in DB in real world). 

Client id will contained inside the RPC request. Server side will keep a table of the client role (admin/normal/etc.). In this project, I will put the hard code of this table inside server app's memory.

Potential packages can support the mTLS:

+ rustls
+ native-ssl
+ openssl

Depends on the demo I wrote, cipher suites should be `TLS13_AES_256_GCM_SHA384`.

#### cgroups ####

After jobs run, their pid should add to specific cgroup before it can folk..

#### Logs ####

Every jobs should keep their logs in file for client to check.

For real time streaming, there should be a output buffer and a error buffer of each jobs' stdout/stderr. The log file writing behavior will read the data from buffer and clean it. When client want to see the streaming log, buffer structure should not only write log to file, but also write to client socket.

### Client side ###

#### CLI interface ####

CLI is a single terminal application call the gRPC API on server. It has ability of print out the response from server and streaming printing. 

Example:

```shell
client --cacert ca.crt --cert client_0.pem start 'sleep 2'
```

## Tests ##

### How to run ###

As the design upper, there are one CA crt, one server pem (or p12), and at least two client pem (or p12). Run commands below can generate them through the script:

> cd ca
> sh gen.sh

After the all keys generated, use `cargo run -- --bin server` to start the server. 

Then, in other terminal, `cargo run -- --bin client` to run the CLI app.
