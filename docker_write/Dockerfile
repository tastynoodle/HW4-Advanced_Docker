FROM ubuntu:14.04

RUN apt-get update
RUN apt-get install -y socat
EXPOSE 9001
RUN echo "Hello, world." > foo.txt
CMD ["socat", "TCP-LISTEN:9001,reuseaddr,fork", "SYSTEM:'cat foo.txt'"]