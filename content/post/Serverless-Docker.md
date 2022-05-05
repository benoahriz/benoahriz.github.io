---
title: Server...less docker
date: 2016-07-24 21:39:25
tags:
  - docker
  - golang
  - go-dockerclient
  - go-dexec
author:      "Benjamin Rizkowsky"
---

## Server...less Docker... or stream processing with docker using golang.

At Dockercon I saw a 'cool hack' they called [server-less docker](https://github.com/bfirsh/serverless-docker).  Basically the idea is to use containers as functions in programs instead of using containers as servers.  

This brings back the concept of [cgi](https://en.wikipedia.org/wiki/Common_Gateway_Interface).  Further on the idea of server-less docker is to create a program and inside the program functions call out to containers to do the work.

Instead of needing to maintain a huge cluster of servers for a particular purpose you can have raw compute available as a swarm cluster and just utilize it as necessary for whatever computation you need.  These functions can scale as much as you need on demand.

In my experiments I got familiar with the following golang docker libraries.

- [go-dockerclient](https://github.com/fsouza/go-dockerclient)
- [go-dexec](https://github.com/ahmetalpbalkan/go-dexec)

### go-dexec

This is a library to emulate most of the functionality with some exceptions to the exec library for calling functions and attaching to the cmd inputs and outputs. 

### go-dockerclient

This is the best docker client library for golang.  Pretty much if you are doing any work with the docker api you'll want to use this library.  


Ok so in comes my project called [go-hydra](https://github.com/benoahriz/go-hydra).  I have a few goals in  my experiments.  

One was to create an [unoconv](https://github.com/dagwieers/unoconv) container that can take stdin and produce stdout that is usable.  

Another goal was to get familiar with the go http or mux package as well as go-dexec.

The goal of go-hydra is to be a front end http api/server that serves client requests that get processed on the back end which would normally be a docker swarm cluster.  Each function runs a separate container and sends the response back to the client.  The containers would primarily use stdin and stdout if possible.

## The first piece.

In my use case I wanted to create a docker container that would take stdin and produce stdout of raw bytes.   This would allow you to do something like the following.

``` bash
echo "test" |docker run unoconv2 --stdin --stdout f pdf > testpdf.pdf
```

or

```bash
cat test/Checklist-Ubuntu_v7.pdf| docker run unoconv2 --stdin --stdout --format=txt > file.txt
```

This was a bit of a headache as the standard versions of unoconv in ubuntu 14.04 do not have support for stdin.  This means no streaming input :(

I had to basically build it off the latest code in the repo which has the support for stdin. See the dockerfile for details.  [Link](https://github.com/benoahriz/go-hydra/blob/master/Dockerfile)

Also unrelated to my hack projects main goals I also got a proof of concept apt-cacher-ng detector for build scripts which can greatly reduce the time needed for building container images that use apt packages.

## golang http

My second implementation experiment was to create a simple rudimentary http server that would execute some proof of concept code using the containers.

First I implemented a simple uploader... in theory my goal is to not need the uploader at all and stream the data direct from the client.  However one step at a time.

```bash
go run main.go
```

Then visit this url.  http://localhost:8080/upload

You can use this to simply upload your file to be processed to the http server.

Then once the file is uploaded.

```bash
 curl "http://localhost:8080/toupper/txt?filename=README.md"
```

The result you'll see is on the console of the web server.  This was a simple example not using the unoconv script but using simple 'tr' to convert the text to uppercase.

The way this is currently sort of working is that i'm reading the file in and then sending it to stdin to the container then capturing the result in stdout of the app.  The real goal was to get the output streamed into the body of the response but there are some shortcomings with the dexec library.

There are some major issues with these experiments.  Firstly that the unoconv library doesn't always work as expected.  I didn't go into  depth trying to figure out what was wrong with it.  So instead I continued with using the tr example on the dexec readme.  The other issues I ran into were the implementation of stdinpipe and stdoutpipe in dexec only allow you to use one at a time and I will need to dig deeper to figure out how to get that to work right.
