# Docker-Atom Tutorial
A simple tutorial to get started with Docker and Atom.

Although Docker is really easy to understand, there are not many didactic resources out there, so I wanted to create something to get everyone started quickly.

### Why Atom?
Atom is one of the best text editors out there. It's cross-platform, modern, approachable, yet hackable. Also, Atom comes with a very large number of packages to customise your editor exactly as you want it, all installable from the GUI.

### Why Docker?
Docker is the world’s leading software container platform. Download a container with your favourite software preinstalled and start coding in seconds.

### Why Atom and Docker?
There's a package in Atom for pretty much everything, including an in-window terminal. This is beyond useful when creating Docker containers.

## Show me.
Sure. I'll need something from you first though... Go to these two links and download + install both Docker and Atom:

Docker: https://docs.docker.com/engine/installation/
Atom: https://atom.io

## What is a Container?
Containers are a way to package software in a format that can run isolated on a shared operating system. Unlike VMs, containers do not bundle a full operating system - only libraries and settings required to make the software work are needed. This makes for efficient, lightweight, self-contained systems and guarantees that software will always run the same, regardless of where it’s deployed.

### Containers vs VM's

![alt text](https://github.com/dformoso/docker-atom-tutorial/blob/master/images/containers_vs_vms.png)

### Containers and VM's

![alt text](https://github.com/dformoso/docker-atom-tutorial/blob/master/images/containers_and_vms.png)

The explanation I've included here is the TL;DR version of Github's Doc page. If you need more detail, navigate to: https://www.docker.com/what-container

### Containers in Linux, Mac and Windows
Docker has been designed to run on Linux, with Ubuntu as the preferred Linux distro.

When installing on the Mac, Docker for Mac uses HyperKit, a lightweight macOS virtualization solution built on the Hypervisor.framework.

When installing on Windows, Docker for Windows uses Hyper-V.

In both Mac and Windows environment, Docker automatically creates and manages one virtual machine running Ubuntu, and all Docker containers run on that VM. This is transparent to the user, as there's no need to manage the VM.

## Docker Commands
There are many Docker commands available, so I'm going to focus only on the minimum to get you going on simple environments. If you want all the commands available, go here: https://docs.docker.com/engine/reference/commandline/docker/#child-commands

### Environment Info
Let's start with a few easy commands.
The following will print your Docker Version, the Images available (might not have any yet), and the Containers running (might not have any yet).

```shell
# Print Docker Version
docker --version
# Print Docker Images (To be used to create Containers)
docker images
# Print Docker Containers
docker ps -a
# List Docker Networks
docker network ls
```
### Downloading your first Docker instance
There are hundreds of Docker images ready to be downloaded. A great place to look for available images is Docker Hub https://hub.docker.com, as well as right here in Github.

To download and run your docker instance, you'll need to run a 'docker run' command. When you run the command, Docker will go and download the image for you. The image will have the following components:

- Base Image (an OS user space minus the kernel)
- Application (NGINX, MySQL, any many, many, many others)
- Additional OS or App Config.

A complete environment? Yes. Sounds to good to be true right? Let's pick an application and test it out: Nginx.

Nginx (pronounced "engine-x") is an open source reverse proxy server for HTTP, HTTPS, SMTP, POP3, and IMAP protocols, as well as a load balancer, HTTP cache, and a web server (origin server). It's a great choice because it's a web server that immediately gives us a Web front-end to go and verify it all installed correctly.

To install NGINX on Debian, and get it ready for use, we could type the following command:

```shell
docker run -itd \
    --restart always \
    --name my-nginx \
    --hostname my-nginx \
    --volume "/tmp/:/tmp/" \
    -p 8080:80 \
  nginx:latest
```

Now go to http://localhost:8080 and you should see the Nginx Welcome screen.

Yeah, that just happened. Just like there's a Docker container for Nginx, we have one for MySQL, Mongo, Jenkins, Splunk, Hadoop, and anything else you can think of.

### What do all those 'docker-run' modifiers mean?
Here's the full reference https://docs.docker.com/engine/reference/run/

In short:

- '-i':    Keep STDIN open even if not attached
- '-t':    Allocate a pseudo-tty
- '-d':    Detached mode. Run the container in the background.

- '--restart':  Restart policy of always so that if the container exits, Docker will restart it.
- '--name':     Name of Docker container to be used to stop, start, remove the container. Name it at will.
- '--hostname': Hostname for the docker instance. Accessible from other Docker instances by its hostname if within a Custom Docker Network.
- '--volume':   Maps Host volumes to Docker volumes so they are one and the same. <local>:<container>
- '-p':         Maps Host ports to Docker ports so they are one and the same. <local>:<container>

- 'nginx:latest': Use the nginx image, referencing the 'latest' version. If the image is not local, it'll get downloaded.

### Accessing your Instances
How can we get onto our Docker instance's shell? Easy:

```shell
docker exec -it my-nginx bash
```

You can even run commands without getting into the shell. This will push the 'rm' command into the nginx instance.

```shell
docker exec -d my-nginx /bin/sh -c 'rm -rf /tmp/*'
```

And you have the added benefit of getting to pick the user under which the command is to be run.

```shell
docker exec --user root -d my-nginx /bin/sh -c 'rm -rf /tmp/*'
```

### Managing your Instances
The following commands are a staple. You can probably guess what they do:

```shell
# Start, Stop and Remove your Instances
docker start my-nginx
docker stop my-nginx
docker rm my-nginx
docker rm --force my-nginx

# List your images and Remove them if needed
docker images
docker rmi nginx
```

### Docker Networking
You can create as many networks as you like in Docker, and connect Containers to those networks.

The following is an example of two docker containers connected to a User Defined Network. UDF's are useful in many ways. One useful tip is that UDF's configure a DNS and update the /hosts file of all containers so every container in the network can access any other using the hostname configured when creating the container.

```shell
# Create a User-Defined bridge network (needed for DNS hostname discovery)
docker network create \
  --driver=bridge \
  --subnet=172.28.0.0/16 \
  --ip-range=172.28.5.0/24 \
  --gateway=172.28.5.254 \
  jupyter-splunk

# Create a docker container running the latest Splunk version.
docker run -itd \
  --restart always \
  --name splunk \
  --hostname splunk \
  --network=jupyter-splunk \
  -p 8000:8000 \
  -e "SPLUNK_START_ARGS=--accept-license" \
  -e "SPLUNK_USER=root" \
  splunk/splunk:latest
  
# Create a docker container running an unauthenticated Jupyter instance.
docker run -itd \
  --restart always \
  --name jupyter \
  --hostname jupyter \
  --network=jupyter-splunk \
  -p 8888:8888 \
  -v "/tmp/:/home/jovyan/" \
  jupyter/tensorflow-notebook:latest \
  start-notebook.sh --NotebookApp.token=''
```

You can now connect to either instance's shell and ping the other using its hostname.

### Manage Docker Networks
Needed commands. Also easy to guess what they do:

```shell
# List all Docker Networks
docker network ls
# Remove Network
docker network rm jupyter-splunk
```

### Create your own Docker images
So far we have been downloading images that others have built, but we can also create our own.

Docker does this by making use of, and building a, Dockerfile. The Dockerfile contains instructions on how to build an image. The image is then available for you to use the 'docker run' command as explained previously.

The steps are really simple:
- Create a file named 'Dockerfile' in some directory and fill that Dockerfile with Dockerfile commands.
- From your terminal, move into the directory where the Dockerfile file sits.
- Run the following command to build your Dockerfile: ```docker build -t <choose_name_for_image> --force-rm=true .```
- The above command will look for a file called 'Dockerfile' in the current directory and build it into a Docker image.
- Check that the image was create successfully: ```docker images```
- Start an instance of your image: ```docker run -itd <other parameters> <your image name>```

Have a look at existing Dockerfile(s) in Github to give you an idea of what they look like:

- Splunk: https://github.com/splunk/docker-splunk/blob/master/enterprise/Dockerfile 
- MySQL: https://github.com/docker-library/mysql/blob/master/8.0/Dockerfile
- Jenkins: https://github.com/jenkinsci/docker/blob/15dc59d7dbd47da5259a50a9ebfa8895d594444f/Dockerfile
- Mongo: https://github.com/docker-library/mongo/blob/00a8519463e776e797c227681a595986d8f9dbe1/3.0/Dockerfile

And become an expert by reading the Dockerfile docs:

- https://docs.docker.com/engine/reference/builder/


### Atom + platformio-ide-terminal package
The most important Atom package to install for this tutorial is platformio-ide-terminal. It'll give you a terminal embedded in your text editor. Install it first by going to Atom's Preferences, select Install +, and search for 'platformio'. Here's a screenshot:

![alt text](https://github.com/dformoso/docker-atom-tutorial/blob/master/images/platformio.png)

## In-window Terminal
After you've installed the platformio-ide-terminal package, you'll now have a '+' button at the bottom-left corner of your screen. Click on it to open a terminal window.

The magic starts with a simple shortcut: Control+Enter on the Mac. It'll send the current line (or selection) from the text editor and run it in the terminal. This is great for scriptingless scriting, as it can save you a lot of time. Look at this screenshot. All commands were run from the text editor into the terminal by pressing Control+Enter three times.

![alt text](https://github.com/dformoso/docker-atom-tutorial/blob/master/images/atom.png)

>Let's try running some Docker commands from out text editor by pressing Control+Enter. I'll leave you >with one more screenshot, before moving to simply posting the commands here. You can do the copy/pasting.

![alt text](https://github.com/dformoso/docker-atom-tutorial/blob/master/images/dockercom.png)


## About Me
>https://www.linkedin.com/in/danielmartinezformoso/
