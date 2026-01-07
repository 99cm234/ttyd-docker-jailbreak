# ttyd + nsenter command line flags reference

This document explains the command-line flags used by the `ttyd` web terminal
container to provide browser-based shell access to the host system.

It is intended to complement the main project README and focuses specifically
on what each flag does and why it is required.

---

## Full Command

The container is started with the following command:

    ttyd -p 7681 -i 127.0.0.1 \
      nsenter -t 1 -m -u -i -n -p \
      su - yourusername

This command has two logical parts:

1. `ttyd`  exposes a terminal over HTTP
2. `nsenter`  enters the host’s namespaces and launches a shell

---

## ttyd Flags

`ttyd` runs a web server that presents a terminal session in a browser.

### -p 7681  
Port number

- Specifies the TCP port ttyd listens on
- The terminal is accessible at:

      http://127.0.0.1:7681

---

### -i 127.0.0.1  
Bind address

- Restricts ttyd to listen only on the loopback interface
- Prevents direct access from external networks

This is a critical security control.  
Removing this will expose the terminal on all interfaces.

---

## nsenter Flags

`nsenter` allows the container to enter the host’s Linux namespaces so that
commands run as if they were executed directly on the host.

### -t 1  
Target process

- Targets process ID 1 on the host
- PID 1 is the host init system (systemd or init)
- All namespaces are entered relative to this process

---

### -m  
Mount namespace

- Grants access to the host filesystem
- Allows browsing and modifying `/`, `/home`, `/etc`, etc.

---

### -u  
UTS namespace

- Shares hostname and domain name with the host
- Commands see the same hostname as the host system

---

### -i  
IPC namespace

- Shares inter-process communication resources
- Enables access to shared memory, semaphores, and message queues

---

### -n  
Network namespace

- Uses the host’s network stack
- Required for host-level networking tools and services

---

### -p  
PID namespace

- Allows visibility of host processes
- Enables tools like `ps`, `top`, and `htop` to show host processes

---

## User and Shell Selection

### su - yourusername

- Switches to a login shell for the specified user
- Loads the user’s environment and shell configuration files

---

### Common Alternatives

Run as root:

    /bin/bash

Specify a shell explicitly:

    su - yourusername -s /bin/bash

---

## Why This Setup Works

- privileged: true allows namespace access
- pid: host enables visibility of host processes
- network_mode: host provides full host networking
- nsenter bridges the container and host environments

The result is a browser-accessible terminal that behaves as if it is running
directly on the host system.

---

## Security Warning

This setup provides full host access.

It should only be used:
- On trusted systems
- Bound to localhost
- Behind a reverse proxy providing authentication and access control

Never expose this service directly to the internet.
