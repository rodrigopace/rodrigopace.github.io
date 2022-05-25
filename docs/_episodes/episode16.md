---
layout: page
title: Episode 16 - Introducing Github Codespaces - Dev Environments as Code

---

 16 - Introducing Github Codespaces - Dev Environments as Code

<div class="video-container"><iframe width="560" height="315" src="https://www.youtube.com/embed/maVJsDAtyzc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

Welcome to Episode 16 of the “Head in the Clouds” Video Series. I am Ken Hartman, a SANS Certified Instructor and content creator for the SANS Cloud Security Curriculum.

Today’s episode is titled: “Introducing Github Codespaces - Dev Environments as Code”

In prior episodes of head in the Clouds, we have talked about the benefits of _Infrastructure as Code_ (IaC) and how we can easily provision cloud resources with a script. However, to deploy the Infrastructure as Code, one must have the necessary dependencies in place such as the credentials and the IaC tool, (such as Terraform) installed. Different development projects require different dependencies so wouldn't it be awesome if we could quickly and easily provision a development environment for each moments before we needed it and tear it down when we are done working for the day?

That is the idea behind _Development Environments as Code_. Dev Environments as Code can be thought of as a special type of Infrastructure as Code. Github Codespaces is one of many service providers that offer Dev Environments as Code, and this service is the topic of today's episode.

Before we jump into the particulars of Github Codespaces, let's answer the question that you may be asking yourself, "I am a security person, not a developer--why do I care about this?" Well, we can expand the idea of Dev Environments as Code to _Security Testing Environments as Code_ or _Learning Environments as Code_ or perhaps it makes more sense to just think in terms of _Environments as Code_. Regardless of your role, I think that the idea of being able to jump right into work is much more appealing than having to invest a lot of setup time before getting into the important work.

# Setup

To use Github Codespaces, you have to have your Github account associated with a Team plan or Enterprise plan that has authorized your Github account to use the Codespaces functionality. The easiest way to set this up if you do not currently have access to Codespaces yet is to subscribe to the Team plan for $4 per user per month, even if it is just for a single month. And yes, you can have a team consisting of a single user. That's what I did, except for I chose the annual billing plan to simplify my book keeping. Go ahead and set up the organization that you plan to associate with your new team account. For example, I called my new organization "Rampant Lab."

Ok, now that you are subscribed and set up your team account and organization, you need to grant your account the permission to use Codespaces. At the Organization level, click on "Setting" and then select "Codespaces" in the left-hand menu. Set the User Permissions to allow at least your Github user account to access Codespaces. Also, set the "Access and security" radio button to "All repositories" for now.

# The Default Codespace

Great, now we can create a new repository that is owned by our new organization. Call the new repo `space1` or something like that. if you want, provide an optional description and leave the repository private. Add a README and a License like "Apache License 2.0" while you are at it. Then click the "Create repository" button.

Now, click the "Add file" button and select "create new file." Call the new file "hello.py" and add the following line of code:

```
print("Hello World!")
```

And then click the "Commit new file" button.

Don't worry if you do not have Python installed, the default Codespace does! Let's check it out.

Click the Green "Code" button and then select the "Codespaces" tab. Note: if you do not see the "Local" and "Codespaces" tab, you are either not in your Organization or did not grant your account access to Codespaces as described in the setup.

Next, click the "New Codespace" button. Choose the "2 core" option and click the "Create codespace" button. Within two or three minutes, the Codespaces service will spin up a container to serve as your environment and the Web browser will load the Microsoft Visual Studio web-based Integrated Development Environment with our repository set as the working directory.

The container that gets spun up is on a dedicated Linux virtual machine so that you can have full access to all of the resources of the virtual machine including full root access to your container.

At the prompt, type `ls` and notice that the three files from the repository are shown. We can even run the python program by typing `python hello.py`

Click on the "hello.py" file in the EXPLORER pane and then edit 'hello.py' by replacing "Hello World!" with "Head in the Clouds." Rerun it to observe the change.

Since our change works, let's commit the change. Run the following commands in the TERMINAL pane:

```
git status
git add hello.py
git commit -m "revised hello.py"
git push

```

We have just pushed the change from our local development environment (which is running in a cloud container) up to our repository in Github. We can look at it in the repository by clicking the blue "Codespace" button in the bottom left-hand corner and selecting the "Go to repository" option.

Notice that the commit comments that read "revised hello.py" show that our last commit was indeed pushed. To get back to our Codespace, click the green "Code" button again and then select the running codespace.

To prove that we have root access, type `sudo su`

very cool! Type `exit` to lower our privilege level back down.

The default Codespaces container image supports many different languages not only Python, but Java, Go, C++ and a few others. You may have noticed that it does not support all of our cloud command line interfaces. To prove it, run the following commands:

```
which aws az gcloud
```

Interesting! The only one that the default image supports is the Azure CLI. Hmm, I wonder who owns Github these days?

Is Terraform installed?

```
which terraform
```

looks like it isn't installed either.

The default image definition is maintained at [https://github.com/microsoft/vscode-dev-containers/tree/main/containers/codespaces-linux](https://github.com/microsoft/vscode-dev-containers/tree/main/containers/codespaces-linux), so consult this repo to learn more about it.

There are many more standard definitions to choose from as well. There can be seen at [https://github.com/microsoft/vscode-dev-containers/tree/main/containers](https://github.com/microsoft/vscode-dev-containers/tree/main/containers)

# Customizing a Codespace with Dotfiles

If we wanted to install the AWS and GCloud command line interfaces on our Codespace environment that we just launched, we could since we have full control, but then we would have to run through the manual install process each time we spun up a Codespace. That would be rather inefficient, besides, we want to be able to hit the ground running, without any additional setup.   

One option to customize our Codespace environment is to use dotfiles. Dotfiles take their name from the fact that many of the files used to configure a Linux or Mac are hidden by virtue of the fact that the first character in their name is a period.

There are a couple of steps to setting up Codespaces to work with dotfiles. First we need to create a **public** repository named "dotfiles" in our **personal** Github account. And then second, we need to enable Codespaces to use this to configure our Codespaces for us.

## Create a Dotfile Repository

In your **personal** github account, create a new public repository named "dotfiles." Next, create a file named `setup.sh` in the new repository that contains the following lines:

```
#!/bin/bash
# Install Terraform
sudo apt update && sudo apt install -y gnupg software-properties-common curl
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt update && sudo apt install terraform
```

## Enable Codespaces to Use Dotfiles

The next step is to enable code spaces to use our dotfiles. Click on your profile photo in the upper right-hand of the Github web page and select "Settings." In the new page that loads, select "Codespaces" in the right-hand menu. Now, click the "Automatically install dotfiles" checkbox.

## Test to Confirm the Dotfiles script executed.

To test that the Dotfiles script will execute, let's create a new code space for our "space1" repository. Navigate back to your organization and select the "space1" repository. This time, instead of  selecting the running code space, click the "Manage all" link. Click the elipsis (the three dots) on the row that lists the running codespace, and then click "Delete."

Now, click the "New Codespace" button and then click "Create codespace."

Once the prompt apears, run the following command to confirm that terraform is installed.

```
which terraform
```

And...bad news...it is not installed. Github has a good troubleshooting document, so let's consult [it](https://docs.github.com/en/codespaces/troubleshooting/troubleshooting-dotfiles-for-codespaces).

To see if our dotfiles were cloned, we can run:

```
ls -la /workspaces/.codespaces/.persistedshare/dotfiles
```

Hmm, it looks like it was cloned. Next, let's check the creation log:

```
less /workspaces/.codespaces/.persistedshare/creation.log
```

Well, it looks like the issue is that our `startup.sh` script is not executable. Unfortunately, this script cannot be made executable via the web UI. So I will clone it down to my local system, use chmod to make the file executable and push it back to the repo. For this episode, I will not be demonstrating how to do that, since this episode is not on Github basics.

We do, however want to test it now that we made this fix. Click the blue Codespaces button in the lower left corner and click "My Codespaces." Delete the active codespace and create a new one.

Now when we run `which terraform` we see that it is indeed installed.

# Customizing Codespaces with a Custom Container

The installation of Terraform after the container spins up using the dotfiles technique is not terribly slow, but if we were to install the cloud CLI's it would add a considerable amount of time--and that is exactly what we are trying to avoid. So this brings us to the other option: using a custom image.

The benefit of using a custom image is that we can install all the software that we need for our development environment and leave anything that we do not need out of it. Fortunately, we already have a container image that has the command line tools for all three cloud service providers as that was the subject of Episode 15.

To experience this, create a new repository named `space2` that is owned by the organization that granted you Codespace access. Leaving it as a private repo is fine and Add a README file. Click "Create repository."

Create a file called `.devcontainer.json` and add in the following content:

```
{
  "name": "HitC Tools",
  "image": "resistor52/hitc-tools:ep15"
}
```

Then commit the change.

**Note:** Even if you followed along with Episode 15 and made a public image in Docker Hub, you may still want to use my image as I have made some changes to the image since publishing that Episode. After completing the lesson in this Episode, you can fork my [hitc-tools repo](https://github.com/Resistor52/hitc-tools) and build your own custom images derived on that.

## Test the Custom Container in Codespaces

To test if Codespaces will use the specified container, spin up a Codespace in this repository after deleting any unneeded running codespaces.

Once the prompt appears and the Dotfiles configuration completes, run the following command:

```
which aws az gcloud terraform
```

and we should see the following output:

```
/usr/local/bin/aws
/usr/bin/az
/usr/bin/gcloud
/usr/bin/terraform
```

## Optimize the Codespace configuration

Great, This worked, but it would be even more efficient if we installed Terraform in the container image and turned off the dotfiles. Generally, it makes sense to build a container that has everything that the entire team needs and to allow each user to make minor customizations via dotfiles in their personal repo.

The Github [Resistor52/hitc-tools](https://github.com/Resistor52/hitc-tools) was updated with the following [commit](https://github.com/Resistor52/hitc-tools/commit/0facb0f2b211aff0577868a70ebf0567dc063915) so that the `resistor52/hitc-tools:ep16` container image would have Terraform installed.

So, lets switch to that image in our `.devcontainer.json` configuration for our `space2` repository. The contents of the `.devcontainer.json` file should now read:

```
{
  "name": "HitC Tools",
  "image": "resistor52/hitc-tools:ep16"
}
```

Turn off the "Automatically install dotfiles" setting in your personal settings and then launch Codespaces again. Notice that the prompt appears in short order and there is not the additional delay of the dotfiles process needing to install terraform.

# Using the CLI Tools

We can confirm that all four of our CLI tools have been installed, including Terraform using:

```
which aws az gcloud terraform
```

Now that we have a basic Codespace configuration, there is still some additional configuration that we will need to do.

First run the following AWS command:

```
aws sts get-caller-identity
```

Notice that we get the following error:

```
Unable to locate credentials. You can configure credentials by running "aws configure"
```

Of course we have to configure our credentials, and while we could do that via the `aws configure` command in the terminal, then we would have to do it every time we needed to spin up a new codespace for this repository.

Fortunately, the solution to this is to use "Codespaces Secrets."

# Codespaces Secrets

Codespaces has a facility to store the secrets that each user needs to work with external services from within their Codespaces. The Secrets are stored by the Codespaces service in an encrypted format and are only exposed to the codespaces that you designate by setting the secret as an environment variable as the container is launched.  

## The AWS Secrets and Parameter

The [AWS Command Line Interface Documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html) specifies various environment variables that can be used to set configurations needed by the CLI. The three that we are interested in are:
* AWS_ACCESS_KEY_ID
* AWS_SECRET_ACCESS_KEY
* AWS_DEFAULT_REGION

The first two are sensitive so we will set them as Secrets and the third one is not sensitive so we will set it via the `.devcontainer.json` file.

Navigate to your personal Github settings by clicking on your avatar in the upper right corner and selecting "settings." Then select "Codespaces in the right-hand menu." The Codespaces secrets section is right below the Dotfiles setting that we were interacting with earlier.

Click the "New secret" button. Type the name of the secret exactly how you want it to appear as an environment variable. Next provide the value of the secret and select the repositories that will need the secret. Note that you can always update the secret value later but you cannot get it to display when you click the "Update" button. You can also add or remove the repositories that the secret is available to at any time.

You should create a new AWS Identity that is dedicated to Codespaces and generate a new AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY. The reason that I recommend a dedicated key is that by using dedicated keys for different purposes, it is actually easier to detect and isolate a key compromise.

For the AWS region, we will add a line to the `.devcontainer.json` file that reads:

```
 "remoteEnv": { "AWS_DEFAULT_REGION":"us-east-1" }
```

So update that file so that it looks like the following when you are done:

```
{
  "name": "HitC Tools",
  "image": "resistor52/hitc-tools:ep16",
  "remoteEnv": { "AWS_DEFAULT_REGION":"us-east-1" }
}
```

When that is done, let's spin up a new codespace for our "space2" repository and let's test it.

Rerun `aws sts get-caller-identity` once the prompt appears. We should get a result that looks like:

```
{
    "UserId": "AIDAEXAMPLEKEY",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/codespaces"
}
```

Awesome. Now run `env | grep AWS` and you should see something like:

```
AWS_DEFAULT_REGION=us-east-1
AWS_SECRET_ACCESS_KEY=GWbYrZZZREDACTEDZZZ21JRxpCfax
AWS_ACCESS_KEY_ID=AIDAEXAMPLEKEY
```

## The Azure Secrets

For Azure, we need to generate a service principal. The easiest way to do this is to log into your Azure portal and use the Cloud Shell and run:

```
az ad sp create-for-rbac
```

And you will get output that looks like the following:

```
In a future release, this command will NOT create a 'Contributor' role assignment by default. If needed, use the --role argument to explicitly create a role assignment.
Creating 'Contributor' role assignment under scope '/subscriptions/911d9999-9999999999-9999a4f55a'
The output includes credentials that you must protect. Be sure that you do not include these credentials in your code or check the credentials into your source control. For more information, see https://aka.ms/azadsp-cli
'name' property in the output is deprecated and will be removed in the future. Use 'appId' instead.
{
  "appId": "e564cf70-c358-40e3-adeb-83d015ebcc10",
  "displayName": "azure-cli-2021-10-23-21-17-11",
  "name": "e5ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ10",
  "password": "DHZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ4U",
  "tenant": "e9439999999999999999999af64c0"
}
```

Note that by default that command created a service principal with the "Contributor" role for the whole subscription. Is it over permissioned? Yes, most likely, and that all depends on what permissions that the repositories that you create will actually need to do. Most importantly **protect these credentials**.

Next, while referring to the output from your cloud shell, set the following three Codespaces Secrets as was done for AWS:

```
AZURE_TENANT_ID=e9439999999999999999999af64c0                 <--"tenant"
AZURE_CLIENT_SECRET=DHZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ4U        <--"password"
AZURE_CLIENT_ID=e5ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ10           <--"name"
```

Since we want to be able to run Azure CLI commands as soon as the container boots up, we need to add a configuration to the  `.devcontainer.json` file that logs us into Azure but uses the secrets that get set as environment variables. Here is the command that can do that:

```
az login --service-principal --username $AZURE_CLIENT_ID --password $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID
```

So we would need to make the `.devcontainer.json` file read as follows, by adding in the line with the `postStartCommand`:

```
{
  "name": "HitC Tools",
  "image": "resistor52/hitc-tools:ep16",
  "remoteEnv": { "AWS_DEFAULT_REGION":"us-east-1" },
  "postStartCommand": "az login --service-principal --username $AZURE_CLIENT_ID --password $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID"
}
```

Great. Let's test the latest incarnation. Delete any previous code spaces and spin up a new one for the 'space2' repository.

To test it, run the following command once the prompt appears:

```
az account show
```

The returned results should look like:

```
{
  "environmentName": "AzureCloud",
  "homeTenantId": "e9439999999999999999999af64c0",
  "id": "911d9999-9999999999-9999a4f55a",
  "isDefault": true,
  "managedByTenants": [],
  "name": "Pay-As-You-Go",
  "state": "Enabled",
  "tenantId": "e9439999999999999999999af64c0",
  "user": {
    "name": "e5ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ10",
    "type": "servicePrincipal"
  }
}
```

**NOTE:** You can always delete the service principle using `az delete sp delete`

## The GCP Secrets

In GCP create a new Service account called "codespaces" and grant it the editor role. Yes, this is a very powerful account, but we want it to be able to do everything but grant IAM access.

Next, create a new JSON-formatted key. The key will download.

Now we need to base64 encode the service account key. Assuming that the only JSON file in your downloads directory is the key, we can use the following command to convert it:

```
cat *.json | base64 -w0 > key_base64.txt
```

Note that the "w0" switch disables wrapping of the output which is necessary for our use case.

Open up the `key_base64.txt` file and create a Codespaces secret. Name the secret "GCP_KEY" and paste the entire contents of the `key_base64.txt` file in as the value.

Create another secret called "GCP_SERVICE_ACCOUNT" and paste in the name of the service account. The name should be in the format of `codespaces@{YOUR PROJECT ID}.iam.gserviceaccount.com`  

Make sure that the repositories selected for both are set to 'space2'

Similar to the trick that we did with Azure, we are going to run a command post-launch to log our Codespace into GCP. The command we will use is:

```
echo $GCP_KEY  | base64 -d > /tmp/key.json && gcloud auth activate-service-account $GCP_SERVICE_ACCOUNT --key-file=/tmp/key.json --project=$GCP_PROJECT_ID && rm /tmp/key.json
```

so that will make the new `.devcontainer.json` file look like the following:

```
{
  "name": "HitC Tools",
  "image": "resistor52/hitc-tools:ep16",
  "remoteEnv": { "AWS_DEFAULT_REGION":"us-east-1", "GCP_PROJECT_ID":"calm-energy-331601" },
  "postStartCommand": "az login --service-principal --username $AZURE_CLIENT_ID --password $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID && echo $GCP_KEY | base64 -d > /tmp/key.json && gcloud auth activate-service-account $GCP_SERVICE_ACCOUNT --key-file=/tmp/key.json --project=$GCP_PROJECT_ID && rm /tmp/key.json"
}
```
Note that the `remoteEnv` name/value pair in the JSON above also adds a new environment variable, the GCP_PROJECT_ID. Be sure to set this as appropriate for your GCP project.

Time to test our handiwork. Delete any previous code spaces and spin up a new one for the 'space2' repository. Run the following command once the prompt appears:

```
gcloud auth list
```

The output should look similar to:

```
Credentialed Accounts
ACTIVE  ACCOUNT
*       codespaces@calm-energy-331601.iam.gserviceaccount.com

To set the active account, run:
$ gcloud config set account `ACCOUNT`
```

And while we are at it, let's confirm that the AWS and Azure authentication still work as expected:

```
az account show
aws sts get-caller-identity

```

## Wrap up

Well, there you go. We covered a popular trend in cloud-based development called _Development Environments as Code_ and looked at Github's implementation. We looked at the default container and showed how a Github user can use dotfiles to customize it for their unique preferences. After that we used a `.devcontainer.json` file to customize the dev environment for the specific needs of our repository. In this case, our goal was to have a dev environment that allows us to run CLI commands or Terraform scripts against all three of the top cloud service providers. To do that, we had to leverage Codespaces Secrets to avoid having authentication credentials in our configurations.

If you have thoughts or comments on today’s episode, feel free to chime in on the comments for this YouTube video.

If you appreciate this video and want to see more like it, be sure to give it a "thumbs up."

Stay tuned for another installment of “Head in the Clouds” as announcements of new episodes are made on the SANS Cloud Security Twitter feed.

Meanwhile, be sure to check out the other great videos on the [SANS Cloud Security YouTube Channel](https://www.youtube.com/c/SANSCloudSecurity).

Take care.
