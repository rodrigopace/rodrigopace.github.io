---
layout: page
title: Episode 15 - Getting to Know Docker Hub

---

 15 - Getting to Know Docker Hub

<div class="video-container"><iframe width="560" height="315" src="https://www.youtube.com/embed/LRpYEK270n4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

Welcome to Episode 15 of the “Head in the Clouds” Video Series. I am Ken Hartman, a SANS Certified Instructor and content creator for the SANS Cloud Security Curriculum.

Today’s episode is titled: “Getting to Know Docker Hub”

Lately we have been talking about Docker containers. In [Episode 13](./episode13) we covered some Docker basics and then in [Episode 14](./episode14) we created and tested an image that contained the command line interfaces for AWS, Azure, and GCP. In this episode, we will be pushing the image up to Docker Hub and setting up an integration with Github, so that when we push a change to the main branch of our git repo, Docker Hub will build a new image based on our latest changes. While we are at it, we will turn on vulnerability scanning.

# Setup

We will assume that you have Docker community engine installed on an Ubuntu 20.01 machine. Refer to [Episode 13](./episode13) for instructions on installing Docker. **SHORTCUT:** Launch an Ubuntu 20.04 VM and run the commands found in this [Gist](https://gist.githubusercontent.com/Resistor52/ba603b268d67239089902b515023e083/raw/dd78549a8ebd7b5a43d56ddf322b615fe6d2a630/gistfile1.txt).

In Episode 14, we created a Dockerfile for our "container of cloud tools." To pick up where we left off last episode, recreate that Dockerfile on your Ubuntu VM, using a text editor. For your convenience, you can just copy and paste this content, right from the Show Notes website:

```
FROM ubuntu

# Add the Non-privileged user
RUN useradd -m hitc && apt update && apt install -y sudo && echo "hitc ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
USER hitc
WORKDIR /home/hitc

# AWS CLI
RUN sudo apt update && sudo apt install -y curl unzip sudo && curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && unzip awscliv2.zip && sudo ./aws/install && rm awscliv2.zip

# Azure CLI
RUN sudo apt update && sudo apt install -y ca-certificates curl apt-transport-https lsb-release gnupg && curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null && AZ_REPO=$(lsb_release -cs) && echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | sudo tee /etc/apt/sources.list.d/azure-cli.list && sudo apt update && sudo apt install -y azure-cli

# GCloud CLI
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && sudo apt update -y && sudo apt install google-cloud-sdk -y

```

Before we can jump into our new content, let's build it and run it --just to make sure everything is fine and dandy.

```
docker build -t hitc-tools .
docker run --rm -it hitc-tools

```

When the prompt changes to "hitc@<some random hex string>" we know that we are good to go. Type `exit` to exit the container.

# Introducing Docker Hub

Navigate over to [hub.docker.com](https://hub.docker.com/) and sign up, assuming that you do not have an account yet. Note that the second half of what we will be covering during today's episode assumes that you have a paid subscription  (Pro, Team or Business). The Pro subscription is $5 per user per month and that is what I have, but I paid the annual fee which is $60/year.

Now that you have a Docker Hub account, create a repository by clicking the blue button at the top right side of the web page. Name your new repository `hitc-tools` and set the visibility to "public." You can leave the description blank for now and leave the Build Settings alone for now. We will be setting that up in a little bit.

Once the repository has been created in Docker Hub, we can see a black box titled "Docker commands." This box contains the docker command that we will need to use to push our image up to Docker Hub.

# Push the Image to Docker Hub

Before we can push the image we need to tag it with our repository name and log into Docker Hub from the VM. Let's start by looking at the images that we have on the system:

```
ubuntu@ubuntu:~$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
hitc-tools   latest    b677d5555ad4   29 minutes ago   2.04GB
ubuntu       latest    ba6acccedd29   8 days ago       72.8MB
```

Our image is "hitc-tools" but it needs to have our Docker Hub username as part of the tag. my Docker Hub user name is "resistor52" so I will use the following command to tag my image:

```
docker tag hitc-tools resistor52/hitc-tools
```

Just remember to replace "resistor52" with your Docker Hub username.

Now when we run `docker images` we see two lines with the same Image ID and we see that an entry with our user name has been added.

Now, to login to docker hub. Type `docker login` and provide your Docker Hub credentials:

```
ubuntu@ubuntu:~$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: resistor52
Password:
WARNING! Your password will be stored unencrypted in /home/ubuntu/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

Cool, now we can push our image up to the Docker Hub repo:

```
docker push resistor52/hitc-tools:latest
```

Again, remember to change "resistor52" to your Docker Hub user name.

It will take a little bit while each layer is pushed up, but eventually, you will get something that looks like:

```
ubuntu@ubuntu:~$ docker push resistor52/hitc-tools:latest
The push refers to repository [docker.io/resistor52/hitc-tools]
0dfa38321923: Pushed
539315a077e7: Pushed
7d4d4a4638fc: Pushed
9bcd961d2190: Pushed
9f54eef41275: Mounted from library/ubuntu
latest: digest: sha256:a3c0d7b00e009b34b1e998b0ba33f431f666c7e8cba94ff628765b6fea29abd6 size: 1379
```

Now, refresh the Docker Hub webpage that is showing your repository and you will see the image with the 'latest' tag is showing.

Click on the "latest" tag as it is a hyperlink. This new page shows the layers. if we select a layer, we can see the commands that are part of the layer.

# Pull the Image from Docker Hub

Now that we have stored the image in Docker Hub, we can pull it down whenever we need it, to any system that has Docker installed. And since it is a public repository, we don't even need to be logged in.

Run `docker logout` to prove this.

Next, let's blow away all images and containers on our VM. Type `docker system prune --all` and then hit "y" when prompted.

This will output something similar to the following:

```
ubuntu@ubuntu:~$ docker system prune  --all
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all images without at least one container associated to them
  - all build cache

Are you sure you want to continue? [y/N] y
Deleted Images:
untagged: hitc-tools:latest
untagged: resistor52/hitc-tools:latest
untagged: resistor52/hitc-tools@sha256:a3c0d7b00e009b34b1e998b0ba33f431f666c7e8cba94ff628765b6fea29abd6
deleted: sha256:b677d5555ad46da76a9a52a05f0186caaf0f46239a06f14054bff87dd59719e3
deleted: sha256:5df3c35f95a167d7be02a0a5abe40bb0a5ce04e9fa33eb424db496f2700e55e2
deleted: sha256:047b4badea5681a6ff536889cdc2309b2107f608872a742a8d2620ab4cc16859
deleted: sha256:d5c6aaf4dc01584522df7cbba592a19dac9abaf460981d226ea9afada83a5f62
deleted: sha256:c6f39dbccd2a62e18fa599ed8d00780afec79d95b14f88764cbfa651b8ea851e
deleted: sha256:c82c31d6df9b81a168c070020f2d853563fad097a8d9e850f9650bb0b3c6b0f9
deleted: sha256:4722de1ae57d6272eb5ce34f2af07b39515f773f75e32011bd4d047a9cefafe9
deleted: sha256:2303e9689e8eb1687ff68d49fdc202b6d42cde3c4c6f2f7638b1c5d066930bdb
deleted: sha256:678d0fff5a4af4d5ce111b5a59e8d28bdae57762ff412615949603dcf1b1b063
deleted: sha256:79535afc33be621b5a0f0ae5df62911aac835b3ac20dee3508a4a4066b9a8395
untagged: ubuntu:latest
untagged: ubuntu@sha256:626ffe58f6e7566e00254b638eb7e0f3b11d4da9675088f4781a50ae288f3322
deleted: sha256:ba6acccedd2923aee4c2acc6a23780b14ed4b8a5fa4e14e252a23b846df9b6c1
deleted: sha256:9f54eef412758095c8079ac465d494a2872e02e90bf1fb5f12a1641c0d1bb78b

Total reclaimed space: 2.04GB
```

We can verify that we do not have any images on the local system:

```
ubuntu@ubuntu:~$ docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```

Ok, lets pull down our image:

```
docker pull resistor52/hitc-tools:latest
```

Now when we run `docker images` we can verify that we have a local copy:

```
ubuntu@ubuntu:~$ docker images
REPOSITORY              TAG       IMAGE ID       CREATED             SIZE
resistor52/hitc-tools   latest    b677d5555ad4   About an hour ago   2.04GB
```

And we can run it:

```
docker run --rm -it resistor52/hitc-tools
```

# Set up Github Integration

Wouldn't it be cool if we could connect a Github code repo with our Docker Hub image repo such that every time we committed a code change to our main branch, Docker Hub would automatically build the image for us?

Well, we will be doing that next. Note that this capability assumes that you have a paid Docker Hub subscription.

Log into your Github account and create a new **public** Git Repo called "hitc-tools." Use the following blurb for the description:

 ```
A Docker container of cloud tools. Includes the AWS, Azure, and GCloud command line interfaces.
 ```

Click the "Add a README file" and chose the "MIT License." Click the green "Create Repository" button.

Next, let's add our Dockerfile to the Git Repo. In the Web UI, click the "Add file" button and select "Create new file." Call the new file Dockerfile and paste in the following contents:

```
FROM ubuntu

# Add the Non-privileged user
RUN useradd -m hitc && apt update && apt install -y sudo && echo "hitc ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
USER hitc
WORKDIR /home/hitc

# AWS CLI
RUN sudo apt update && sudo apt install -y curl unzip sudo && curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && unzip awscliv2.zip && sudo ./aws/install && rm awscliv2.zip

# Azure CLI
RUN sudo apt update && sudo apt install -y ca-certificates curl apt-transport-https lsb-release gnupg && curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null && AZ_REPO=$(lsb_release -cs) && echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | sudo tee /etc/apt/sources.list.d/azure-cli.list && sudo apt update && sudo apt install -y azure-cli

# GCloud CLI
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && sudo apt update -y && sudo apt install google-cloud-sdk -y
```

Click the "Commit new file" button at the bottom of the page.

Now that we have a Git Repo with our Dockerfile, lets connect that to Docker Hub.

Back in the Docker Hub web page, looking at the "hitc-tools" repository, click on the "Builds" tab. Click the "Link to Github" button. This will open up a new page. On the line that lists Github, click the "Connect" hyperlink and link your Github account to Docker Hub.

Click the button again to inform Docker Hub of which Git Repo to use.

* Set the Source Repository. (In my case it is "Resistor52 / hitc-tools")
* Change the Autotest setting to "Internal Pull Requests"
* Change the Build Rules so that the "Source" field reads "main" instead of "master" **NOTE:** The default branch in github for new repos in now "main" so as to be more inclusive.
* Click "save"

Great. Now let's make some changes to our Dockerfile in Github and verify that Docker Hub gets triggered to build a new image when we commit the change to Github.

Using the Web UI for Github, edit the Docker file. We want to make two changes. First, we want to install `git`, so add that to line 4 after the word "sudo." While we are at it, let's use the line continuation character "`\`" to make our code more readable.  

Once your code looks like the following, go ahead and commit the change:

```
FROM ubuntu

# Add the Non-privileged user
RUN useradd -m hitc && apt update && apt install -y sudo git \
&& echo "hitc ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
USER hitc
WORKDIR /home/hitc

# AWS CLI
RUN sudo apt update && sudo apt install -y curl unzip sudo \
&& curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" \
&& unzip awscliv2.zip && sudo ./aws/install && rm awscliv2.zip

# Azure CLI
RUN sudo apt update && sudo apt install -y ca-certificates curl apt-transport-https lsb-release gnupg \
&& curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor \
| sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null && AZ_REPO=$(lsb_release -cs) \
&& echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" \
| sudo tee /etc/apt/sources.list.d/azure-cli.list && sudo apt update && sudo apt install -y azure-cli

# GCloud CLI
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" \
| sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list \
&& curl https://packages.cloud.google.com/apt/doc/apt-key.gpg \
| sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - \
&& sudo apt update -y && sudo apt install google-cloud-sdk -y
```

Back in the Docker Hub Web UI, looking at the Builds tab. Click refresh. After a few minutes, you should now see "IN PROGRESS" under the "Latest Build Status" column. It will take a while, but eventually Docker Hub will finish building the new image.

Notice the Bar Charts under the "Build Activity" in the web form.

# Test the New Image

Once the latest Build status reads "SUCCESS" we can test our new image. Back on the VM, let's run the new image right from the repo.

First, to demonstrate this, lets zap everything local using `docker system prune --all`

Now run a container based on our Docker Hub image:

```
docker run --rm -it resistor52/hitc-tools
```  

Note that the output reads "unable to find image locally" so docker has to pull it down before docker can run it.:

```
ubuntu@ubuntu:~$ docker run --rm -it resistor52/hitc-tools
Unable to find image 'resistor52/hitc-tools:latest' locally
latest: Pulling from resistor52/hitc-tools
7b1a6ab2e44d: Pull complete
1b60658ffa6f: Pull complete
4f4fb700ef54: Pull complete
aa816e75ae95: Pull complete
1aded45ec442: Pull complete
907e0e173b79: Pull complete
Digest: sha256:d0039763ccc5c07fd905fe5ef5aa053fe624b63a2a43225c69efdd5498a4b429
Status: Downloaded newer image for resistor52/hitc-tools:latest
hitc@0a84b95a1da9:~$
```

Since we have the container's prompt, let's see if the container has git installed. Run `git --version`:

```
hitc@0a84b95a1da9:~$ git --version
git version 2.25.1
```

Awesome.

# Enable Vulnerability Scanning

Another amazing feature of the Docker Hub paid subscriptions is image vulnerability scanning. let's enable that next. Back in the Docker Hub Web UI, click on the "General" tab under our "hitc-tools" repository.

Right in the middle of the screen, notice that there is a message that reads "VULNERABILITY SCANNING - DISABLED." Click the enable hyperlink button that is right under the message. This brings you to the "Settings" tab. Click the blue "Enable Image Scanning" button.

# Make another Change to the Dockerfile to Trigger a Vuln Scan

Let's make one more change to the Dockerfile over in Github. Not sure if you noticed, but the shell for our ubuntu user is `sh` and perhaps we want it to be `bash`

Lets change lines 1-7 to read:

```
FROM ubuntu

# Add the Non-privileged user
RUN useradd -s /bin/bash -m hitc && apt update && apt install -y sudo git && echo "hitc ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
USER hitc
WORKDIR /home/hitc
SHELL ["/bin/bash", "-c"]
```

And commit the change. This will trigger Docker Hub to perform another build, this time with a Vulnerability scan.

Since this will take a while, now is a great time to refill your coffee.

After our coffee break, when we take a look at the General Tab, we can see that Snyk found some medium and low priority vulns. Click on the colored boxes and it will bring you to the Vulnerabilities tab.

We can click on most of the results to get additional details and there is also a link to read more at the snyk.io web page.

Hmm, how come there are so many vulnerabilities? Even though it is an recent ubuntu 20.04 image that we used as our base image, it turns out that it is wrong to assume that it was free of known vulnerabilities. Good thing we checked!

Drilling into the vulnerabilities on the Snyk website, it looks like there not any patches (yet) for the items that were found.

This is further confirmed by running the following command at the interactive container's prompt:

```
sudo apt update && sudo apt upgrade -y
```

And we get:

```
Calculating upgrade... Done
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```

Had this manual step found patches to apply, it would have been added to the Dockerfile.

At this point, we need to decide if we are going to accept the risk. Considering that this container is not running internet facing services and will only run code that I, as the system owner control, I will accept the risk. You, my dear listener, have to make a similar decision for yourself. Remember, known risks are always better than unknown risks.

Always remember that security research is unending and new vulnerabilities are found on a frequent basis, so it is important to build a new image on a regular basis to ensure that it is as patched as possible. On top of that, the command line interfaces are updated frequently by each of the cloud service providers. Kicking off a new build ensures that the image has the most up-to-date functionality in the CLI's as well.

## Wrap up

Well, there you go. We pushed our hitc-tools container image up to Docker Hub, demonstrated how we can pull it down and run it from anywhere. Next, we set up a code repo in Github and linked it to our Docker Hub image repo. This way, as we make changes to the Git Repo, it will trigger Docker Hub to build a new image. Next we enabled vulnerability scanning, powered by Snyk--and discussed vulnerability remediation.   

If you have thoughts or comments on today’s episode, feel free to chime in on the comments for this YouTube video.

If you appreciate this video and want to see more like it, be sure to give it a "thumbs up."

Stay tuned for another installment of “Head in the Clouds” as announcements of new episodes are made on the SANS Cloud Security Twitter feed.

Meanwhile, be sure to check out the other great videos on the [SANS Cloud Security YouTube Channel](https://www.youtube.com/c/SANSCloudSecurity).

Take care.
