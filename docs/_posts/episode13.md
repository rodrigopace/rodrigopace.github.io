---
layout: page
name: Episode 13 - Docker Jumpstart

---

# Episode 13 - Docker Jumpstart

<div class="video-container"><iframe width="560" height="315" src="https://www.youtube.com/embed/k2pkDX5o4no" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

Welcome to Episode 13 of the “Head in the Clouds” Video Series. I am Ken Hartman, a SANS Certified Instructor and content creator for the SANS Cloud Security Curriculum.

Today’s episode is titled: “Docker Jumpstart”

Many folks that work in information security know that they should learn about Docker, and keep meaning to do so, but for some reason have not entered the foray. If you fall into that category, well jump right in and follow along with this episode.

## Setup

For this episode, we will assume that you have a fresh Ubuntu 20.01 virtual machine running and have a SSH connection to it. For your reference, this section follows the steps outlined in the [Docker installation documentation](https://docs.docker.com/engine/install/ubuntu/#installation-methods).

First, set up the repo by running the following commands:

```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release
```

Then add Docker’s official GPG key

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

```

Next, set up Docker Engine (Community Edition):

```
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

```

## Explore Docker

Verify that there are no docker images on the local system:

```
sudo docker images
```

Note that if we try to run the `docker images` command with out sudo we get a permissions error:

```
ubuntu@ubuntu:~$ docker images
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/images/json": dial unix /var/run/docker.sock: connect: permission denied
```

We can fix that by adding the "ubuntu" user to the "docker" group:

```
sudo usermod -aG docker ubuntu
newgrp docker       # Trick to load new group permissions
```

Now we can run the Docker version of the classic 'hello world' test:

```
ubuntu@ubuntu:~$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:9ade9cc2e26189a19c2e8854b9c8f1e14829b51c55a630ee675a5a9540ef6ccf
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

```

Let's rerun the `docker images` and see what changed:

```
ubuntu@ubuntu:~$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED      SIZE
hello-world   latest    feb5d9fea6a5   8 days ago   13.3kB

```

Ok, great. We don't need that image around anymore. So what command do we need to delete the image? Run `docker help` to see a list of all the possible docker commands.

It looks like it would be `docker rmi`, so let's try it:

```
ubuntu@ubuntu:~$ docker rmi hello-world
Error response from daemon: conflict: unable to remove repository reference "hello-world" (must force) - container 39c0bbdce667 is using its referenced image feb5d9fea6a5
```

Well, that didn't work. We can't remove the image while there is a container referencing it. Let's see what containers are running, using `docker ps`

```
ubuntu@ubuntu:~$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

Well, that didn't show us what we wanted to see, let's rerun it with the `--all` option:

```
ubuntu@ubuntu:~$ docker ps --all
CONTAINER ID   IMAGE         COMMAND    CREATED              STATUS                          PORTS     NAMES
39c0bbdce667   hello-world   "/hello"   About a minute ago   Exited (0) About a minute ago             nifty_turing
```

The `--all` option shows all containers, not just the running containers. (For more info see [https://docs.docker.com/engine/reference/commandline/ps/](https://docs.docker.com/engine/reference/commandline/ps/))

Now that we know the container id that is referencing our `hello-world` image, we can delete it using the `docker rm` command, as follows:

```
docker rm 39c0bbdce667
```

Now if we rerun the `docker ps --all` command, we see that the container is gone:

```
ubuntu@ubuntu:~$ docker ps --all
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

Once the container is zapped, we can delete the image:

```
ubuntu@ubuntu:~$ docker rmi hello-world
Untagged: hello-world:latest
Untagged: hello-world@sha256:9ade9cc2e26189a19c2e8854b9c8f1e14829b51c55a630ee675a5a9540ef6ccf
Deleted: sha256:feb5d9fea6a5e9606aa995e879d862b825965ba48de054caab5ef356dc6b3412
Deleted: sha256:e07ee1baac5fae6a26f30cabfe54a36d3402f96afda318fe0a96cec4ca393359
```

And just to confirm that it is actually deleted:

```
ubuntu@ubuntu:~$ docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```

Great, now let's pull down a different image. Let's pull down the official Ubuntu container image from Docker hub:

```
ubuntu@ubuntu:~$ docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
f3ef4ff62e0d: Pull complete
Digest: sha256:44ab2c3b26363823dcb965498ab06abf74a1e6af20a732902250743df0d4172d
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

We can see that it was downloaded:

```
ubuntu@ubuntu:~$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
ubuntu       latest    597ce1600cf4   46 hours ago   72.8MB
```

Next, let's run the container in an interactive mode so that we execute various commands from the bash shell within the container. To do that, we need two options:

* -i --> Interactive, Keep STDIN open even if not attached
* -t --> Allocate a pseudo-TTY

While we are at it, we will assign a name to the container with the `--name` option.

So our command becomes:

```
docker run --name test -it ubuntu bash
```

And then we see our prompt change. Now we can run some commands that are executed inside the container:

```
ubuntu@ubuntu:~$ docker run --name test -it ubuntu bash
root@a843c11e5313:/# whoami
root
root@a843c11e5313:/# hostname
a843c11e5313
root@a843c11e5313:/# pwd
/
root@a843c11e5313:/# ls /
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@a843c11e5313:/# ls /home
root@a843c11e5313:/# exit
exit
ubuntu@ubuntu:~$
```

The `exit` command kills the container executing in the foreground and returns us to our original prompt.

List the container:

```
ubuntu@ubuntu:~$ docker ps --all
CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS                     PORTS     NAMES
a843c11e5313   ubuntu    "bash"    4 minutes ago   Exited (0) 3 minutes ago             test
```

We can run the container again by using the `docker start` command, remembering to pass in the interactive option:

```
ubuntu@ubuntu:~$ docker start -i test
root@a843c11e5313:/#
root@a843c11e5313:/# exit
exit
ubuntu@ubuntu:~$
```  

If we use the `--rm` option with the run command, it cleans up the container when the command passed into the container exits:

```
docker run --name test2 -it --rm ubuntu ls
```

In this case, the ls command runs in the container, and then the container exits:

```
ubuntu@ubuntu:~$ docker run --name test2 -it --rm ubuntu ls
bin   dev  home  lib32  libx32  mnt  proc  run   srv  tmp  var
boot  etc  lib   lib64  media   opt  root  sbin  sys  usr
ubuntu@ubuntu:~$
```

Note that we named the container "test2" but when we run `docker ps --all` we do not see it listed, because of the `--rm` switch.

```

ubuntu@ubuntu:~$ docker ps --all
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                         PORTS     NAMES
a843c11e5313   ubuntu    "bash"                   2 hours ago      Exited (0) About an hour ago             test
```

## Create a Custom Image
The most common use case for docker is to run web services, so let's create a simple web server to illustrate some important concepts. First, lets create a basic web page called "index.html" in a directory called "html"

```
mkdir html
echo "Head in the Clouds" > html/index.html

```

With that done, we need to create a `Dockerfile`

Images are typically built from a base image adding and removing stuff as needed. Our base image will be 'nginx' since it already has the web server installed. Then we will copy our "html" folder onto the image stomping on the existing  /usr/share/nginx/html

```
echo 'FROM nginx' > Dockerfile
echo 'COPY html /usr/share/nginx/html' >> Dockerfile

```

That creates our Dockerfile, thanks to the magic of file redirection. Double-checking our Dockerfile:

```
ubuntu@ubuntu:~$ cat Dockerfile
FROM nginx
COPY html /usr/share/nginx/html

```

Now let's build a custom image based on our Dockerfile:

```
docker build -t hitc .
```

Note that the "." indicates "the current directory"

```
ubuntu@ubuntu:~$ docker build -t hitc .
Sending build context to Docker daemon  14.85kB
Step 1/2 : FROM nginx
latest: Pulling from library/nginx
07aded7c29c6: Pull complete
bbe0b7acc89c: Pull complete
44ac32b0bba8: Pull complete
91d6e3e593db: Pull complete
8700267f2376: Pull complete
4ce73aa6e9b0: Pull complete
Digest: sha256:765e51caa9e739220d59c7f7a75508e77361b441dccf128483b7f5cce8306652
Status: Downloaded newer image for nginx:latest
 ---> f8f4ffc8092c
Step 2/2 : COPY html /usr/share/nginx/html
 ---> 995b0de5245b
Successfully built 995b0de5245b
Successfully tagged hitc:latest
ubuntu@ubuntu:~$

```

Now when we run `docker images` we see two new images along with the 'ubuntu' image from before:

```
ubuntu@ubuntu:~$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
hitc         latest    995b0de5245b   44 seconds ago   133MB
ubuntu       latest    597ce1600cf4   47 hours ago     72.8MB
nginx        latest    f8f4ffc8092c   4 days ago       133MB
```

The "hitc" is the image that we just created, and it is derived from the "nginx" image referenced in the Dockerfile. Because it was referenced, Docker pulled down a local copy from Docker Hub.

Ok, lets spin up a container from this image and see if we can hit with curl:

```
docker run --name episode13 -d -p 8080:80 hitc
```

And we get:

```
ubuntu@ubuntu:~$ docker run --name episode13 -d -p 8080:80 hitc
61747901bb594c88ec03736a50886985b404aca4a4a3d85b2b9608d190a8c626
ubuntu@ubuntu:~$ curl localhost:8080
Head in the Clouds
ubuntu@ubuntu:~$

```

Note that the `-d` option ran the container in the background and the `-p` option maps port 8080 on the host to port 80 in the container.

We can run `docker ps` to see that it is still running in the background:

```
ubuntu@ubuntu:~$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                   NAMES
61747901bb59   hitc      "/docker-entrypoint.…"   5 minutes ago   Up 5 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp   episode13
```

If we want to see what processes are running inside the container, we can use `docker top [CONTAINER]` like so:

```
ubuntu@ubuntu:~$ docker top episode13
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                10642               10616               0                   01:47               ?                   00:00:00            nginx: master process nginx -g daemon off;
systemd+            10693               10642               0                   01:47               ?                   00:00:00            nginx: worker process

```

If we stop the container, we will no longer be able to connect to port 8080:

```
ubuntu@ubuntu:~$ docker stop episode13
episode13
ubuntu@ubuntu:~$ curl localhost:8080
curl: (7) Failed to connect to localhost port 8080: Connection refused
```

## Wrap up

Well, there you go. We installed Docker Engine Community Edition, learned some basic Docker commands, and launched a very basic website. Hopefully this video has removed any reasons you may have had to procrastinate and inspired you to jump in and play. Of course there is much more to learn and many additional tutorials that are just an internet search away. In our next episode, we will build on this foundation and create a Docker image that will have utility for members of this audience as we continue to explore the cloud.   

If you have thoughts or comments on today’s episode, feel free to chime in on the comments for this YouTube video.

If you appreciate this video and want to see more like it, be sure to give it a "thumbs up."

Stay tuned for another installment of “Head in the Clouds” as announcements of new episodes are made on the SANS Cloud Security Twitter feed.

Meanwhile, be sure to check out the other great videos on the [SANS Cloud Security YouTube Channel](https://www.youtube.com/c/SANSCloudSecurity).

Take care.
