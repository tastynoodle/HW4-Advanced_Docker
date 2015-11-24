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

The structure of this project is:

* deploy/
	* App/
	* blue.git/
	* green.git/
	* Dockerfile

I built three dockers for app, green and blue. Their dockerfiles are the same. For convenience I used the codes of Deployment Workshop.

```
FROM    centos:centos6

# Enable EPEL for Node.js
RUN     rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
# Install Node.js and npm
RUN     yum install -y npm

# Bundle app source
COPY ./App /src
# Install app dependencies
RUN cd /src; npm install

EXPOSE  8080
CMD ["node", "/src/main.js", "8080"]
```

To finish the task three hosts are added. One is the `post-commit` hook of main project.

```
#!/bin/bash

cd Deploy
docker build -t app .
docker stop app
docker rm app
docker run -p 3000:8080 -d --name app hw4-app
docker tag -f app localhost:5000/app:latest
docker push localhost:5000/app:latest
```

When do a `git commit`, the project will be rebuilt and pushed to the registery.

Another two hooks are `post-receive` hook of green and blue projects. Below is the hook for green project.

```
#!/bin/bash

docker pull localhost:5000/hw4-app:latest
docker stop app-green
docker rm app-green
docker rmi localhost:5000/app:current
docker tag localhost:5000/app:latest localhost:5000/app:current
docker run -p 3001:8080 -d --name app-green localhost:5000/app:latest
```

This hook makes sure the slices are updated on time.