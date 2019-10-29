# Installing Singularity on Mac/Windows

Singularity is not compatible with OSX/Windows.
The advised way to create your image on those OS is to use a linux virtual machine.

One particular practical way is to use the vagrant option: 
https://sylabs.io/guides/3.0/user-guide/installation.html#install-on-windows-or-mac (for mac)
and for windows: https://sylabs.io/guides/3.0/user-guide/installation.html#install-on-windows-or-mac

Note for Mac, a native client is also possible: https://sylabs.io/singularity-desktop-macos/
but it is still in beta (and it offers limited possiblity including the impossiblity to build image locally)

# Installing Singularity On Linux

Follow the following instructions: https://sylabs.io/guides/3.0/user-guide/

# first check

If everything went according to plan, you now have a working installation of Singularity.
Simply typing `singularity` will give you a summary of all the commands you can use.
Typing `singularity help <command>` will give you more detailed information about running an individual command.

You can test your installation like so:

```
$ singularity run docker://godlovedc/lolcow
```

You should see something like the following.

```
Docker image path: index.docker.io/godlovedc/lolcow:latest
Cache folder set to /home/ubuntu/.singularity/docker
[6/6] |===================================| 100.0%
Creating container runtime...
 _______________________________________
/ Excellent day for putting Slinkies on \
\ an escalator.                         /
 ---------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

Your cow will likely say something different (and be more colorful), but as long as you see a cow your installation is working properly.  

This command downloads, converts, and runs a container from [Docker Hub](https://hub.docker.com/r/godlovedc/lolcow/).
In the next exercise, we will learn how to build a similar container from scratch.
