# Web-based Host Terminal via ttyd + Docker

This project runs a lightweight `ttyd` web terminal in Docker that provides direct shell access to the host system as a specified user (or root).

It uses `nsenter` to enter the host's namespaces, combined with `ttyd` to serve the terminal over HTTP.

Access the terminal at: **http://127.0.0.1:7681** (or the host's IP if accessing remotely).

## Features

- Direct host shell access from a browser
- Minimal Alpine-based container
- Color support (xterm-256color)

## Security Note

The container requires privileged access and host namespaces to function.  
It is bound to localhost by default (`-i 127.0.0.0`).  
This setup is intended for use behind a reverse proxy that provides appropriate authentication and access control (e.g., Nginx, Traefik, Caddy with Basic Auth, OAuth, etc.).

## docker-compose.yml

```yaml
services:
  ttyd:
    build:
      context: .
      dockerfile_inline: |
        FROM alpine:latest
        RUN apk add --no-cache ttyd util-linux bash shadow
        ENV TERM=xterm-256color
    container_name: host-terminal
    restart: always
    privileged: true
    pid: "host"
    network_mode: "host"
    # Change the last part to your desired user/shell
    # Normal user: su - yourusername
    # Root:        /bin/bash
    command: ttyd -p 7681 -i 127.0.0.1 nsenter -t 1 -m -u -i -n -p su - yourusername
