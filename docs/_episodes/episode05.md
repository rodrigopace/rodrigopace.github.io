---
layout: page
title: Episode 5 - Tips for success with Command Line Interfaces using BASH
---

 5 - Tips for success with Command Line Interfaces using BASH

<div class="video-container"><iframe width="560" height="315" src="https://www.youtube.com/embed/BwoU9BJeGKU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

Welcome to Episode 5 of the “Head in the Clouds” Video Series. I am Ken Hartman, a SANS Certified Instructor and content creator for the SANS Cloud Security Curriculum.

Today’s episode is titled: “Tips for success with Command Line Interfaces using BASH.”

The purpose of HITC is to teach foundational cloud skills and security knowledge that will help others thrive in the cloud. The content ideas come from my personal observation of skills that I see some students lacking when they show up to a SANS Cloud Security course. Other ideas are passed on from fellow SANS instructors.

Today’s topic is based on a conversation with SANS Principal Instructor Eric Johnson during a break while co-teaching SEC510 at SANS Security West. The full title of SEC510 is “[Public Cloud Security: AWS, Azure, and GCP](https://www.sans.org/cyber-security-courses/public-cloud-security-aws-azure-gcp/)” and throughout the entire class, various Command Line Interface commands are run from the Linux BASH Terminal. This is the case with most of the other classes in the SANS Cloud Security Curriculum as well.

First off, what is the big deal with command line interfaces? Command line interfaces allow one to manipulate a cloud service from a terminal. This is a powerful technique because it is succinct and efficient. It is also easier to copy and paste a single command from a playbook than it is to say, “go to your web console, click on the “X” menu and select “Y” item, then click on the “Z” object, etc.”

Multiple command line interface commands can be combined to make a script. And this is where knowledge of BASH will be payoff as well. And since this is a brief tutorial on BASH, lets jump into it.
The full name of BASH is the _Bourne Again Shell_. It is the most popular (but not the only shell) that can be used by the Linux Operating System. It has been around since 1989 and is still actively maintained [1].

## BASH History

One of the first tricks that a user will figure out, is that they can use the up arrow to revisit the last command that they ran. This is a handy trick, and I use it all of the time. However, I also see new students use the up-arrow dozens of times to access a command that they ran much earlier in their session. This is rather painful, so our first few tricks involve how to access one’s command history.

To view the commands that have been run recently, just type `history` and you will see a numbered list with the highest number being the last command.

![Image of BASH History](/images/ep5/history.png)

To execute any of the listed commands simply type an exclamation point and the number that corresponds to the command. So, for example, if I want to rerun the `whoami` command, I can just type “bang” 5.

![Execute a command from BASH History](/images/ep5/execute_from_history.png)

This is a great time saver, and very good to know. I can also pipe one command into another. In fact, we can pipe the output of the history command into grep and use that to filter our results. For example, if we used the `cat` command to look at the contents of a text file recently but forgot the path, we could use `history | grep cat` to find it.

![Grepping BASH History for a command containing cat](/images/ep5/grep_for_cat.png)

Grep is powerful, and we will explore it more in the next edition of HITC along with some other powerful commands for slicing and dicing log files.

Perhaps when you ran the history command just now, you wondered where it pulled its data from? If so, great. Its good to be curious. If you look in your home directory, you will see a file called `.bash_history`

To view the file listing, type `ls -la ~/.bash_history`

Remember that `ls` is the command to list the directory contents and that you can view the manual page of any command by typing `man` in front of the command. For example, `man ls`

![Viewing the man page for ls](/images/ep5/man_ls.png)

In fact, the best advice that is often given to new users of Linux is to RTFM – Read the Fine Manual. By looking at the man page for `ls` we can see that the `l` option is to direct the `ls` command to display the file listing in the long format. We can also see that the `a` option directs the `ls` command to output all files that match the FILE specification, including hidden files.

![Listing the BASH history file using ls](/images/ep5/ls_output.png)

Note: Looking at the output of this `ls` command, we can see that the permissions on the .bash_history file are read and write by the owner, so this file should not be regarded as a robust audit log. If you are not familiar with Linux permissions, I will refer you to the [link displayed on screen](https://www.linux.com/training-tutorials/understanding-linux-file-permissions/).

![Screenshot of Understanding Linux File Permissions article](/images/ep5/understanding_file_perms.png)

Incidentally, `.bash_history` is a hidden file and we can tell this because any filename that starts with a period is a hidden file. By default, the .bash_history for any user is in their home directory. So when we ran `ls -la ~/.bash_history` we told the `ls` command to look for the .bash_history file in our home directory.
This is because the tilde character is an alias for the $HOME environment variable. We will talk about variables next, but before we move on, let’s take a look at the contents of our .bash_history file. Let’s run `cat ~/.bash_history`

![Using cat to view the BASH history file](/images/ep5/cat_bash_history.png)

## BASH Variables

BASH has two types of variables—local variables and environment variables. Local variables can be set inside a script or an interactive shell session, but when the script has completed or the shell session as terminated, the variables are released from memory. On the other hand, environment variables are maintained in memory until the system is rebooted, so that other scripts, programs, or shell sessions can access them. Also note that many of the environmental variables are set by start-up scripts that run at boot time.

To set a variable we use an equal sign. For example, to set the value of ‘488’ to a local variable called ‘SANS’ we would use `SANS=488`. We can view the contents of the variable using the `echo` command. Recall that to learn more about the echo command, you can use `man echo` to view its manual page. All of these commands have a multitude of options that make them quite versatile and worth learning.

![Accessing a variable](/images/ep5/echo_sans.png)

Note that when we use the variable, we prepend a dollar sign to access the contents of the variable. Without the dollar sign, the word ‘SANS’ is treated as a string.

If I open up a new terminal session, the value of the SANS variable has not been set.

Earlier in this edition, I mentioned that the tilde character was a shorthand representation for the $HOME environmental variable. Let’s prove that:

![Comparing the ~ and HOME Variables](/images/ep5/echo_home.png)

Since we are talking about environmental variables, you may be wondering what other environmental variables have been set on your system. Well, fortunately, Linux has a command for that as well. Use the man command to learn more about the `env` command.

![Viewing the man page for env](/images/ep5/man_env.png)

Great. Now let’s check out our environment variables:

![Viewing the env output](/images/ep5/env.png)

Notice the HOME, PWD, USER, and PATH variables. These are used by the operating system to keep track of specifics about the user’s environment. Many command line interfaces will use environment variables to set credentials that the CLI will use to authenticate to the cloud service.

![Grepping the env output](/images/ep5/grep_env.png)

To set an environment variable, add the `export` command in front of the assignment statement. For example, with AWS, I can set two environment variables with valid a access key id and access key secret as shown:

```
export AWS_ACCESS_KEY_ID=AKIA2PRSBJQH7AVO5KX7
export AWS_SECRET_ACCESS_KEY=rkjDr/Wx7BrX+omy+TWQMbvcHCUyYI3bR6dGU7u2
```

And that will allow me to run various AWS CLI commands as allowed by the permissions associated with the identity tied to the AWS Access Key ID.

![Running the aws s3 ls command](/images/ep5/aws_s3_ls.png)

Checkout [https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html) for a list of environment variables supported by the AWS CLI [3].

Sometimes it may be necessary to change or remove an environment variable. To change a variable, just set the same variable to a different value. Use the `unset` command to remove it.

![Unset an environment variable](/images/ep5/unset.png)

When helping SANS students, I have often seen them hitting the right arrow many times to get to the end of the command line or hitting the left arrow key multiple times to get to the beginning. There is an easier way!

Use **Control-A** to jump to the beginning of the line and hit **Control-E** to jump to the end of the line. I use the following memory trick: the letter “A” is at the beginning of the alphabet and “E” is for “End.”

Many folks know that you can use **Control-C** to “cancel” a program that is running from the command line. But you can also do this if you mess up typing a command. Just hit **Control-C** to cancel it rather than backspacing to the beginning of the line. When you do so, you will be returned to an empty prompt.

There are many more BASH terminal shortcuts, so as we close, I will leave you with good reference that lists several:  [https://ss64.com/bash/syntax-keyboard.html](https://ss64.com/bash/syntax-keyboard.html).

Of course, to remember them, you have to use them! So, practice one at a time until it becomes second nature. Over time, you will thank yourself for becoming more efficient.

## Wrap Up

That's it! Now you know some tips and tricks that will help you navigate the BASH Shell and how to work with environment variables.

If you have thoughts or comments on today’s episode, feel free to chime in on the comments for this YouTube video.

If you appreciate this video and want to see more like it, be sure to give it a "thumbs up."

I also want to mention that we have the SANS CloudSecNext Summit coming up quickly. It runs next week. That is June 3-4.

Visit [sans.org/CloudSecNextSummit](https://www.sans.org/event/cloudsecnext-summit-2021/) for details and to register.
 It’s not too late and, best of all the Summit is free. We are anticipating upwards of 10,000 attendees. I will be a chair of the Solutions Track. We always have lots of fun at these summits and they are packed with great information.

![Image announcing the SANS CloudSecNext Summit](/images/ep4/SUMMIT_CloudSecNext Slide.png)

Stay tuned for another installment of “Head in the Clouds” as announcements of new episodes are made on the SANS Cloud Security Twitter feed.

Meanwhile, be sure to check out the other great videos on the SANS Cloud Security YouTube Channel.

Take care.

____________

[1] https://opensource.com/19/9/command-line-heroes-bash

[2] https://www.linux.com/training-tutorials/understanding-linux-file-permissions/

[3] https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html

[4] https://ss64.com/bash/syntax-keyboard.html
