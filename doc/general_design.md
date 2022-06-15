# Design Doc #

## Requirements ##

> Implement a prototype job worker service that provides an API to run arbitrary Linux processes.

## General Design ##

This project will have client app and server daemon app. Server app will open API and receive the commands from client side then run the command as the client wish.

Client side is a command line app for users run the commands through terminal. Users should have the ability of `start, stop, get status, and stream output of a job`.

### API ###

gRPC

### CLI interface ###

### Authorization ###

mTCP

### cgroups ###


## Tests ##

### Unit tests ###

### Integration tests###
