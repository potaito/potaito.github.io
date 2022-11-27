---
layout: post
title:  "Docker usernames done right"
date:   2021-03-24 16:00:00 +0100
categories: docker development linux technical
---

This is another rather short note about a method I just discovered for managing the user permissions inside docker properly.

The problem: When creating a new docker container, the default user is going to be root. This is not desirable for a few reasons, and some applications even flat out refuse to work when launched as root. It particulraly becomes annoying when using docker containers as a build system for compiling an application. The advantage in that lies in a deterministic build chain. The disadvantage is that we would build as root, and when mounting the build directory into docker, the resulting binaries would be owned by root, even on the host machine.

The obvious solution then is to create a non-root user inside the docker container, that can obtain root access via `sudo` for when that is necessary. There are however various ways to achieve this. I don't want to dive into all the methods that did not work well for me, but rather present one method that does the trick.

## The method
We will create a new user inside the docker container, that has the same username, user id and group id as the currently logged in user on the host machine. This has the advantage, that when mounting directories and files with `docker run -v`, the user inside the docker container will be the same one as the user outside the container on the host system. This is nice from a permissions perspective, as we won't have to mess with `chmod` constantly. 

The user name and id will be passed as `ARG` arguments with the docker build command. These `ARG` values are accesssible inside the Dockerfile, but will not persist during runtime later.

Dockerfile:
```dockerfile
FROM ubuntu:18.04

# Default arguments
ARG USER=nobody
ARG UID=1000
ARG GID=1000

# Perform the system setup as root, in this case we use htop as an example
RUN apt update
RUN apt install -y htop

# Clean up APT when done.
RUN apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# User management
RUN groupadd -g $GID $USER \
    && useradd -u $UID -g $GID -ms /bin/bash $USER \
    && usermod -a -G sudo $USER \
    && usermod -a -G users $USER \
    && echo "$USER ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# Run as new user from the installation path
USER ${USER}:${GID}

# Copy entrypoint
COPY entrypoint.sh /usr/local/bin/entrypoint.sh
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]

CMD ["/bin/bash"]
```

entrypoiny.sh:
```shell
#!/bin/bash

# additional initialization would go here
# ....

exec "$@"
```

Now simply pass the host machine's user name and ID to the docker build command, and it will bake them into the docker container:
```shell
docker build . -f Dockerfile -t my-container \
            --build-arg USER=$USER \
            --build-arg UID=$(id -u) \
            --build-arg GID=$(id -g)
```

Start the container and open an interactive shell. The user inside the docker container should be identical to the user on the host machine. To prevent any damage from this small tutorial the command below is mounting the volume here with read-only access, which means that the docker daemon will prevent and write operations even though from a file-system perspective the user would have permission to do that. Change the mount to wahtever directory you need to work with.

```shell
docker run -it --rm \
    -v /home/$USER:/home/$USER:ro \ # This is just an example
    -w /home/$USER \
    my-container:latest
```