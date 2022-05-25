---
layout: page
name: Episode 14 - Make a Container of Cloud Tools

---

# Episode 14 - Make a Container of Cloud Tools

<div class="video-container"><iframe width="560" height="315" src="https://www.youtube.com/embed/pr4Ni483CM0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

Welcome to Episode 14 of the “Head in the Clouds” Video Series. I am Ken Hartman, a SANS Certified Instructor and content creator for the SANS Cloud Security Curriculum.

Today’s episode is titled: “Make a Container of Cloud Tools”

This Episode builds on Episode 13, "Docker Jumpstart." Now that we have learned some Docker basics, let's build a useful image that we can use in future _Head in the Clouds_ Episodes. We will build a container image that has each of the command line interfaces for AWS, Azure, and GCP.

# Setup

For this episode, we will assume that you have Docker community engine installed on an Ubuntu 20.01 machine. Refer to [Episode 13](./episode13) for instructions on installing Docker.

# Create the Initial "HitC Tools" Image with just the AWS CLI

A Docker image is created using a Dockerfile, so to start, add a line to a new Dockerfile to specify that our base image will be `ubuntu`

```
echo 'FROM ubuntu' > Dockerfile
echo " " >> Dockerfile
```

Next, we need to look up the commands to install the AWS CLI to Ubuntu.

According to [https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html) The commands are:

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

But let's test them in a container based on our "ubuntu" image to see if we are missing any dependencies. Spin up the container in interactive mode:

```
docker run --name dev -it ubuntu
```

and then paste in the first command:

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

We see that we get an error:

```
root@b0dabf129639:/# curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
bash: curl: command not found

```

As suspected, curl is not installed yet. Let's install it:

```
apt update
apt install -y curl

```

Ok, try the curl command again:

```
root@b0dabf129639:/# curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 42.4M  100 42.4M    0     0  50.3M      0 --:--:-- --:--:-- --:--:-- 50.3M
```

Good, try the next command:

```
unzip awscliv2.zip
```

It is not installed either, so install it:

```
apt install -y unzip

```

Now we can unzip the file that we downloaded to our container:

```
unzip awscliv2.zip

```

At this point, we can try to install the AWS CLI:

```
sudo ./aws/install
```
We get:

```
root@2883310a829c:/# sudo ./aws/install
bash: sudo: command not found
```

Oops, `sudo` is not installed either. Let's add it as well:

```
apt install -y sudo

```

And now, finally, we can run the AWS CLI installer:

```
sudo ./aws/install

```

And we get:

```
root@2883310a829c:/# sudo ./aws/install
You can now run: /usr/local/bin/aws --version
```

To test that the CLI was properly installed, just run:

```
aws --version
```

And we should get something like:

```
root@2883310a829c:/# aws --version
aws-cli/2.2.45 Python/3.8.8 Linux/5.4.0-1045-aws exe/x86_64.ubuntu.20 prompt/off
```

All right, hence we conclude that the full set of commands to install the AWS CLI are:

```
apt update
apt install -y curl unzip sudo
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
rm awscliv2.zip
```

For good housekeeping, we removed the zip file as it is no longer necessary. Also, since these commands were executed as root, we technically did not need to install the CLI using `sudo` however, toward the end of the episode we will be modifying our container so that we can use the CLI tools from a non-privileged user and that will require that we create our image with `sudo` installed.

Type `exit` to exit the interactive container.

In a Dockerfile, it is considered a best practice to separate related commands with `&&` which means _execute the following command only if the previous command ran successfully_ (returned an exit code of zero).

Let's add this to our Dockerfile as a RUN command:

```
echo "# AWS CLI" >> Dockerfile
echo 'RUN apt update && apt install -y curl unzip sudo && curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && unzip awscliv2.zip && sudo ./aws/install && rm awscliv2.zip' >> Dockerfile
echo " " >> Dockerfile

```

Every RUN command creates a new layer (or intermediate image) and that's why related commands are grouped into a single RUN command. See [https://docs.docker.com/engine/reference/builder/#run](https://docs.docker.com/engine/reference/builder/#run) to learn more.

Now let's test our Dockerfile. First by building our image:

```
docker build -t hitc-tools .
```

and then by running it:

```
docker run --rm  -it hitc-tools bash
```

Now we should be able to run `aws --version` and see a result similar to:

```
root@5fe7341eff0d:/# aws --version
aws-cli/2.2.43 Python/3.8.8 Linux/5.4.0-1045-aws exe/x86_64.debian.10 prompt/off
```

As before, type `exit` to terminate the container.

# Add the Azure CLI to the Image

Now to repeat the process for the Azure CLI:

From the [Azure documentation](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt), we can see that the commands that we need to use are:

```
sudo apt-get update
sudo apt-get install ca-certificates curl apt-transport-https lsb-release gnupg
curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null
AZ_REPO=$(lsb_release -cs)
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | sudo tee /etc/apt/sources.list.d/azure-cli.list
sudo apt-get update
sudo apt-get install azure-cli
```

For the sake of space and consistency, let's use the short form of `apt` versus `apt-get` and add the `-y` switch to all of the install commands to make them non-interactive:

```
sudo apt update
sudo apt install -y ca-certificates curl apt-transport-https lsb-release gnupg
curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null
AZ_REPO=$(lsb_release -cs)
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | sudo tee /etc/apt/sources.list.d/azure-cli.list
sudo apt update
sudo apt install -y azure-cli
```

Let's append that to our Dockerfile as a RUN command, separating each of the above commands using `&&`:

```
echo "# Azure CLI" >> Dockerfile

echo 'RUN sudo apt update && sudo apt install -y ca-certificates curl apt-transport-https lsb-release gnupg && curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null && AZ_REPO=$(lsb_release -cs) && echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | sudo tee /etc/apt/sources.list.d/azure-cli.list && sudo apt update && sudo apt install -y azure-cli' >> Dockerfile

echo " " >> Dockerfile

```

Now, test our latest Dockerfile:

```
docker build -t hitc-tools .
docker run --rm  -it hitc-tools bash
```

Once we get the prompt from the container, we can run `az --version` and the output will look something like:

```
ubuntu@ubuntu:~$ docker run --rm -it hitc-tools bash
root@963a51baed25:/# az --version
azure-cli                         2.29.0

core                              2.29.0
telemetry                          1.0.6

Python location '/opt/az/bin/python3'
Extensions directory '/root/.azure/cliextensions'

Python (Linux) 3.6.10 (default, Oct  8 2021, 09:26:22)
[GCC 9.3.0]

Legal docs and information: aka.ms/AzureCliLegal


Your CLI is up-to-date.

Please let us know how we are doing: https://aka.ms/azureclihats
and let us know if you're interested in trying out our newest features: https://aka.ms/CLIUXstudy

```

Very good. Exit from the container.

# Add the GCloud CLI to the Image

One more cloud service provider to go--Google Cloud Platform.

The nice thing about the [Google documentation](https://cloud.google.com/sdk/docs/install) is it provides you with the Docker run command!

```
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && apt-get update -y && apt-get install google-cloud-sdk -y

```

Let's use a text editor this time to modify the Docker file. It should look like this when done:

```
FROM nginx

# AWS CLI
RUN  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && unzip awscliv2.zip && sudo ./aws/install && rm awscliv2.zip

# Azure CLI
RUN curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# GCloud CLI
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && apt-get update -y && apt-get install google-cloud-sdk -y
```

Time to test it again:

```
docker build -t hitc-tools .
docker run --rm -it hitc-tools bash
```

When we get the container's prompt, run `gcloud --version` and the results should look like:

```
root@93d257522d99:/# gcloud --version
Google Cloud SDK 360.0.0
alpha 2021.10.04
beta 2021.10.04
bq 2.0.71
core 2021.10.04
gsutil 5.3
```

It works, so exit from the container.

# Create a Non-privileged User for the Image

The cloud command line interfaces are designed to be run as a non-privileged user, so let's make that change to our
Dockerfile. Using a text editor, add the following lines right below the `FROM ubuntu` line at the top of the file:

```
# Add the Non-privileged user
RUN useradd -m hitc && apt update && apt install -y sudo && echo "hitc ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
USER hitc
WORKDIR /home/hitc
```

What do these commands do? First, we are adding a new user called **hitc** complete with a home directory. Next we install `sudo` so that the **sudoers** file is created as part of the install process. lastly, we need to add a line to the **sudoers** file to prevent the prompting for a password when `sudo` is run from the **hitc** user context.  

The USER instruction tells docker to switch to the "hitc" user for the following steps while the WORKDIR instruction changes the working directory. To learn more about these instructions, see the [official documentation](https://docs.docker.com/engine/reference/builder/#user)

Make sure that your Docker file looks like:

```
FROM ubuntu

# Add the Non-privileged user
RUN useradd -m hitc && apt update && apt install -y sudo && echo "hitc ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
USER hitc
WORKDIR /home/hitc

# AWS CLI
RUN sudo apt install -y curl unzip sudo && curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && unzip awscliv2.zip && sudo ./aws/install && rm awscliv2.zip

# Azure CLI
RUN sudo apt update && sudo apt install -y ca-certificates curl apt-transport-https lsb-release gnupg && curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null && AZ_REPO=$(lsb_release -cs) && echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | sudo tee /etc/apt/sources.list.d/azure-cli.list && sudo apt update && sudo apt install -y azure-cli

# GCloud CLI
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && apt-get update -y && apt-get install google-cloud-sdk -y
```

Great, let's test the build. Note that it will take longer because Docker will use cached layers if it can, but since we are now installing the three CLI's under the `hitc` user context, all of the intermediate containers will need to be rebuilt.:

```
docker build -t hitc-tools .

```

Bonk! We have an error:

```
---> Running in 76d84d16d559

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

Reading package lists...
E: Could not open lock file /var/lib/apt/lists/lock - open (13: Permission denied)
E: Unable to lock directory /var/lib/apt/lists/
The command '/bin/sh -c apt update && apt install -y curl unzip sudo && curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && unzip awscliv2.zip && sudo ./aws/install && rm awscliv2.zip' returned a non-zero code: 100
```

Upon inspection of the error we can see that it is in the RUN command that installs the AWS CLI and it is a "Permission denied" error. Since it worked before we changed the execution context to the "hitc" user, it is clear that we are missing some `sudo` commands.

Change the AWS RUN instruction as follows:

```
RUN sudo apt update && sudo apt install -y curl unzip sudo && curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && unzip awscliv2.zip && sudo ./aws/install && rm awscliv2.zip
```

And, let's test it again:

```
docker build -t hitc-tools .

```

Another error:

```
---> Running in eac768a8ebaf
deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main
tee: /etc/apt/sources.list.d/google-cloud-sdk.list: Permission denied
The command '/bin/sh -c echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && apt-get update -y && apt-get install google-cloud-sdk -yI' returned a non-zero code: 1
```

It's the same type of error as before, but this time in the GCloud RUN instruction. Change the last two lines of the Dockerfile to read as follows:

```
# GCloud CLI
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && sudo apt update -y && sudo apt install google-cloud-sdk -y
```

Now it should build without an error:

```
docker build -t hitc-tools .

```

Awesome! Now we can run it and put it thru it's paces:

```
ubuntu@ubuntu:~$ docker run --rm -it hitc-tools bash
hitc@9e703ed5f1de:~$ aws --version
aws-cli/2.2.45 Python/3.8.8 Linux/5.4.0-1045-aws exe/x86_64.ubuntu.20 prompt/off
hitc@9e703ed5f1de:~$ az --version
azure-cli                         2.29.0

core                              2.29.0
telemetry                          1.0.6

Python location '/opt/az/bin/python3'
Extensions directory '/home/hitc/.azure/cliextensions'

Python (Linux) 3.6.10 (default, Oct  8 2021, 09:26:22)
[GCC 9.3.0]

Legal docs and information: aka.ms/AzureCliLegal


Your CLI is up-to-date.

Please let us know how we are doing: https://aka.ms/azureclihats
and let us know if you're interested in trying out our newest features: https://aka.ms/CLIUXstudy
hitc@9e703ed5f1de:~$ gcloud --version
Google Cloud SDK 360.0.0
alpha 2021.10.04
beta 2021.10.04
bq 2.0.71
core 2021.10.04
gsutil 5.3
hitc@9e703ed5f1de:~$
```

Everything looks good!

## Wrap up

Well, there you go. We created a Docker image that has each of the command line interfaces for AWS, Azure, and GCP. Along the way, we performed some troubleshooting and configured the image to execute as a non-privileged user. At this point, you should have confidence creating your own custom docker images.

If you have thoughts or comments on today’s episode, feel free to chime in on the comments for this YouTube video.

If you appreciate this video and want to see more like it, be sure to give it a "thumbs up."

Stay tuned for another installment of “Head in the Clouds” as announcements of new episodes are made on the SANS Cloud Security Twitter feed.

Meanwhile, be sure to check out the other great videos on the [SANS Cloud Security YouTube Channel](https://www.youtube.com/c/SANSCloudSecurity).

Take care.
