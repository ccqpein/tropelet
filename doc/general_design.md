# Design Doc #

## Requirements ##

> Implement a prototype job worker service that provides an API to run arbitrary Linux processes.

## Design ##

This project will have client app and server daemon app. Server app will open API and receive the commands from client side then run the command as the client wish.

Client side is a command line app for users run the commands through terminal. Users should have the ability of `start, stop, get status, and stream output of a job`.

### Server side ###

Server side application should listen the port `9090` for gRPC requests. 

#### runtime ####

This project will use [tokio 1.x](https://docs.rs/tokio/latest/tokio/) as the runtime for supporting async festures.

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

#### CLI interface ####

CLI is a single terminal application call the gRPC API on server. It has ability of print out the response from server and streaming printing. 

#### Authorization ####

Use mTLS as the authentication method for making sure the identity of client. After the transport layer (mTLS) authorization, we also need a way to know which client it is. In this project, I might hard code some `client id` of clients, keep the publickey-client relations inside server side (which should keep in DB in real world). 

#### cgroups ####



### Client side ###

## Tests ##

### Unit tests ###

### Integration tests###
