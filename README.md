### HW4-Advanced_Docker

#### File IO

In this part two dockers are used. One is called `docker_write`, it will write to the `foo.txt`.

```
FROM ubuntu:14.04

RUN apt-get update
RUN apt-get install -y socat
EXPOSE 9001
RUN echo "Hello, world." > foo.txt
CMD ["socat", "TCP-LISTEN:9001,reuseaddr,fork", "SYSTEM:'cat foo.txt'"]
```

Another one is called `docker_read`, it will read the `foo.txt`.

```
FROM ubuntu:14.04

RUN apt-get update
RUN apt-get -y install curl
CMD ["curl", "docker_write:9001"]
```

[demo](https://youtu.be/8LGlXIkZyzw)

#### Ambassador

In this part I build two vagrants. One is called `Ambassador`, it uses two containers to deploy redis server and ambassador.

```
redis_ambassador:
    container_name: redis_ambassador
    links:
     - redis
    ports:
     - "6379:6379"
    image: svendowideit/ambassador
redis:
    container_name: redis
    image: redis
```

Another one is called `client`, it used two containers to deploy redis client and ambassador.

```
client_ambassador:
    container_name: client_ambassador
    expose:
     - "6379"
    environment:
     - REDIS_PORT_6379_TCP=tcp://192.168.33.10:6379
    image: svendowideit/ambassador
client:
    container_name: client
    links:
     - client_ambassador:redis
    image: relateiq/redis-cli
```

[demo](https://youtu.be/Psbp2ivQYfs)

#### Docker Deploy

In this part I build three dockers for app, green and blue.