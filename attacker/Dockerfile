FROM alpine:latest

# https://github.com/alpinelinux/docker-alpine/issues/98
RUN sed -i 's/https/http/' /etc/apk/repositories

RUN apk update && apk upgrade && \
    apk add --no-cache bash powershell openssh curl net-tools iputils netcat-openbsd socat
    
EXPOSE 5009/tcp
EXPOSE 5009/udp

ENTRYPOINT [ "/bin/bash" ]

