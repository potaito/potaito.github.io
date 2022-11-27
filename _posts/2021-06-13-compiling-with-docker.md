---
layout: post
title:  "Compiling with docker"
date:   2021-06-13 15:50:00 +0100
categories: [development]
---

Here's a quick note on how to use Docker containers for compiling. Why is this useful? Because it allows us to test the installation of the toolchain and external libraries on a virtually new machine, and it's possible to test the compilation with specific Operating Systems.

![Docker containers are boxes for your code!](/images/pexels-photo-6505027.jpeg)

_Photograph by [Ryanniel Masucol](https://www.pexels.com/@ryanniel-masucol-1503495)_


In this example a hello-world C++ program is compiled and executed in an Ubuntu 20.04 Docker container. In a first step install the docker engine as described in [Docker's official documentation](https://docs.docker.com/engine/install/).

Then this example will require two files. The first one containes the C++ source code, which in this case is a very simple hello-world program. The second file is the build recipe for the docker container, which installs the dependencies that are required to compile our executable. For the sake of simplicity, create both files in the same directory.

C++ source code `hello.cpp`:
```c++
#include <iostream>

int main() {
    std::cout << "Hello World!" << std::endl;
    return 0;
}
```

The docker recipe `Dockerfile`. It's important to name it exactly like that:
```dockerfile
FROM ubuntu:20.04

# Perform the system setup as root and install the necessary dependencies
RUN apt update
RUN apt install -y clang

CMD ["/bin/bash"]
```

The first time, and everytime when there is a change in the Dockerfile, the docker container needs to be re-built. Building the docker container uses the recipe to create a ready-to-go container that can be spun up later. Build the docker container with

```shell
docker build . -t my_builder
```

The `-t my_builder` argument tags the container with the specified label, which can be used later to reference this container. The building can take a while depending on the amount of steps in the Dockerfile. The process will print the output of the commands as they are being executed, in this case the `apt` update and installation.

Once the docker container is built, we can launch it and take a look inside:

```shell
docker run -it --rm my_builder
```

This will start up the container we just built and create an interactive shell. The argument `-i` specifies the interactive mode allowing us to enter commands, and `-t` opens a pseudo TTY device, which is a terminal that we can use. Any change that is performed inside of the docker container now will not persist when shutting down the docker container. Exit the container with `CTRL+D`, which will also automatically destroy the container because we specified the `--rm` argument.

Now, onto using this docker container to build the `hello.cpp` C++ program. The command for running the docker container from above needs to be extended a little bit to map the current folder into the docker container as a read-write path. The command below will do that and compile our test application:

```shell
docker run -it --rm -v $PWD:$PWD:rw -w $PWD my_builder clang++ hello.cpp -o hello && ./hello
```

The arguments that are new here are:
- `-v $PWD:$PWD:rw`: Maps the current directory we are in to the docker container, and places it under the same path. That's why `$PWD` appears twice. The `:rw` flag at the end tells docker to map the specified folder with read-write permissions, allowing the docker container to make changes to the host filesystem. We need this because we want to write the compiled binary to the mapped folder, such that it persists when the docker container is closed.
- `-w $PWD$` This is a convenience argument that tells docker to change the work-directory to the specified path upon launching the container.
- `my_builder` is the name of the docker image that is used to launch a container. Multiple containers based on the same image could be launched simultaneously.
- `clang++ hello.cpp -o hello && ./hello`: This is the custom shell command that will be executed inside the docker container once it's launched. The command uses clang to compile the C++ binary and then executes it afterwards. This command would look different, if for example we would be using `cmake` or `make` instead of calling `clang++` directly.

After running the command above, docker should start a container, compile the binary, and exit the container again. Check with `ls` that the `hello` binary was created, and execute it with `./hello`. Note that this will only work, if the host system and the docker container base OS are compatible, for example when both are running the exact same Linux OS. Otherwise it might happen, that the host machine cannot execute the compiled binary because it was compiled and linked to incompatible libraries, that are not present on the host machine.

## Making your own build chain
This setup is quite simple in the sense that we only need to install one dependency in Ubuntu, namely the `clang` compiler and call it directly to compile our binary. Extend the Dockerfile above to install different compilers or other dependencies such as libraries and build tools like `cmake`. Then modify the `docker run` command to call exactly the commands you need to build your program.

The method above allows a developer to test exactly how a new machine needs to be set up in terms of installing the build tools. And it makes it possible to test compilation and execution of a program on different operating systems. Last but not least this method also forms the basis for a continuous-devleopment setup on a server, for example [github actions](https://github.com/features/actions) or [Jenkins](https://www.jenkins.io/), that uses a docker container in exactly the same way to compile test and release versions of your software.

## Further reading
You may have noticed that the executable compiled with docker ends up being owned by `root` on the host machine. As such `sudo rm` is required to remove it. By default docker executes containers as root, and with the root user inside the docker container. But it's actually possible to use any user inside the docker container, and we can even use the same user name and ID as the currently logged-in user on the host machine to compile the binaries inside the docker container. This is an advanced topic and the full method is explained in an [earlier post](../docker-usernames-done-right) I wrote.