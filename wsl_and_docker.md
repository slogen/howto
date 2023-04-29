# Installing WSL on Win10, Win11 or Windows Server 2022+
## Enable WSL

In an admin shell: "C:\Windows\system32>"

        dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
        dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
        dism.exe /Online /Enable-Feature /All /FeatureName:Microsoft-Hyper-V

Restart (if it wasn't already installed) 

        shutdown /r /t 0
    
## Default to WSL2

Set WSL version, otherwise we will get error. So, back in your regular user shell: "C:\Users\hej>"

        wsl --set-default-version 2
        wsl --update

# Instal Debian

You could also install some other variant. The details on installing docker will vary.

        wsl --install -d Debian

This will start a linux shell, and prompt you for a UNIX username and password:

        C:\Users\hej>wsl --install -d Debian
        Installing: Debian GNU/Linux
        Debian GNU/Linux has been installed.
        Launching Debian GNU/Linux...
        Installing, this may take a few minutes...
        Please create a default UNIX user account. The username does not need to match your Windows username.
        For more information visit: https://aka.ms/wslusers
        Enter new UNIX username: hej
        New password:
        Retype new password:
        passwd: password updated successfully
        Installation successful!
        hej@HEJ-E14:~$

You can choose any name and password you want. *Neither* will be synced with your windows account.

## Sudo for UNIX user

Setup so your default user can sudo without password:

        hej@HEJ-E14:~$ echo "$USER ALL = NOPASSWD: ALL" | sudo tee /etc/sudoers.d/user_nopass
        
        We trust you have received the usual lecture from the local System
        Administrator. It usually boils down to these three things:

            #1) Respect the privacy of others.
            #2) Think before you type.
            #3) With great power comes great responsibility.

        [sudo] password for hej:
        hej ALL = NOPASSWD: ALL

Here you will get asked to repeat your password, for the last time :)

## Initial docker requisites

Update OS package-lists (update), then upgrade all (dist-upgrade) and install what we need:

        hej@HEJ-E14:~$ sudo apt -y update
        ...
        hej@HEJ-E14:~$ sudo apt dist-upgrade -y
        ...
        hej@HEJ-E14:~$ sudo apt -y install --no-install-recommends apt-transport-https ca-certificates curl gnupg2
        ...

## Install docker

We will be installing from dockers own repository. That depends on our os-release, requires trusting their APT-key, and configuring their repo:

        hej@HEJ-E14:~$ . /etc/os-release
        hej@HEJ-E14:~$ curl -fsSL https://download.docker.com/linux/${ID}/gpg | sudo tee /etc/apt/trusted.gpg.d/docker.asc
        ...
        hej@HEJ-E14:~$ echo "deb [arch=amd64] https://download.docker.com/linux/${ID} ${VERSION_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/docker.list

Now we can update packages again (from the newly configured repo), and then install:

    hej@HEJ-E14:~$ sudo apt -y update
    ...
    hej@HEJ-E14:~$ sudo apt -y install docker-ce docker-ce-cli containerd.io
    ...

### Predictable groups

If we want to share some files between different WSL instances, we need consistent groups, and we need the current user to be in the docker group:

    hej@HEJ-E14:~$ GID=36257
    hej@HEJ-E14:~$ sudo sed -i -e "s/^\(docker:x\):[^:]\+/\1:$GID/" /etc/group
    hej@HEJ-E14:~$ sudo groupmod -g $GID docker
    hej@HEJ-E14:~$ sudo usermod -aG docker $USER

### Legacy IPTABLES (firewall)

Docker still requires legacy iptables command:

    hej@HEJ-E14:~$ sudo update-alternatives --set iptables /usr/sbin/iptables-legacy

### Configure DockerD usable from windows:

So we can reach it from localhost, that is,... outside directly from windows (127.0.0.1)

    hej@HEJ-E14:~$ echo '{ "hosts": [ "tcp://127.0.0.1:2375", "unix:///var/run/docker.sock" ] }' | sudo tee /etc/docker/daemon.json


### Starting docker

    hej@HEJ-E14:~$ service docker status
    Docker is not running ... failed!
    hej@HEJ-E14:~$ sudo service docker start
    Starting Docker: docker.
    hej@HEJ-E14:~$ service docker status

### checking that it runs

We are still in a shell for $USER which is *not* in group docker, so lets get one:

    hej@HEJ-E14:~$ groups
    hej adm cdrom sudo dip plugdev
    hej@HEJ-E14:~$ sudo su $USER
    hej@HEJ-E14:~$ groups
    hej adm cdrom sudo dip plugdev docker
    hej@HEJ-E14:~$ docker ps
    CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

As final validation, lets run the hello-world images:

    hej@HEJ-E14:~$ docker run -ti --rm hello-world
    Unable to find image 'hello-world:latest' locally
    latest: Pulling from library/hello-world
    2db29710123e: Pull complete
    Digest: sha256:4e83453afed1b4fa1a3500525091dbfca6ce1e66903fd4c01ff015dbcb1ba33e
    Status: Downloaded newer image for hello-world:latest

    Hello from Docker!
    This message shows that your installation appears to be working correctly.

    To generate this message, Docker took the following steps:
     1. The Docker client contacted the Docker daemon.
     2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
        (amd64)
     3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
     4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.

    To try something more ambitious, you can run an Ubuntu container with:
     $ docker run -it ubuntu bash

    Share images, automate workflows, and more with a free Docker ID:
     https://hub.docker.com/

    For more examples and ideas, visit:
     https://docs.docker.com/get-started/

# Running from windows

## "Install" docker.exe

@StefanScherer maintains a "docker-cli-builder" repo on Github where you can download a standalone docker.exe (https://github.com/StefanScherer/docker-cli-builder/releases)

Below I am using version 20.10.9 (https://www.virustotal.com/gui/file/da090d8e9504e48cb124e7ee1b453b0f178a2873c02f6b3bcfc2aa0d51b1bd04)

## Running with TCP

    C:\Users\hej\Downloads>docker -H localhost:2375 run -ti --rm hello-world

    Hello from Docker!
    This message shows that your installation appears to be working correctly.

    To generate this message, Docker took the following steps:
     1. The Docker client contacted the Docker daemon.
     2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
        (amd64)
     3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
     4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.

    To try something more ambitious, you can run an Ubuntu container with:
     $ docker run -it ubuntu bash

    Share images, automate workflows, and more with a free Docker ID:
     https://hub.docker.com/

    For more examples and ideas, visit:
     https://docs.docker.com/get-started/

## Persistant DOCKER_HOST

Set the "DOCKER_HOST" environment variable for new shells:

    C:\Users\hej>setx DOCKER_HOST tcp://localhost:2375

Now you must start a *new* shell from the UI, but from that:

    Microsoft Windows [Version 10.0.19044.2846]
    (c) Microsoft Corporation. All rights reserved.

    C:\Users\hej>echo %DOCKER_HOST%
    tcp://localhost:2375

    C:\Users\hej>docker run -ti --rm hello-world

    Hello from Docker!
    This message shows that your installation appears to be working correctly.

    To generate this message, Docker took the following steps:
     1. The Docker client contacted the Docker daemon.
     2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
        (amd64)
     3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
     4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.

    To try something more ambitious, you can run an Ubuntu container with:
     $ docker run -it ubuntu bash

    Share images, automate workflows, and more with a free Docker ID:
     https://hub.docker.com/

    For more examples and ideas, visit:
     https://docs.docker.com/get-started/




