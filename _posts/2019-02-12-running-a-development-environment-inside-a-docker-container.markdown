---
layout: post
title: "Running a development environment inside a Docker container"
date: 2019-02-12
description: >-
  An introduction to setting up a dedicated Docker container for hassle-free
  scientific code development.
author: Bert Vandenbroucke
tags: 
  - Scientific computing
  - Tools
  - Code Development
---

Dependencies are usually a nightmare for scientific software projects. 
Scientific software needs to run on a variety of systems, ranging from 
remote clusters with very strict limitations on the available compilers 
and libraries, to student laptops that have nothing you need already 
installed or - even worse - run Windows. As a result, you should think 
very carefully about the dependencies you introduce; if you use a 
library to do something that you can easily do without that library, 
you're probably better off not using it to avoid future problems.

Despite that, some dependencies are unavoidable. If you write code in a 
low-level language like C(++) (or ancient alternatives), you need a 
compiler to convert this code into highly efficient machine code. If you 
want to somewhat automate your compilation process, you need a build 
tool like GNU Make. If you don't want to write Makefiles manually, you 
need a configuration tool like CMake or Automake. If you want somewhat 
portable and efficient I/O, you probably want to use a library like HDF5 
to manage your I/O. This is just the bare minimum of what seems 
necessary. And let's not forget a repository system like git. Always 
important...

And so you always end up with a list of things that need to be present 
in order to build and run your software, even if this list is small and 
reasonable. Which means you will run into situations where you try to 
compile your software on a system that does not have the right 
libraries/compilers. Or that has an older/newer version of a 
library/compiler that triggers unexpected errors.

# The solution: containers?

Container systems like the hugely popular 
[Docker](https://www.docker.com/) provide a promising solution to these 
issues. In essence, these systems are a lightweight version of virtual 
machines (like [VirtualBox](https://www.virtualbox.org/)), which enable 
you to run a *guest* operating system within your *host* operating 
system, as if it were simply a program. The guest operating system runs 
as if it was running on a separate machine, and generally does not know 
anything about the host operating system. For virtual machines, this is 
absolutely true: the host operating system will *emulate* a virtual 
computer and the guest system needs to be installed and run within that 
virtual computer. Container systems use some of the machinery from the 
host system (e.g. the *kernel* that runs the operating system), but 
still pretty much look and feel like separate systems.

The idea of these containers is that they allow you to bundle and manage 
your dependencies in a portable way. You can create a container *image* 
within Docker on one machine, and then copy that image to another 
machine and run it there. For the container, it will seem as if nothing 
changed. So instead of just copying your software to another system, you 
copy the entire environment that is necessary to build and run your 
software as well. As long as the target machine has Docker installed, it 
should be able to recreate this environment and guarantee that all 
libraries and compilers are available.

There are a few caveats to this. First of all, the requirement that 
Docker is installed on the target machine clearly introduces a new 
dependency. So you are still replacing a bunch of dependencies by a 
single, very important dependency. For systems that you control (your 
own computer, a student's computer...) this is easy to realise (Docker 
is available on all 3 major desktop operating systems). For remote 
clusters, this is less straightforward. Furthermore, there are some 
limits to what you can bundle inside a container image: software that is 
compiled within a container on one machine might not work within that 
same container on another machine if the compilation makes use of 
system-specific optimisations. Which again is something you probably 
want to do on a remote cluster. So as a general rule, containers are not 
really suitable for remote clusters.

Despite these caveats, I find Docker a very useful tool in many 
situations, and I mainly use it in two situations:
 * to compile and run software with different versions of libraries and 
compilers than are available on my desktop. This is useful to debug 
issues encountered on remote systems that would otherwise be impossible 
to reproduce.
 * to provide a more consistent build environment between different 
computers running different operating systems or operating system 
versions. This is especially useful when you just want to work on the 
software without requiring a system-tuned build.

In both use cases, you need to create a container with the right 
environment and then be able to access it appropriately. Below I will 
guide you through the setup of an example container. I will assume you 
are on a system that has Docker installed, and that your Docker 
installation works (I have experienced problems with [Docker having no 
internet](https://stackoverflow.com/questions/20430371/my-docker-container-has-no-internet) 
in the past). This post is similar to a [code and cake presentation I 
gave last year for the Physics and Astronomy department at the 
University of St 
Andrews](https://code-and-cake.github.io/talks/2018-07-11-bert-Docker), 
and features real-life examples of containers (that are also useful as a 
personal reference).

# Step 1: container creation

The first step in building a container is actually creating the 
container *image*. Note the terminology: *container* refers to the 
running instance of a Docker container, while *image* refers to the 
portable saved state of the container. Images in Docker are built up in 
a hierarchical way: every image is based on a *base image* and extends 
this image with additional features. [Docker 
Hub](https://hub.docker.com/) hosts more than 100,000 Docker images that 
can be used as base image, including all versions of major Linux 
distributions.

To build an image, you need to run the `docker build` command, and tell 
it which base image to use and which additional packages to install. You 
do this by creating a `Dockerfile`. A basic `Dockerfile` looks like 
this:

```
FROM ubuntu
RUN apt-get update && apt-get install gcc git -y
```

The corresponding image can be created with

```
docker build .
```

The `.` tells Docker to look for the `Dockerfile` in the current working 
directory.

This command will create a new container based on the latest stable 
image of Ubuntu (`FROM ubuntu`) and install `git` and `gcc` within that 
container (`apt-get install gcc git -y`, `-y` automatically replies 
`yes` to any confirmation messages displayed by `apt-get`). The command 
will produce a lot of output and will finally finish with something 
similar to

```
Successfully built 26c860824c1a
```

The hash string in the end is the ID of the newly created image. This ID
is also displayed if you list all Docker images on the system using

```
docker images
```

This list will also contain all the images that make up the hierarchical 
structure of the new image, e.g. the `ubuntu:latest` image that is the 
base image.

If you are only interested in creating a container, then this is all you 
need. Feel free to jump ahead to [step 
2](#step-2-accessing-the-container).

## A more complete container

The container above has very little functionality; it consists of a bare 
Ubuntu system with a minimal set of libraries and no user functionality. 
Some important system features, like SSH-connectivity, are not 
available. This makes it hard to use the container for actual software 
development. Below is an example of a `Dockerfile` that sets up a more 
useful image, including a fully functional user account (this is a 
real-life example of a container I used to test my code 
[CMacIonize](https://bwvdnbro.github.io/CMacIonize/) on Ubuntu 18 before 
I upgraded my computers to this Ubuntu version):

```
FROM ubuntu:18.04

RUN apt-get update && \
    apt-get install git locales gcc g++ cmake libboost-dev libhdf5-dev make -y
RUN locale-gen en_GB.UTF-8
ENV LANG en_GB.UTF-8
ENV LANGUAGE en_GB:en
ENV LC_ALL en_GB.UTF-8
RUN useradd -u 110090 -o -ms /bin/bash bv7
USER bv7
WORKDIR /home/bv7/
VOLUME /home/bv7/.ssh
```

In this case, I created the image in a slightly different way by giving 
it a *tag* that makes it easier to run it:

```
docker build --rm -t cmi_u18 .
```

The `--rm` option tells Docker to remove the intermediate images, while 
the `-t cmi_u18` sets the image tag.

The `Dockerfile` is similar to the one above, but installs more 
dependencies. It also installs correct *locales* (`RUN locale-gen...` to 
`ENV LC_ALL...`), which are the language settings that are used to parse 
regular expressions. I need these because I use regular expressions in 
CMacIonize. I then add a user account with the same name and user ID as 
my actual user account on the host system (`RUN useradd...`), tell 
Docker to automatically run the container using that user (`USER bv7`), 
and to open the container in the user home folder (`WORKDIR /home/bv7/`) 
as is also the case when you log in to a remote Linux system. The final 
line creates an empty `.ssh` folder inside the container that will be 
used to mount my actual `.ssh` folder [later 
on](#step-3-sharing-files-with-the-host-system).

# Step 2: accessing the container

To run a container like the container created above, we need the `docker 
run` command, as well as the ID of the container image. This ID is 
usually a 12 character hash string, but can also be a tagged name 
(depending on how the image was created). By default, `docker run` will 
not do anything; if you want to run some software inside the container, 
you have to pass the relevant commands on to the `docker run` command, 
e.g.

```
docker run 26c860824c1a gcc --version
```

This will open the container we created above (assuming this container 
has ID `26c860824c1a`), run `gcc --version` inside it and then exit the 
container again. Nothing that happened inside the container is actually 
saved. This might seem quite useless, but is actually the main way 
Docker is used in real applications, where Docker containers are used to 
run web services on remote servers and they only need to do a single 
thing, i.e. run the web service software.

For development purposes, it is useful to have interactive access to the 
container. To get this, we need to run a *shell* command inside the 
container (e.g. `bash`), and add appropriate flags to the `docker run` 
command to make sure that it does not automatically exit:

```
docker run -i -t 26c860824c1a bash
```

The `-i` makes sure Docker reads input from the standard input (as if it 
was a shell program itself), while `-t` makes sure all appropriate 
*signals* are forwarded to the `bash` command, as if it was an actual 
shell. Both flags can be combined into `-it`.

This command will open a `bash` shell inside the container that exists 
until you exit it (using `exit`), at which point the container will 
stop. Note that again, changes inside the container are not saved.

To save the container, we have to convert the container into a new image 
(remember that *images* are saved *containers*). This can be done using 
the `docker commit` command. This command takes the ID of a container as 
argument, and creates a new image based on the contents of that 
container. Note two important points: (a) the ID of a container is not 
the same as the ID of the underlying image (to get a list of containers 
and their IDs, run `docker container ls`), (b) `docker commit` will 
overwrite the previous image on which the container was based, and hence 
literally saves the changes you made to the container. It will still 
create a new ID for the new image; the old image ID will no longer work.

Again, this is quite a rudimentary way of using a Docker container; I 
prefer a more systematic approach in which the container is treated as 
if it were a remote system, and with a more systematic way of saving it 
(and keeping a consistent image name). I will illustrate this below, 
using the [more complete container image](#a-more-complete-container).

## Normal access

To automate saving the container state whenever I exit the container, I 
will wrap the `docker run` command into two auxiliary scripts. The first 
script allows normal access to the container; the second script allows 
[root access](#root-access) to the container in case this is required.

The first script consists of three `docker` commands:
 - a call to `docker run` that accesses the container using the user 
account we created during the image build and that mounts the local 
`.ssh` folder so that the container shares the SSH keys and 
configuration of the host system
 - a call to `docker ps` that recovers the hash tag of the container 
when it exits
 - a call to `docker commit` that saves the final state of the container 
as an image with the same tag as the original container image

The full script is shown below:

```
#! /bin/bash
docker run -u bv7 -v ~/.ssh:/home/bv7/.ssh -t -i cmi_u18 bash -l
docker_id=$(docker ps --format "{{.ID}}" -l)
docker commit $docker_id cmi_u18
```

The `-v` syntax for volume mounting will be explained in more detail 
[below](#step-3-sharing-files-with-the-host-system). The additional `-l` 
flag is passed on to the `bash` command and tells `bash` to run as if it 
were a *login shell* (which means it will automatically load a set of 
configuration files, as if it were a login shell on a remote cluster). 
The `docker ps` command is fine-tuned to only output the ID of the 
latest container that was *created* (note that this script will behave 
weirdly if you run multiple containers at the same time - don't do 
this), and its output is saved in a variable called `docker_id`, which 
is then passed on to the `docker commit` command.

If you are on a Linux system, you can make this script executable by 
running `chmod +x script.sh` and then run it using `./script.sh`. You 
can also just run the script directly using `bash script.sh` or `source 
script.sh` (or using any other shell). This is all you need to access 
the container; whatever you do inside the container is now automatically 
saved!

## Root access

If you did not create a user account during image creation, then `docker 
run` by default gives you root access inside your container. This means 
that you can do pretty much everything a root user can do within your 
container system, e.g. install additional software. If you save your 
container state after those changes using `docker commit`, then you can 
change the container after its creation using `docker build`. This is 
not ideal from the point of view of dependency management (it is pretty 
useful to have all dependencies listed in a `Dockerfile`), but is 
nonetheless very handy if you use your Docker container for development, 
when dependencies can change.

If you set up your container using the advanced `Dockerfile` that also 
creates a user account, then by default `docker run` will access the 
container using that user account, and you will no longer have root 
access. In this case, there is no way to install additional dependencies 
from within the container; `sudo` will not work, and `su` will ask you 
for some unknown password (as far as I can tell it is not the password 
of any user account on the system, nor an empty password, so I have 
really no idea what the password is). The only was to still install 
additional software is by accessing the container as root when starting 
it using `docker run`. This is exactly what the below variant of the 
wrapper script does:

```
#! /bin/bash
docker run -u 0 -t -i cmi_u18 bash -l
docker_id=$(docker ps --format "{{.ID}}" -l)
docker commit $docker_id cmi_u18
```

The main difference with the normal wrapper script is the `-u 0` 
parameter, which tells Docker to use the root user instead of the 
default user. We also omitted the volume mounting, as we are not 
interested in doing anything else with our container than installing 
additional software. The two final lines still save the latest state of 
the container, so that changes are also saved.

# Step 3: sharing files with the host system

The containers we have used until now are separate, *isolated* systems 
running inside your host operating system. They act and feel as 
completely separate systems, and as such it is not possible to share 
data between the container and the host system. Since containers do not 
ship with a graphical environment and hence do not support things like 
viewing images or opening windows, this can be quite annoying.

Fortunately, Docker supports data sharing, by means of *volumes*. A 
*volume* is a shared folder (or a network location or even an actual 
volume like a USB drive) that exists on the host system, and that is 
*mounted* in the container as if it were a folder in the container file 
system. If the host system runs the same operating system as the 
container, then the folder is simply shared; if the file systems on host 
and guest are different, some underlying conversions happen that can 
cause some overhead. Volumes can be mounted in various ways; I will only 
discuss the option I find the most user-friendly.

Mounting a volume is pretty straightforward, you just add an option to the
`docker run` command:

```
docker run -it -v /home/bv7/Documents:/Documents 26c860824c1a bash
```

This will automatically mount the `/home/bv7/Documents` folder on the 
host system as `/Documents` in the container. You can tell Docker that 
you will always mount a certain volume by adding a `VOLUME` line to the 
`Dockerfile` (see the [more complete container 
example](#a-more-complete-container)), but this is not required.

If you add files to the shared volume within the container, these 
changes will also show up within the host file system, and vice versa. 
Changes made to the shared folder will always persist when you exit the 
container, even if you do not save the container state using `docker 
commit`.

Which folders can be shared in this way depends on the host system. On 
Linux systems you can pretty much share everything, while OSX has very 
strict settings that limit which system folders can be shared. Note that 
files created within the container will automatically get permissions 
based on the Docker user. Since by default this is the root user, this 
can lead to security issues. This is another reason why you should 
probably use a more complete container if you plan to do things like 
volume sharing. It is also the reason that I make sure the Docker user 
in the [more complete container](#a-more-complete-container) has the 
same user ID as my user account on the host system.
