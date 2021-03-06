---
title: "Get Started, Part 3: Services"
keywords: services, replicas, scale, ports, compose, compose file, stack, networking
description: Learn how to define load-balanced and scalable service that runs containers. 
---
{% include_relative nav.html selected="3" %}

## Prerequisites

- [Install Docker](/engine/installation/).
- Read the orientation in [Part 1](index.md).
- Learn how to create containers in [Part 2](part2.md).
- Make sure you have pushed the container you created to a registry, as
  instructed; we'll be using it here.
- Ensure your image is working by
  running this and visiting `http://localhost/` (slotting in your info for
  `username`, `repo`, and `tag`):

  ```
  docker run -p 80:80 username/repo:tag
  ```

## Introduction

In part 3, we scale our application and enable load-balancing. To do this, we
must go one level up in the hierarchy of a distributed application: the
**service**.

- Stack
- **Services** (you are here)
- Container (covered in [part 2](part2.md))

## Understanding services

In a distributed application, different pieces of the app are called
"services." For example, if you imagine a video sharing site, there will
probably be a service for storing application data in a database, a service
for video transcoding in the background after a user uploads something, a
service for the front-end, and so on.

A service really just means, "containers in production." A service only runs one
image, but it codifies the way that image runs -- what ports it should use, how
many replicas of the container should run so the service has the capacity it
needs, and so on. Scaling a service changes the number of container instances
running that piece of software, assigning more computing resources to the
service in the process.

Luckily it's very easy to define, run, and scale services with the Docker
platform -- just write a `docker-compose.yml` file.

## Your first `docker-compose.yml` File

A `docker-compose.yml` file is a YAML file that defines how Docker containers
should behave in production.

### `docker-compose.yml`

Save this file as `docker-compose.yml` wherever you want. Be sure you have
pushed the image you created in [Part 2](part2.md) to a registry, and use
that info to replace `username/repo:tag`:

```yaml
version: "3"
services:
  web:
    image: username/repo:tag
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"
    networks:
      - webnet
networks:
  webnet:
```

This `docker-compose.yml` file tells Docker to do the following:

- Run five instances of [the image we uploaded in step 2](part2.md) as a service
  called `web`, limiting each one to use, at most, 10% of the CPU (across all
  cores), and 50MB of RAM.
- Immediately restart containers if one fails.
- Map port 80 on the host to `web`'s port 80.
- Instruct `web`'s containers to share port 80 via a load-balanced network
  called `webnet`. (Internally, the containers themselves will publish to
  `web`'s port 80 at an ephemeral port.)
- Define the `webnet` network with the default settings (which is a
  load-balanced overlay network).

## Run your new load-balanced app

Now let's run it. You have to give your app a name -- here it is set to
`getstartedlab` :

```
docker stack deploy -c docker-compose.yml getstartedlab
```

> **Note**: If you get an error that "this node is not a swarm manager," go
  ahead and run `docker swarm init` and then retry. We'll get into the meaning
  of that command in [part 4](part4.md).

See a list of the five containers you just launched:

```
docker stack ps getstartedlab
```

You can run `curl http://localhost` several times in a row, or go to that URL in
your browser and hit refresh a few times. Either way, you'll see the container
ID randomly change, demonstrating the load-balancing; with each request, one of
the five replicas is chosen at random to respond.

## Scale the app

You can scale the app by changing the `replicas` value in `docker-compose.yml`,
saving the change, and re-running the `docker stack deploy` command:

```
docker stack deploy -c docker-compose.yml getstartedlab
```

Docker will do an in-place update, no need to tear the stack down first or kill
any containers.

### Take down the app

Take the app down with `docker stack rm`:

```
docker stack rm getstartedlab
```

It's as easy as that to stand up and scale your app with Docker. You've taken
a huge step towards learning how to run containers in production. Up next,
you will learn how to run this app on a cluster of machines.

> Note: Compose files like this are used to define applications with Docker, and
can be uploaded to cloud providers using [Docker Cloud](/docker-cloud/), or on
any hardware or cloud provider you choose with [Docker Enterprise
Edition](https://www.docker.com/enterprise-edition).

[On to "Part 4" >>](part4.md){: class="button outline-btn" style="margin-bottom: 30px"}

## Recap and cheat sheet (optional)

Here's [a terminal recording of what was covered on this page](https://asciinema.org/a/b5gai4rnflh7r0kie01fx6lip):

<script type="text/javascript" src="https://asciinema.org/a/b5gai4rnflh7r0kie01fx6lip.js" id="asciicast-b5gai4rnflh7r0kie01fx6lip" speed="2" async></script>

To recap, while typing `docker run` is simple enough, the true implementation
of a container in production is running it as a service. Services codify a
container's behavior in a Compose file, and this file can be used to scale,
limit, and redeploy our app. Changes to the service can be applied in place, as
it runs, using the same command that launched the service:
`docker stack deploy`.

Some commands to explore at this stage:

```shell
docker stack ls              # List all running applications on this Docker host
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file
docker stack services <appname>       # List the services associated with an app
docker stack ps <appname>   # List the running containers associated with an app
docker stack rm <appname>                             # Tear down an application
```
