---
layout: post
title:  "Docker containers deeper dive"
date:   2017-01-26
author: "@lucjuggery"
tags: [linux,developer,operations]
categories: beginner
---
Let's play with Docker containers !

## What you will do

In this first lab, you’ll put into practice the base commands to manage Docker containers.
That means you will start to play and to have fun with containers :)

Important note: in the labs, when you see a command inside some grey rectangle, you just need to click on it so it is executed in the terminal (but if you want you can also write it manually into the terminal).
Let's see the first example below to get the version of the Docker Engine running on the platform.

```.term1
docker version
```

You should get an output like the following that shows Docker Engine (Server) and Client are running version 17.03.0.

```
Client:
 Version:      17.03.0-ce
 API version:  1.26
 Go version:   go1.7.5
 Git commit:   60ccb22
 Built:        Thu Feb 23 10:40:59 2017
 OS/Arch:      darwin/amd64

Server:
 Version:      17.03.0-ce
 API version:  1.26 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   3a232c8
 Built:        Tue Feb 28 07:52:04 2017
 OS/Arch:      linux/amd64
 Experimental: true
```

## Launch containers

### Running a container in foreground

Let's go direct to the point and launch a container based on alpine, and use the options to run it interactive mode.

```.term1
docker container run -ti alpine
```

Seems there is a problem there... you should have an error message like the following one.

```
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
0a8490d0dfd3: Pull complete
Digest: sha256:dfbd4a3a8ebca874ebd2474f044a0b33600d4523d03b0df76e5c5986cb
02d7e8
Status: Downloaded newer image for alpine:latest
docker: Error response from daemon: No command specified.
See 'docker run --help'.
```

The **No command specified** error means that you ran a container that does not have any default command and you did not specify one neither.

Let's fix that and run a new container providing the **sh** command.

```.term1
docker container run -ti alpine sh
```

Once created, the container should run the **sh** command. You should then now be in a shell running in an alpine linux.
Let's check that.

```.term1
cat /etc/issue
```

Part of the output should be like

```
Welcome to Alpine Linux 3.5
```

You can now exit the container.

```.term1
exit
```

### Running a container in background

Very often, containers are ran in background. They can expose services like HTTP API, databases, ...

Let's now use the **mongo** official image (for those who might not be very familiar with it, MongoDB is a very popular NoSQL database) and run a container in background (using the **-d** option).

```.term1
docker container run -d --name mongo mongo:3.2
```

Note: we have also used the **--name** option to assign a name to the container.

The command should pull the mongo image and return the ID of the newly created container.

```
Unable to find image 'mongo:3.2' locally
3.2: Pulling from library/mongo
5040bd298390: Pull complete
ef697e8d464e: Pull complete
67d7bf010c40: Pull complete
bb0b4f23ca2d: Pull complete
69454c78dfd1: Pull complete
d1fba36478cc: Pull complete
241f8db18496: Pull complete
32a0ed9f848b: Pull complete
0e19543cb407: Pull complete
Digest: sha256:6e8000e4efe09ec67328a16cecffff18e83fc0d20fa8f6b03b7396cf75b389e4
Status: Downloaded newer image for mongo:3.2
94b9d8babe13fe4b28ce926378a0b62ce32455c31791f4528c98dffa9a7a3524
```

An interesting thing we can do is to use the name of the container or the ID returned by the previous command and jump into the running container.
For this, we will use the **exec** command and the **-ti** options (to get an interactive tty).

```.term1
docker container exec -ti mongo bash
```

We are now in the container, that can be really handy for debugging purposes sometimes.
Let's check the running processes.

```.term1
ps ax
```

The output should be pretty much like the following one. As we can see, the process with PID 1 is **mongod** which is the MongoDB deamon, this is the command ran by default when a container is instantiated from a mongo image.

```
   PID TTY      STAT   TIME COMMAND
     1 ?        Ssl    0:01 mongod
    37 ?        Ss     0:00 bash
    51 ?        R+     0:00 ps ax
```

You can now exit the **mongodb** container:
```.term1
exit
```

## Inspection of a container

A container is a quite complex thing under the hood, the container API provides the **inspect** command to get all its details.

Let's launch a container based on nginx in background and name it www.

```.term1
docker container run --name www -d nginx
```

Use the inspect command against this container.

```.term1
docker container inspect www
```

There is a lot of information here, so we will use the Go template notation to get only the information we are interested in: the hostname and the API address of the container.

The **Hostname** key is under the **Config** one, and can be retrieved with the following command.

```.term1
docker container inspect --format "{{ "{{ .Config.Hostname " }}}}" www
```

You should see the below sha (not the exact value but something similar):
```
ec7cee99c78d
```

The **IPAdress** key is under the **NetworkSettings** and can be retrieved with

```.term1
docker container inspect --format "{{ "{{ .NetworkSettings.IPAddress " }}}}" www
```

You should see the below IP address (value may vary):
```
172.17.0.3
```

Select some other elements of the whole json structure returned by the **inspect** command and try to get them using the Go template format.

## Explore the other commands of the container's API

All the commands linked to the container can be listed with

```.term1
docker container --help
```

We have already seen some of them and will see some other ones in the following but feel free to test them by yourself and to experiment with some fun container commands and features.

## Understand the container layer

The container layer is the layer created when a container is run. This is the layer in which the changes applied are stored.
This layer is deleted when the container is removed and thus cannot be used for persistent storage.

We will start by running a container in interactive mode based on the **ubuntu** image.

```.term1
docker container run -ti ubuntu
```

Note: you can notice here that we do not have any error message as this was the case when we ran our first alpine container. The reason for this is because ubuntu does have a default command **bash** that is specified. The **bash** command with the **-ti** option enables us to get into an interactive shell within this container.

**figlet** is a package that takes a text as input and displays the same text in an Ascii-art format. By default this package is not installed in the ubuntu image, but let's check that.

```.term1
figlet
```

You should get something like

```
bash: figlet: command not found
```

We will update the packages and install figlet then

```.term1
apt-get update -y
apt-get install figlet
```

Make sure figlet is correctly installed

```.term1
figlet Holla
```

You should get a nicely formated output

```
 _   _       _ _
| | | | ___ | | | __ _
| |_| |/ _ \| | |/ _` |
|  _  | (_) | | | (_| |
|_| |_|\___/|_|_|\__,_|
```

You can now exit the container
```.term1
exit
```

We will now run a new container using the **ubuntu** image.

```.term1
docker container run -ti ubuntu
```

Is **figlet** package still there ? Let's figure this out.

```.term1
figlet
```

You should get an error message like the following.

```
bash: figlet: command not found
```

Can you explain why ?

In fact, this new ubuntu container is different from the previous one, the one in which figlet was installed. Both containers have their own container's layer.
Remember, the container's layer is the read-write layer created when a container is run, it's the place where changes done within the container are saved.

Let's exit from this container.

```.term1
exit
```

We can list all the running container.

```.term1
docker container ls
```

And then all the container existing on the host.

```.term1
docker container ls -a
```

From this list, get the id of the container in which we installed the figlet package and restart the container using the 'start' command.

```
docker container start CONTAINER_ID
```

Run an interactive shell in this container. We will use the **exec** command to do so.

```
docker container exec -ti CONTAINER_ID bash
```

Verify figlet is present in this container.

```.term1
figlet still there !
```

If you get the funny output, everything is fine.

We can now exit the container once again.

```.term1
exit
```

## Cleanup

We will now remove all the containers from the machine. There should not be any container in the running state though.
Let's check that.

```.term1
docker container ls -a
```

If we had the -q option to the previous command, we get only the ID of the container.

```.term1
docker container ls -aq
```

This is really handy when we need to remove several containers at the same time as we can feed the **rm** command with this list of ids.

```.term1
docker container rm -f $(docker container ls -aq)
```

There should not be any more container on the host.

```.term1
docker container ls -a
```

## What we seen in this lab

We have started to play with containers and to understand the container layer, the read-write layer that is added to each container that is ran. We also started to play with the container API and the commands used the most (run, exec, ls, rm, inspect).

{:.quiz}
Which command helps you access the commandline on a running container?
- ( ) docker container ls
- ( ) docker container inspect
- ( ) you can't access the commandline of a running container
- (x) docker container exec

{:.quiz}
True or false: Once a container stops it is removed from the system?
( ) True
(x) False
