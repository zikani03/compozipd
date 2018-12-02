Compozip
========

This project implements a server (and client) for downloading packaged composer
vendor folders.

You upload your `composer.json` and the server will take of downloading the 
dependencies and zip them up in a nice downloadable archive.

Vendor archives can be downloaded as either `zip` or `tar` archives.

## API

Basically, it's a simple API with one endpoint. It also has an [OpenAPI specification](./swagger.yaml) ;P

```
POST /vendor/{project}/{extension}

Content-Type: multipart/form-data

... multipart-boundary stuff
composer=FILE_DATA
```

## Downloading Vendor archives

You may be thinking it's too much work to download from that API endpoint.
Fortunately, you don't have to code to download from the API endpoint.

You have at least two options, illustrated below:

### Download with CURL

```sh
curl -i -F "composer=@test_composer.json" --output test-vendor.zip -XPOST http://localhost:8080/vendor/test/zip
```

### Download with `compozip` CLI

```sh
$ ./compozip -port 8080 -c test_composer.json
Uploading test_composer.json ...
Downloading vendor archive (vendor.zip)...
Downloaded vendor archive to vendor.zip

$ ls
test_composer.json  vendor.zip.
```

<small>NOTE: You should be able to use a tool like [Postman](https://getpostman.com) 
or create your own HTML form ([See example in index.html](./index.html))</small>

## Building the code

You need to have [Go 1.11 (or greater)](https://golang.org), [PHP](https://php.net) and [Composer](https://getcomposer.org) installed.

You MUST build the code before starting the server.

```sh
$ git clone https://github.com/zikani03/compozipd.git
$ cd compozipd

# Build the Server
$ go build compozipd.go

# Build the CLI client
$ go build compozip.go
```

## Starting the Server

### Run the server binary directly

```
$ compozipd -u ./uploads -h "localhost:8080"
```

### Run the server via Docker

This project comes with a [Dockerfile](./Dockerfile) for running the server in
a Docker container. 

Use the following commands to build and start the Docker container.

```sh
$ sudo docker	build --rm -t=compozipd .
$ sudo docker	run --rm --publish=80:80 -it compozipd
```

## Why did you make this?

I saw a website that provided a "service" like this sometime back but couldn't 
seem to find it, so I decided to implement it myself.

## What problem does it solve/address?

It solves a problem that shouldn't be there, but unfortunately downloading composer
dependencies sometimes takes a long time when you're living in a 
third-world country with slow internet since Composer has to do multiple HTTP/Git requests.

The idea is to put this on a fast server in the Cloud to download the dependencies,
and just download the archive which _should be_ faster since it's just one HTTP
request instead of bajillions.

I intend to try using something like [WebSockets](https://w3c.github.io/websockets/)
or [Rsocket](https://rsocket.io) to stream the bytes of the vendor archive to 
the client. I hope that would make it somewhat faster to download the archive - 
not sure how true that is, yet.

## Caveats

* There is no guarantee that downloading the vendor archive will be faster than running
`composer install` on your machine as composer most likely has caches on your PC if
you use it often.

* This won't play nicely with private repositories/composer packages. I don't know
a good solution for that yet. Sorry.

* The Docker image/container uses [php:7.2-fpm-alpine](https://github.com/docker-library/php/blob/b99209cc078ebb7bf4614e870c2d69e0b3bed399/7.2/alpine3.8/fpm/Dockerfile) 
image from [Dockerhub](https://hub.docker.com/_/php/) and as such may NOT contain
all the PHP extensions that dependencies in your `composer.json` require. I
might create a base Docker image sometime that installs the MOST common 
PHP extensions to prevent this from happening. Contributions are WELCOME here!

* This is not _production_ ready but you are FREE to try whatever, yo.


## Contributing

So yea, you can file an Issue if you find a bug (most likely) or
have an Idea to improve this and I will try my best to resolve it.

Pull requests are most welcome and encouraged. :)

## LICENSE

MIT

----

Copyright (c) 2018, Zikani Nyirenda Mwase