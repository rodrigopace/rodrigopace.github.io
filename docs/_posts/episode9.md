---
layout: page
name: Episode 9 - How to Inventory EC2 Instances with a Single Line of Code

---

# Episode 9 - How to Inventory EC2 Instances with a Single Line of Code

<div class="video-container"><iframe width="560" height="315" src="https://www.youtube.com/embed/FY4Vwvk42LQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

Welcome to Episode 9 of the “Head in the Clouds” Video Series. I am Ken Hartman, a SANS Certified Instructor and content creator for the SANS Cloud Security Curriculum.

Today’s episode is titled: “How to Inventory EC2 Instances with a single line of code”

The purpose of HITC is to teach foundational cloud skills and security knowledge that will help others thrive in the cloud. The content ideas come from my personal observation of skills that I see some students lacking when they show up to a SANS Cloud Security course. Other ideas are passed on from fellow SANS instructors.

Today’s topic will cover how to use a loop to iteratively call a cloud CLI while passing in a variable. This is a very powerful technique that can allow us to do many things much more rapidly via the terminal than could be done by the cloud provider's web console.

For this episode, we will be using the `aws ec2 describe-instances` command, but we could replace this with any of the other aws cli "describe-*" commands to generate inventories of all of our various cloud assets--and once you learn it you will almost certainly find occasions to use this technique with other cloud service providers.

## Set Up
Assuming that you do not have any EC2 instances running in your AWS account, launch 2 in your default region and one more in two other regions, for at least four instances running in at least three regions. For fun, change up the operating systems too.

For example, I launched two **Amazon Linux 2** instances in **us-east-1**, an **Ubuntu** instance in **us-east-2**, and a **Windows server** in **us-west-2**. Now we have something to work with.

To follow along with this episode, I am also assuming that you have [installed the AWS CLI (version 2)](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) and have also [configured it for your account](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html). If you have not yet done that, please pause the video and get that taken care of.   

Let me show you my configuration:

```
ken@msi:~$ aws configure
AWS Access Key ID [****************W4XJ]:
AWS Secret Access Key [****************GAzD]:
Default region name [us-east-1]:
Default output format [json]:
```

Notice that my default region is "us-east-1" and my default output is "json," but its ok if yours are different--just make sure you launched a couple VMs in your default region.

Once that is done, we can now run some CLI commands with some command-line kung fu.

## The describe-instances Command

First, lets start with the basic "describe-instances" command. To run that, just type:

```
aws ec2 describe-instances
```

This command outputs several lines of JSON output, because I have two instances running in my default region of **us-east-1**.

Ok, but that is almost too much information to be able to interpret on the screen. So, let's pair it down some. We can use a **query** parameter to select the data elements of interest.

## Using a Query Parameter

In this episode, I am not going to take the time to discuss navigating the JSON data structure, because I have already covered that in [Episode 1 - Using jq to Get the Results You Need From Any Command Line Interface](https://headintheclouds.site/episodes/episode1). But if we looks over the JSON output, maybe we are interested in the following items:

* InstanceId
* State (Running, Stopped, Terminated, etc.)
* AvailabilityZone
* PublicIpAddress
* LaunchTime

Well, in that case we would our query parameter would become:

```
'Reservations[].Instances[*].[InstanceId,State.Name,Placement.AvailabilityZone,PublicIpAddress,LaunchTime]'
```

Which would make our full command to be:

```
aws ec2 describe-instances --query 'Reservations[].Instances[*].[InstanceId,State.Name,Placement.AvailabilityZone,PublicIpAddress,LaunchTime]'
```

And this produces output that will looks something like

```
[
    [
        [
            "i-0206766afa3124133",
            "running",
            "us-east-1c",
            "18.210.12.130",
            "2021-07-22T23:38:59+00:00"
        ],
        [
            "i-057026ecacfbfa82d",
            "running",
            "us-east-1c",
            "3.239.21.164",
            "2021-07-22T23:38:59+00:00"
        ]
    ]
]
```

Of course, your metadata will be different, but the format should look similar.

## Format the Output as a Table

Now let's add in another parameter so that our output is displayed as a table:

```
--output table
```

This makes our revised command become:

```
aws ec2 describe-instances --output table --query 'Reservations[].Instances[*].[InstanceId,State.Name,Placement.AvailabilityZone,PublicIpAddress,LaunchTime]'
```

This command produces the following output, displayed as a table:

```
------------------------------------------------------------------------------------------------
|                                       DescribeInstances                                      |
+---------------------+----------+-------------+----------------+------------------------------+
|  i-0206766afa3124133|  running |  us-east-1c |  18.210.12.130 |  2021-07-22T23:38:59+00:00   |
|  i-057026ecacfbfa82d|  running |  us-east-1c |  3.239.21.164  |  2021-07-22T23:38:59+00:00   |
+---------------------+----------+-------------+----------------+------------------------------+
```

much more readable!

## Describing Instances in Other Regions

if we want to describe the instances in another region, simply use the "region" parameter with the command:

```
--region "us-east-2"
```

This makes our revised command become:

```
aws ec2 describe-instances --region "us-east-2" --output table --query 'Reservations[].Instances[*].[InstanceId,State.Name,Placement.AvailabilityZone,PublicIpAddress,LaunchTime]'
```

And now we see the data from this other region:

```
------------------------------------------------------------------------------------------------
|                                       DescribeInstances                                      |
+---------------------+----------+-------------+----------------+------------------------------+
|  i-0e8bf8668aabfefcd|  running |  us-east-2a |  13.58.143.127 |  2021-07-22T23:39:52+00:00   |
+---------------------+----------+-------------+----------------+------------------------------+
```

Great, but what if we want to describe the instances across all of the regions for our AWS account? To
do that, we need to use a "for loop" and a "subshell"

## Looping
A "for loop" assigns each element of a set to a variable one at a time and then performs various commands using that value and, when done, iterates on to the next variable, repeating the various commands inside the loop.

For example, we could have a set of animals that we loop through with a variable called "pet" and use that variable in a command:

```
for pet in cat dog bunny hampster goldfish; do echo "I have a $pet"; done
```

would result in:

```
I have a cat
I have a dog
I have a bunny
I have a hampster
I have a goldfish
```

## Subshells

Sometimes we want to use the output of one command as part of another command.

As a simple example, we could run the `date` command in a subshell and use that in an echo command:

```
echo "today is $(date)"
```

results in:

```
today is Thu Jul 22 21:18:37 EDT 2021
```

We inform BASH that we want to run a subshell by enclosing the command with `$(` and `)`

## Get a List of All AWS Regions

AWS adds new regions from time to time. So to be certain that we know all of the regions, the best practice is to query AWS for this information at run-time.

To do this, AWS provides the **describe-regions** command. Try it out:

```
aws ec2 describe-regions
```

Note that the output has extraneous information, so we need to use a query parameter.

```
aws ec2 describe-regions --query 'Regions[].RegionName'
```

That yields:

```
[
    "eu-north-1",
    "ap-south-1",
    "eu-west-3",
    "eu-west-2",
    "eu-west-1",
    "ap-northeast-3",
    "ap-northeast-2",
    "ap-northeast-1",
    "sa-east-1",
    "ca-central-1",
    "ap-southeast-1",
    "ap-southeast-2",
    "eu-central-1",
    "us-east-1",
    "us-east-2",
    "us-west-1",
    "us-west-2"
]
```

Good, but we want a simple list, not a JSON array, so lets use `--output text` as follows:

```
aws ec2 describe-regions --query 'Regions[].RegionName' --output text
```

And this gives us:

```
eu-north-1      ap-south-1      eu-west-3       eu-west-2       eu-west-1       ap-northeast-3  ap-northeast-2  ap-northeast-1  sa-east-1       ca-central-1    ap-southeast-1  ap-southeast-2  eu-central-1
    us-east-1       us-east-2       us-west-1       us-west-2
```

## Loop Through the Results of a Subshell

Now that we have the regions output via a CLI command, we can put that CLI command in a subshell and use that for our loop:

```
for region in $(aws ec2 describe-regions --query 'Regions[].RegionName' --output text); do echo $region; done
```

now, we get:

```
eu-north-1
ap-south-1
eu-west-3
eu-west-2
eu-west-1
ap-northeast-3
ap-northeast-2
ap-northeast-1
sa-east-1
ca-central-1
ap-southeast-1
ap-southeast-2
eu-central-1
us-east-1
us-east-2
us-west-1
us-west-2
```

## Putting It All Together

Now that we have a functioning for loop that iterates through all of the regions, lets replace the `echo $region` with our describe-instances command that we worked out earlier in the episode:

```
for region in $(aws ec2 describe-regions --query 'Regions[].RegionName' --output text); do aws ec2 describe-instances --region $region --output table --query 'Reservations[].Instances[*].[InstanceId,State.Name,Placement.AvailabilityZone,PublicIpAddress,LaunchTime]'; done
```

Note that the `--region` parameter now has the variable `$region` so that the loop can iterate through each region and repeatedly call the `describe-instances` command for each new value.

Here are my results:

```
------------------------------------------------------------------------------------------------
|                                       DescribeInstances                                      |
+---------------------+----------+-------------+----------------+------------------------------+
|  i-0206766afa3124133|  running |  us-east-1c |  18.210.12.130 |  2021-07-22T23:38:59+00:00   |
|  i-057026ecacfbfa82d|  running |  us-east-1c |  3.239.21.164  |  2021-07-22T23:38:59+00:00   |
+---------------------+----------+-------------+----------------+------------------------------+
------------------------------------------------------------------------------------------------
|                                       DescribeInstances                                      |
+---------------------+----------+-------------+----------------+------------------------------+
|  i-0e8bf8668aabfefcd|  running |  us-east-2a |  13.58.143.127 |  2021-07-22T23:39:52+00:00   |
+---------------------+----------+-------------+----------------+------------------------------+
------------------------------------------------------------------------------------------------
|                                       DescribeInstances                                      |
+---------------------+----------+-------------+----------------+------------------------------+
|  i-04ef44375c4f7a83a|  running |  us-west-2a |  54.218.59.234 |  2021-07-22T23:40:42+00:00   |
+---------------------+----------+-------------+----------------+------------------------------+
```


## Wrap Up

This technique is a powerful tactic to quickly generate an inventory of your EC2 instances. You might find assets in regions that you were unaware had anything. But don't stop with EC2 instances, what about:

* VPCs
* Security Groups
* Peering Connections
* Load Balancers
* and many more assets

Remeber that **_an asset that is not managed, is unlikely to be secure_** therefore **INVENTORY EVERYTHING**

Don't forget to turn off all of your instances! You can rerun the command to verify that they have all been terminated. And for extra credit, you could write a loop to iterate through all of your running instances and execute the **terminate-instances** command.

If you have thoughts or comments on today’s episode, feel free to chime in on the comments for this YouTube video.

If you appreciate this video and want to see more like it, be sure to give it a "thumbs up."

Stay tuned for another installment of “Head in the Clouds” as announcements of new episodes are made on the SANS Cloud Security Twitter feed.

Meanwhile, be sure to check out the other great videos on the [SANS Cloud Security YouTube Channel](https://www.youtube.com/c/SANSCloudSecurity).

Take care.
