---
layout: page
name: Episode 12 - Are Snapshots of Cloud Virtual Hard Drives Forensically Valid?

---

# Episode 12 - Are Snapshots of Cloud Virtual Hard Drives Forensically Valid?

<div class="video-container"><iframe width="560" height="315" src="https://www.youtube.com/embed/DBoB13srqmQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

Welcome to Episode 12 of the “Head in the Clouds” Video Series. I am Ken Hartman, a SANS Certified Instructor and content creator for the SANS Cloud Security Curriculum.

Today’s episode is titled: “Are Snapshots of Cloud Virtual Hard Drives Forensically Valid?”

A few years ago, I heard a talk by an engineer at AWS who said that one should not rely on a snapshot of an EBS Volume for forensic evidence in a court case, because you would not be able to get an AWS Engineer to testify in court regarding the proprietary software that is used to create the snapshot.

That's the wrong way to look at the issue. The proper approach is to use the same technique that would be used with a HDD from a Laptop--compare the cryptographic hash of the copied volume with hash of the original volume.

This episode demonstrates how to clone a virtual hard drive in AWS, Azure, and GCP. Next, we attach the cloned volume to a second virtual machine designated as a forensic workstation in each of the clouds. To demonstrate that it is a true copy, we will compare the SHA1 hash of the source volume with the SHA1 hash of a volume created from the snapshot of the source volume.

## AWS

Let's take a look at each one of the big three cloud service providers, starting with AWS. For this episode, we will be using the consoles.

### AWS Setup

Launch two Amazon Linux 2 EC2 virtual machines in the same availability zone of a region of your choosing. A t2.micro instance type is fine with a single hard drive. All of the default settings are fine. Once the instances are launched, tag one with the name "Test System" and the other "Forensic Workstation."

SSH into both Virtual Machines. So as to avoid confusion regarding the role of each virtual machine, change the hostname using `sudo hostname <NAME>` where <NAME> is either "test-system" or "forensic-workstation" depending on the VM. Run `exec bash` to update the prompt with the new hostname.

### AWS - Attach a Data Volume to Test System

In the SSH session with the "Test System" instance, run the `lsblk` command and observe that there is a single EBS volume ("xvda") with one partition ("xvda1") and that the partition is mounted to root ("/")

```
[ec2-user@test-system ~]$ lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk
└─xvda1 202:1    0   8G  0 part /
[ec2-user@test-system ~]$
```

In the EC2 Web Console, click "Volumes" in the left hand navigation menu. Note the Availability Zone (AZ) of the two existing drives and then click the blue "Create Volume" button. Select the same AZ for the new volume as the existing two volumes that was just noted. Change the size of the volume to be 5 Gb. Leave the snapshot field blank and do not select "Encrypt this volume." Create a "Name" tag for the volume called "Data." Having done that, click the blue "Create Volume."

Next, let's attach this new volume to the "Test System" instance. Select the "Data" volume and then choose the "Attach Volume" option from the "Actions" dropdown field. A form will pop up and select the "Test System" as the instance to attach the volume to. Leave the other defaults and click the blue "Attach" button.

Back in the SSH session with the "Test System" instance, run the `lsblk` command and observe the changes:

```
[ec2-user@test-system ~]$ lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk
└─xvda1 202:1    0   8G  0 part /
xvdf    202:80   0   5G  0 disk
[ec2-user@test-system ~]$
```

Note that the volume has not been mounted and is not even formatted yet, for that matter. Let's do that now:

```
sudo mkfs.ext4 /dev/xvdf
sudo mkdir /data
sudo mount /dev/xvdf /data
```

And now if we rerun `lsblk` we can see that the new volume has been mounted.

```
[ec2-user@test-system ~]$ lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk
└─xvda1 202:1    0   8G  0 part /
xvdf    202:80   0   5G  0 disk /data
```

Now let's add some sample data to the volume:

```
sudo chown ec2-user:ec2-user /data
echo "Here is a bunch of evil" > /data/evil2.txt
sudo dd if=/dev/urandom bs=512 count=2 > /data/evil3.txt
ls -l /data
```

Next we will calculate a SHA1 hash of our new "Data" volume, but before we do, let's unmount it to prevent any accidental changes to the volume subsequent to the hash calculation.

```
sudo umount /data
sudo sha1sum /dev/xvdf
```

The results will look similar to the following, but note that your hash will be different, because evil3.txt is filled with random data.

```
[ec2-user@test-system ~]$ sudo umount /data
[ec2-user@test-system ~]$ sudo sha1sum /dev/xvdf
11ce8efed7c4b538a0dd691502a12f8c02ee6011  /dev/xvdf
[ec2-user@test-system ~]$
```

### AWS - Clone the Data Volume

Next we will clone the data volume by making a snapshot of the volume and then make a new volume from the copy. In the EC2 Web Console, select the "Data" volume and then choose the "Create Snapshot" option from the "Actions" dropdown field. A new form will appear. Enter "Snap of Data Volume" for the description. Create a "Name" tag of "Evidence."

To see the new snapshot, click "Snapshots" in the left hand navigation menu. Once the status of the snapshot shows that it is completed, select "Create Volume" from the "Actions" dropdown field. A new form will load and be sure not to change the size of the new volume to be created. If you do, the SHA1 hash will not match our reference "Data" volume from the "Test System." As before, be sure to select the AZ that the other assets are in. Tag the new volume with a "Name" tag of "Evidence" and click "Create Volume."

### AWS - Attach Evidence Volume to the Forensic Workstation    

The process of attaching the "Evidence Volume" to the "Forensic Workstation" is very similar to the process we followed to attach the "Data" volume to the "Test System." Select the "Evidence" volume and then choose the "Attach Volume" option from the "Actions" dropdown field. A form will pop up and select the "Forensic Workstation" as the instance to attach the volume to. Leave the other defaults and click the blue "Attach" button.

Now, if we run `lsblk` in the SSH session with our forensic workstation, we see that our volume has been attached:

```
[ec2-user@forensic-workstation ~]$ lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk
└─xvda1 202:1    0   8G  0 part /
xvdf    202:80   0   5G  0 disk
```


Now, for the moment of truth--time to calculate the hash of the evidence volume:

```
sudo sha1sum /dev/xvdf
```

My results were:

```
[ec2-user@forensic-workstation ~]$ sudo sha1sum /dev/xvdf
11ce8efed7c4b538a0dd691502a12f8c02ee6011  /dev/xvdf
```

And as you can see the SHA1 hash of the does indeed match the hash of the data volume as calculated on the Test System. Thus we can conclude that the process of making a snapshot of an EBS Volume and making a same-sized volume of the snapshot is a true forensic copy.

Note that we could have made a snapshot of the boot volume and could have made a clone of it. The problem with that is that volume would be constantly changing unless it was unmounted and if it was unmounted, we could use the operating system to calculate the SHA1 hash.

In practice, we simply make a snapshot of the boot volume (or any other volume) as needed and trust the integrity of the snapshot. In AWS, each snapshot is given a Unique ID and that ID is never reused. Recording that Snapshot ID should be treated the same way that a Hash is treated during an incident investigation. Of course, one should also document the Hash of the cloned volume when it is attached to the forensic workstation before other analysis is performed.  

Once the hash is calculated on the forensic workstation, the volume can be mounted read-only and analyzed using forensic tools.

```
sudo mkdir /evidence
sudo mount -o ro /dev/xvdf /evidence
sudo ls -l /evidence
```

Note the read-ony ("ro") option passed into the mount command. These commands produce:

```
[ec2-user@forensic-workstation ~]$ sudo mkdir /evidence
[ec2-user@forensic-workstation ~]$ sudo mount -o ro /dev/xvdf /evidence
[ec2-user@forensic-workstation ~]$ sudo ls -l /evidence
total 24
-rw-rw-r-- 1 ec2-user ec2-user    26 Sep 16 14:33 evil2.txt
-rw-rw-r-- 1 ec2-user ec2-user  1024 Sep 16 14:33 evil3.txt
drwx------ 2 root     root     16384 Sep 16 14:28 lost+found
```

## Azure

Next up is Azure. The process for Azure will be similar to what we just did in AWS, with the exception of UI differences and some terminology.

### Azure Setup

Launch two Azure virtual machines one at a time in a region of your choosing. Put them into a new resource group named "hitc." Use the Ubuntu 20.04 Gen 2 Image and name one "test-system" and the other "forensic-workstation." Select a "Standard_B1s" size VM and either generate a new SSH key or use one already stored in Azure. All of the default settings on the Disks, Networking, Management, and Advanced tabs are ok as is.

SSH into both Virtual Machines. In Azure, the hostname is already set for us.

### Azure - Attach a Data Volume to the Test System

In the SSH session with the "Test System" instance, run the `lsblk` command and observe that there are two volumes, "sda" and "sdb" and that both volumes have partitions.

```
azureuser@test-system:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0     7:0    0 70.3M  1 loop /snap/lxd/21029
loop1     7:1    0 55.4M  1 loop /snap/core18/2128
loop2     7:2    0 32.3M  1 loop /snap/snapd/12883
sda       8:0    0   30G  0 disk
├─sda1    8:1    0 29.9G  0 part /
├─sda14   8:14   0    4M  0 part
└─sda15   8:15   0  106M  0 part /boot/efi
sdb       8:16   0    4G  0 disk
└─sdb1    8:17   0    4G  0 part /mnt
sr0      11:0    1  628K  0 rom
```

Lets add a new Data disk to our "test-system." Back in the Azure portal viewing the "test-system" virtual machine, click on "Disks" in the left-hand navigation menu. In the page that loads, click the "Create and attach a new data disk" button in the middle of the screen.

Name the new disk "Data," set the size to 4 Gb and click "Save." Note, although this page will allow you to provision a disk of any size, you can only make a disk from a snapshot that matches one of the sizes listed here:

[https://docs.microsoft.com/en-us/azure/virtual-machines/disks-types#disk-size](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-types#disk-size)

As we noted in AWS, if the cloned drive is not the same size as the source drive the SHA1 hashes will not match.

Back in the SSH session with the "test-system" VM, run the `lsblk` command and observe the changes:

```
azureuser@test-system:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0     7:0    0 70.3M  1 loop /snap/lxd/21029
loop1     7:1    0 55.4M  1 loop /snap/core18/2128
loop2     7:2    0 32.3M  1 loop /snap/snapd/12883
sda       8:0    0   30G  0 disk
├─sda1    8:1    0 29.9G  0 part /
├─sda14   8:14   0    4M  0 part
└─sda15   8:15   0  106M  0 part /boot/efi
sdb       8:16   0    4G  0 disk
└─sdb1    8:17   0    4G  0 part /mnt
sdc       8:32   0    4G  0 disk
sr0      11:0    1  628K  0 rom
```

We can see that the new volume is "sdc" and as with AWS, the volume has not been mounted and is not formatted. Let's do that and add our sample data as before:

```
sudo mkfs.ext4 /dev/sdc
sudo mkdir /data
sudo mount /dev/sdc /data
sudo chown azureuser:azureuser /data
echo "Here is a bunch of evil" > /data/evil2.txt
sudo dd if=/dev/urandom bs=512 count=2 > /data/evil3.txt
ls -l /data
```

Next we will unmount our new "Data" volume to prevent any accidental changes to the volume and calculate its SHA1 hash.

```
sudo umount /data
sudo sha1sum /dev/sdc
```

The results will look similar to the following except your hash will be different:

```
azureuser@test-system:~$ sudo umount /data
azureuser@test-system:~$ sudo sha1sum /dev/sdc
f336602acb9916122c56fdcaabc19230b26f9d99  /dev/sdc
```

### Azure - Clone the Data Volume

Following our test process, we will clone the data volume by making a snapshot of the volume and then make a new volume from the copy. In the Azure portal, click on the "Data" disk and it will load the page with all of the disks details. Click the "Create Snapshot" button at the top of the page. Name the snapshot instance "Evidence" and click "Review" and then "Create."

Once the snapshot has completed, click the blue "Go to Resource" button. At the top of the snapshot page, there is a "Create disk" button. Press it. Name the disk to be created "evidence-disk" and ensure that the size is set to the same size as the source disk, which in this case is 4 Gb.  

### Azure - Attach Evidence Volume to the Forensic Workstation    

In the Azure portal, select the "forensic-workstation" VM. In the left-hand navigation menu select "Disks" and that will load the page with the disks detail. This time, instead of clicking the "Create and attach a new data disk" button, we will click the "Attach existing disks" button.

In the Disk name field, start to type the name "evidence-disk" and it should appear as a selection. It may take Azure an inordinate amount of time for it to appear. Sometimes refreshing the page helps. If it never does, make sure your disks and forensic-workstation are in the same region as the test-system. Click "Save" once the existing "evidence-disk" has been selected.

In the SSH session to the Forensic Workstation, run `lsblk` and note that there is an unmounted "sdc" device with no partitions:

```
azureuser@forensic-workstation:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0     7:0    0 70.3M  1 loop /snap/lxd/21029
loop1     7:1    0 55.4M  1 loop /snap/core18/2128
loop2     7:2    0 32.3M  1 loop /snap/snapd/12883
sda       8:0    0   30G  0 disk
├─sda1    8:1    0 29.9G  0 part /
├─sda14   8:14   0    4M  0 part
└─sda15   8:15   0  106M  0 part /boot/efi
sdb       8:16   0    4G  0 disk
└─sdb1    8:17   0    4G  0 part /mnt
sdc       8:32   0    4G  0 disk
sr0      11:0    1  628K  0 rom
azureuser@forensic-workstation:~$
```

Ok, now to calculate the hash:

```
azureuser@forensic-workstation:~$ sudo sha1sum /dev/sdc
f336602acb9916122c56fdcaabc19230b26f9d99  /dev/sdc
```

And the good news is that they match!

## GCP

Now for GCP.

### GCP Setup

Create a new project called "hitc" and then launch two Compute Engine instances one at a time. Name one "test-system" and the other "forensic-workstation." Put both instances in the same zone with in a region near you. An e2-medium type is sufficient for our needs. To mix things up, let's keep the default Debian image.  

If you do not have a SSH key pair, run `ssh-keygen` to generate a public/privare key pair. Expand the "Management, security, disks, networking, sole tenancy" form, click the security tab of that form, and copy & paste the contents of public key into the "Enter public SSH key" field. Be sure to set the username on the end of the string to the name you want to use authenticate with. For this example use "hitc."

SSH into both Virtual Machines. In GCP, like with Azure, the hostname is already set for us.

### GCP - Attach a Data Volume to the Test System

In the SSH session with the "test-system" instance, run the `lsblk` command and observe that there is a single volume "sda" with partitions.

```
ken@test-system:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda       8:0    0   10G  0 disk
├─sda1    8:1    0  9.9G  0 part /
├─sda14   8:14   0    3M  0 part
└─sda15   8:15   0  124M  0 part /boot/efi
```

Now to add a new Data disk to our "test-system." In the GCP Web console, click on the "test-system" and its detail page will open. Click on the "EDIT" button at the top of the page and then scroll down to the "Additional Disks" section. This functionality operates similar to Azure. Click the "Add New Disk" button.

Name the new disk "Data," set the size to the minimum, which is 10 Gb and click "Done" to close the form and then click the blue "Save" button to commit the change to the VM.  

Back in the SSH session with the "test-system" VM, run the `lsblk` command and observe the changes:

```
ken@test-system:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda       8:0    0   10G  0 disk
├─sda1    8:1    0  9.9G  0 part /
├─sda14   8:14   0    3M  0 part
└─sda15   8:15   0  124M  0 part /boot/efi
sdb       8:16   0   10G  0 disk
```

We can see that the new volume is "sdb" and as with AWS, the volume has not been mounted and is not formatted. Let's do that and add our sample data as before:

```
sudo mkfs.ext4 /dev/sdb
sudo mkdir /data
sudo mount /dev/sdb /data
sudo chown hitc:hitc /data
echo "Here is a bunch of evil" > /data/evil2.txt
sudo dd if=/dev/urandom bs=512 count=2 > /data/evil3.txt
ls -l /data

```

Next we will unmount our new "Data" volume to prevent any accidental changes to the volume and calculate its SHA1 hash.

```
sudo umount /data
sudo sha1sum /dev/sdb
```

The results will look similar to the following except your hash will be different:

```
hitc@test-system:~$ sudo sha1sum /dev/sdb
7e31bed463ea8f676e7b33d2fb81a54b1fa771df  /dev/sdb
```

### GCP - Clone the Data Volume

In GCP there is two different options. You can make a snapshot of volume and then a disk from the snapshot as with AWS and Azure, or there is a "Clone Drive" easy button. For consistency sake, we will continue to make a snapshot of the volume and then make a new volume from the snapshot.

In the left-hand navigation menu of the GCP cloud console, click on "Disks" and this will load a page with all three disks that we have created so far. Click on the "Data" disk to open a page with its details. At the top are various buttons including "CREATE SNAPSHOT", "CREATE IMAGE", and "CLONE DISK."

Click the "CREATE SNAPSHOT" button. Name the snapshot "evidence" and click the "Create" button. Upon completion, it will load the Snapshots page.

Now click "Disks" in the left-hand navigation menu. Click "CREATE DISK" at the top of the page that loads. Name the disk to be created "evidence" and selct "snapshot" in the "Disk source type" field. Select the "evidence" snapshot in the "Source snapshot" field. **IMPORTANT: ** in the "Size" field, be sure to set the size to match the source disk. Then, click the blue "Create" button.

### GCP - Attach Evidence Volume to the Forensic Workstation    

Open up the details page for the "forensic-workstation" VM and click the "EDIT" button. Scroll down to the "Additional Disks" section. This time, click "Attach existing disk" and select the "evidence" disk we just created. Change the mode to "Read only" and click "Done" to close the form and "Save" to commit the change to the VM.

In the SSH session to the GCE Forensic Workstation, run `lsblk` and note that there is a 10 Gb unmounted "sdb" device with no partitions:

```
hitc@forensic-workstation:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda       8:0    0   10G  0 disk
├─sda1    8:1    0  9.9G  0 part /
├─sda14   8:14   0    3M  0 part
└─sda15   8:15   0  124M  0 part /boot/efi
sdb       8:16   0   10G  1 disk
```

Ok, now to calculate the hash:

```
hitc@forensic-workstation:~$ sudo sha1sum /dev/sdb
7e31bed463ea8f676e7b33d2fb81a54b1fa771df  /dev/sdb
```

And notice that this hash also matches the SHA1 hash from the test system.

## Wrap up

Well, there you go. We have demonstrated that the process to snapshot a virtual hard drive and to make a duplicate volume from the snapshot is a true forensic bit copy of the original volume, assuming that the volumes are the same size. We performed this test in AWS, Azure, and GCP illustrating the technique with each cloud's user interface.

If you have thoughts or comments on today’s episode, feel free to chime in on the comments for this YouTube video.

If you appreciate this video and want to see more like it, be sure to give it a "thumbs up."

Stay tuned for another installment of “Head in the Clouds” as announcements of new episodes are made on the SANS Cloud Security Twitter feed.

Meanwhile, be sure to check out the other great videos on the [SANS Cloud Security YouTube Channel](https://www.youtube.com/c/SANSCloudSecurity).

Take care.
