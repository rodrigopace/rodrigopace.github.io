---
layout: page
name: Episode 7 - Passively Evaluate a SaaS Provider’s AppSec using ZAP
---

# Episode 7 - Passively Evaluate a SaaS Provider’s AppSec using ZAP

<div class="video-container"><iframe width="560" height="315" src="https://www.youtube.com/embed/kGGYuH-68A4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

Welcome to Episode 7 of the “Head in the Clouds” Video Series. I am Ken Hartman, a SANS Certified Instructor and content creator for the SANS Cloud Security Curriculum.

Today’s episode is titled: “Passively Evaluate a SaaS Provider’s AppSec using ZAP”

The purpose of HITC is to teach foundational cloud skills and security knowledge that will help others thrive in the cloud. The content ideas come from my personal observation of skills that I see some students lacking when they show up to a SANS Cloud Security course. Other ideas are passed on from fellow SANS instructors.

Today’s topic draws from some private consulting work I was doing last week for a client concerned about the threat of ransomware. This particular client is in a niche and uses some specialty Software-as-a-Service (SaaS) solutions that are unique to their niche. Some people may have the mistaken belief that to mitigate ransomware all one needs to do to is ensure that there are good backups generated on a regular basis. However, a good backup does not prevent the attacker from dumping all of your sensitive data on the public internet, regardless of whether you pay the ransom!

Therefore, the best mitigation against ransomware is strong access control. This includes multi-factor authentication and protecting against credential compromise. We cover this in depth in the two SANS Cloud Security Courses that I teach: [SEC488: Cloud Security Essentials]( https://www.sans.org/cyber-security-courses/cloud-security-essentials/) and [SANS SEC510: Public Cloud Security - AWS, Azure, and GCP]( https://www.sans.org/cyber-security-courses/public-cloud-security-aws-azure-gcp/). As part of my engagement, I made several recommendations—including the benefits of using single-sign-on.

Now, by no means did I intend to imply that backups are not important. They are an essential part of any business continuity strategy, and not just continuing business if ransomware attacks. The SaaS provider that my client uses does not have a documented API. Therefore, to generate a database backup, an admin user must generate one via the web console. My client wished to have that process automated so that it could happen on a frequent periodic basis without relying on human interaction.

Python has a module called “requests” that is ideal for this type of task, but to write the code, I needed to spy on the HTTPS traffic. The tool of choice for this is an “intercepting proxy” and a great, open source option is **[OWASP Zed Attack Proxy (ZAP)](https://owasp.org/www-project-zap/)**. As I worked on this project, it occurred that this would make a great _Head in the clouds_ episode topic.

**CAUTION:** It is not appropriate to actively scan computer systems that you do not own without permission and may be outright illegal. Consult a legal expert before doing any sort of security testing. I am not a legal expert and laws vary by jurisdiction. Neither I, nor SANS can be held liable for any security testing that you perform, so make sure that you follow the laws where you live and work.

![Image Security Testing Warning](/images/ep7/warning.png)

In this episode, I will be demonstrating the technique of passive scanning while using a SaaS application as a normal user with a web browser. ZAP will not be used to scan or attack the SaaS provider, although those are amazing capabilities of this security tool. ZAP will simply be inspecting the traffic that flows through it as the proxy and identifying any security issues during normal usage. Not only is the fact that you are using ZAP virtually undetectable by the web application (when used in this manner), it should be totally appropriate, assuming you are using your web browser according to the SaaS provider’s terms of service and acceptable use policy. (Just remember, I am not a lawyer, so talk to yours!)

Great, the agenda for the remainder of this episode is to first get ZAP installed and then we will have it proxy our interactions with a cloud service provider. Lastly, we will take a look at the findings and generate a report.

## Setting Up a Cloud Workstation

Since this is ***Head in the Clouds***, lets use a cloud-based Ubuntu workstation. Fortunately, I have a Terraform script that can deploy one for you quite easily. I like to use cloud-based workstations for projects and experiments because I know that I am always starting from a known configuration.

**NOTE:** Rather than running terraform to deploy a cloud-based workstation, you may use an Ubuntu Desktop VM running locally, in VMWare for example. Just skip this section.

To deploy the cloud workstation, open the terminal on a system that has terraform and git installed and clone down the project as shown:

```
which terraform
which git
git clone https://github.com/Resistor52/tf-cloud-workstation.git
cp terraform.tfvars.example terraform.tfvars
```

Next, modify the ` terraform.tfvars` file as appropriate for your AWS environment. If you need more information on how to do this or are curious about how Terraform or Git works, I would encourage you to watch my Tech Tuesday video where I cover this [cloud-based workstation in depth]( https://www.youtube.com/watch?v=5L6yxXXn0-I).

Next, run the following two commands and type ‘yes’ when prompted:

```
terraform init
terraform apply
```

When the terraform script is done, it will display output that looks like:

![Terraform output](/images/ep7/terraform_output.png)

Just remember, that the initialization script that is updating the operating system and installing Ubuntu desktop still takes a little time to finish, so you will not be able to RDP quite yet. So, SSH instead using the string that is displayed in the Terraform outputs.

I like to monitor the initialization script so that I can tell when the setup has completed. To do that I run

```
tail -f /tmp/first-boot.log
```

## Review SaaS Provider’s ToS and AUP

While our cloud workstation initializes and just out of the abundance of caution, let’s pause for a moment and review our SaaS Provider’s Terms of Service (ToS) and Acceptable Use Policy (AUP) to make sure that we are operating above-board. For our example, we will be using Dropbox.com. Dropbox.com is a mature provider and since we just want to demonstrate a technique and not embarrass any company, I chose Dropbox—for reasons that will become apparent in a minute.

*	Dropbox Terms of Service - [https://www.dropbox.com/terms]( https://www.dropbox.com/terms)
*	Dropbox Acceptable Use Policy – [https://www.dropbox.com/acceptable_use]( https://www.dropbox.com/acceptable_use)

A quick review of the AUP indicates that Dropbox.com has a public [Bug Bounty Program]( https://hackerone.com/dropbox). Generally speaking, a company that has a bug bounty program that has been active for a while will have an application security program that is mature enough to prevent the company from going broke by paying out bounties. The presence of this program (which allows active scanning) also means that the passive technique that I will be demonstrating today is permitted.

## Installing the Zed Attack proxy

Once we have an Ubuntu workstation running, we need to install Java...

```
# Install OpenJDK Java
sudo apt install -y default-jre
```

and ZAP:

```
# https://www.zaproxy.org/download/
cd /tmp
wget https://github.com/zaproxy/zaproxy/releases/download/v2.10.0/ZAP_2_10_0_unix.sh
sudo bash ZAP_2_10_0_unix.sh
```

As ZAP installs, it will prompt you to accept the license agreement and to select the type of installation. Choose [1] for the standard installation.

Once that is complete, we can exit the SSH session and RDP in. As a reminder, to see the RDP user name and password, just type:

```
terraform output
```

Great. Now RDP into the IP address shown using the displayed temporary credentials.

## Using ZAP

Since this is a brand-new installation, we need to first launch Firefox and accept all the initialization agreements so that Firefox will play nice with ZAP. Make Firefox your default browser and click “not now” for the rest of the prompts. Having done that, close Firefox.

To launch ZAP, click on the menu in the top left corner and search for “zap.” Then click on “OWASP ZAP” to launch it.

Once ZAP launches, you will be prompted with an option to persist the ZAP session. Choose “no” for the time being.

Since we want to do a passive scan while interacting with our SaaS provider as a normal user, we will choose the “Manual Explore” option. Do not choose the automated scan, as that will start the spider which will crawl the web application and will start to inject potentially abusive data into the application. An automated scan may find more issues, but it can also break things. **Only use the automated scan with permission or systems that you own** and preferably in a non-production environment.

Enter in the URL of the SaaS provider (https://dropbox.com) and then click “Launch Browser.” This time when Firefox launches it will be automatically configured to use ZAP as its proxy and will trust the ZAP SSL certificate that ZAP uses to MitM the HTTPS session.

In a moment, the splash page for the ZAP Heads Up Display will appear. You may want to take the HUD Tutorial, but I will click “Continue to your target.”

We can see a few colored flags on the sides. These are alerts. Without closing Firefox, switch over to ZAP, and we can see all of the sites that Firefox has contacted. Most of these are referenced from the Dropbox.com HTTP content, but a few may be from Firefox itself as it phones home to mozzila.org.

Let’s switch back to Firefox and use Dropbox some. Click a few menu items and then sign in. (Register an account if you do not have one.)

Upload and download an item while you are at it. After you are done playing around and exploring all of the Dropbox functionality, close Firefox.

Next take a look at the all of the Sites that ZAP captured. Delete the ones that do not end in “dropbox.com” or “dropboxstatic.com”

Click on the “Alerts” tab and notice that there are some alerts including a “Vulnerable JS Library” and multiple different cross-domain warnings.

Going back up and expanding the “Sites” object tree, we can see one called “POST:ajax_login().” Take a look at the request and the response messages. Very cool. It looks like the ajax wrapped my password with a public key before posting it with the other data on the login form!

Before we wrap up, Let’s generate a report on the findings, via the “Report” menu. Note that several file format options are available. For now, select “Generate HTML Report…” and take a look at it.

## Wrap Up

That's it! Now you a good technique for passively inspecting a SaaS provider’s application security. However, remember that these are just findings generated by a tool! They are not necessarily exploitable vulnerabilities. In fact, let’s take another look at the Dropbox Bug Bounty program guidelines. They state a couple of bullet points defining out of scope items that are applicable to our findings:

*	Missing security headers which do not lead directly to a vulnerability.
*	Missing best practices (we require evidence of a security vulnerability).
*	Use of a known-vulnerable library (without evidence of exploitability).

It is reasonable to assume that these items have been added to the program guidelines based on multiple conversations with security researchers who have used intercepting proxies like ZAP or Burp. It is also highly likely that Dropbox has mitigating controls and countermeasures to reduce the risk of the things we discovered today. But perhaps not, and that is why they have a Bug Bounty Program after all. However, we are going to stop our investigation here.

Remember, I chose Dropbox for this demo because they have a mature security program and a bug bounty. However, your niche SaaS provider may not have either! Remember that just because there are findings from a tool does not mean that the vendor is screwing up, they are simply discussion points that allow you to delve into your vendor’s AppSec. Always conduct these discussions from a posture of mutual respect and curiosity, rather than patronization.

SANS Has a whole course on [Wep Application penetration Testing (SEC542)](https://www.sans.org/cyber-security-courses/web-app-penetration-testing-ethical-hacking/) and that course goes much deeper into how to use both ZAP and Burp, the two most popular intercepting proxies. Remember that SaaS is just a web application hosted by a cloud service provider.

SANS also has a new course and certification on [Cloud Penetration Testing, SEC588](https://www.sans.org/cyber-security-courses/cloud-penetration-testing/). Check that one out too.

If you have thoughts or comments on today’s episode, feel free to chime in on the comments for this YouTube video.

If you appreciate this video and want to see more like it, be sure to give it a "thumbs up."

Stay tuned for another installment of “Head in the Clouds” as announcements of new episodes are made on the SANS Cloud Security Twitter feed.

Meanwhile, be sure to check out the other great videos on the [SANS Cloud Security YouTube Channel](https://www.youtube.com/c/SANSCloudSecurity).

Take care.
