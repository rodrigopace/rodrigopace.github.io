---
layout: page
name: Episode 11 - Importing Resources into the Terraform State File

---

<div class="video-container"><iframe width="560" height="315" src="https://www.youtube.com/embed/QpI_9NY1Cy0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

# Episode 11 - Importing Resources into the Terraform State File

Welcome to Episode 11 of the “Head in the Clouds” Video Series. I am Ken Hartman, a SANS Certified Instructor and content creator for the SANS Cloud Security Curriculum.

Today’s episode is titled: “Importing Resources into the Terraform State File”

The purpose of HITC is to teach foundational cloud skills and security knowledge that will help others thrive in the cloud. The content ideas come from my personal observation of skills that I see some students lacking when they show up to a SANS Cloud Security course. Other ideas are passed on from fellow SANS instructors.

The idea for today's content came from teaching [SANS SEC510: Public Cloud Security: AWS, Azure, and GCP](https://www.sans.org/cyber-security-courses/public-cloud-security-aws-azure-gcp/) where we use various Terraform scripts to deploy assets to AWS, Azure, and GCP throughout the week. Occasionally during the class a student's Terraform state file may become out of synch with the resources that the student has deployed. The most frequent cause of this is a transient network connectivity issue where Terraform makes an API call to a cloud service but Terraform doesn't get the response and therefore does not update its state, even though the cloud service actually did provision the resource.

The solution to this situation is to import the resource into the Terraform state file, and hence that is the topic for today's episode. Today, we will cover how to import various resources into each of the big three cloud service providers and also how to import when your Terraform scripts use modules.

## Set Up

In this episode, we will use the same GitHub repository that we used in Episode 10 - [https://github.com/Resistor52/tf-vm_in3csps](https://github.com/Resistor52/tf-vm_in3csps). In Episode 10 we covered how to deploy a virtual machine in AWS, Azure, and GCP.  To follow along with this episode, be sure to complete the setup, authentication, and deployment process for each module that we covered in Episode 10.

## An AWS Example

After you have completed all the activities for Episode 10, we are now ready to play with AWS more. Change into the directory for the AWS module:

```
cd ~/tf-vm_in3csps/modules/aws
```

To start, let's deploy the AWS assets. Make sure that the `terraform.tfvars` file is set up for your AWS environment. Since I am using "us-east-1" and the other defaults look good, I can just copy the `terraform.tfvars.example` file.

```
cp terraform.tfvars.example terraform.tfvars
```

With that done, I can initialize Terraform to download the AWS provider and then run `apply` to provision the assets:

```
terraform init
terraform apply

```

Once Terraform completes, take a look at the security group that was just added:

```
aws ec2 describe-security-groups --filter "Name=group-name,Values=hitc-sg"
```

This CLI command will produce output similar to the following:

```
{
    "SecurityGroups": [
        {
            "Description": "Managed by Terraform",
            "GroupName": "hitc-sg",
            "IpPermissions": [
                {
                    "FromPort": 22,
                    "IpProtocol": "tcp",
                    "IpRanges": [
                        {
                            "CidrIp": "3.82.94.254/32"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "ToPort": 22,
                    "UserIdGroupPairs": []
                }
            ],
            "OwnerId": "690634326977",
            "GroupId": "sg-03d802dd559b6dd49",
            "IpPermissionsEgress": [
                {
                    "IpProtocol": "-1",
                    "IpRanges": [
                        {
                            "CidrIp": "0.0.0.0/0"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "UserIdGroupPairs": []
                }
            ],
            "VpcId": "vpc-018dd59eb5d725d61"
        }
    ]
}
```

**NOTE:** In this episode, we will _intentionally_ add resources to the cloud service provider directly to _intentionally_ make the state file out of synch with what is deployed. The best practice pattern is to make changes to your infrastructure only by modifying your terraform *.tf files and applying those changes.

Next we will add in a security group rule for HTTP (port 80) to our “hitc-sg” security group using the authorize-security-group-ingress CLI command. However, we need to supply the Security Group ID. To determine the Security Group ID, run the following command:

```
aws ec2 describe-security-groups --filter "Name=group-name,Values=hitc-sg" --query SecurityGroups[].GroupId --output text
```

Ok, now we can run that as a subshell where the authorize-security-group-ingress CLI command asks for the security group:

```
aws ec2 authorize-security-group-ingress --group-id $(aws ec2 describe-security-groups --filter "Name=group-name,Values=hitc-sg" --query SecurityGroups[].GroupId --output text) --protocol tcp --port 80 --cidr 0.0.0.0/0
```

This command produces the following output:

```
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-015282b295b573fbd",
            "GroupId": "sg-03d802dd559b6dd49",
            "GroupOwnerId": "690634326977",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}
```

We can confirm the change to the security group by re-running:

```
aws ec2 describe-security-groups --filter "Name=group-name,Values=hitc-sg"
```

which will output something like:

```
{
    "SecurityGroups": [
        {
            "Description": "Managed by Terraform",
            "GroupName": "hitc-sg",
            "IpPermissions": [
                {
                    "FromPort": 80,
                    "IpProtocol": "tcp",
                    "IpRanges": [
                        {
                            "CidrIp": "0.0.0.0/0"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "ToPort": 80,
                    "UserIdGroupPairs": []
                },
                {
                    "FromPort": 22,
                    "IpProtocol": "tcp",
                    "IpRanges": [
                        {
                            "CidrIp": "3.82.94.254/32"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "ToPort": 22,
                    "UserIdGroupPairs": []
                }
            ],
            "OwnerId": "690634326977",
            "GroupId": "sg-03d802dd559b6dd49",
            "IpPermissionsEgress": [
                {
                    "IpProtocol": "-1",
                    "IpRanges": [
                        {
                            "CidrIp": "0.0.0.0/0"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "UserIdGroupPairs": []
                }
            ],
            "VpcId": "vpc-018dd59eb5d725d61"
        }
    ]
}
```

Now what happens when we run terraform apply?

We get messages that include the following:

* "Terraform detected the following changes made outside of Terraform since the last 'terraform apply'"
* "Unless you have made equivalent changes to your configuration, or ignored the relevant attributes using ignore_changes, the following plan may include actions to undo or respond to these changes."
* "Your configuration already matches the changes detected above. If you'd like to update the Terraform state to match, create and apply a refresh-only plan: terraform apply -refresh-only"

Since we are just playing around, give that a try. Run:

```
terraform apply --refresh-only
```

When we do so, we get the message:


```
No changes. Your infrastructure still matches the configuration.

Terraform has checked that the real remote objects still match the result of your most recent changes, and found no differences.

Would you like to update the Terraform state to reflect these detected changes?
  Terraform will write these changes to the state without modifying any real infrastructure.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value:
```

If we respond "yes" to the prompt, Terraform will update the state file to include our new security group rule. Then if we run `terraform apply` once more, we will get the message:

```
No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
```

**Question:** Does the `--refresh-only` option solve our problem?

No, because our TF code is still our of whack with what is deployed. Our goal should be to have a three way match between:

* What is deployed in the cloud
* What is captured in the Terraform state file
* What is indicated in the *.TF files

The problem is that our infrastructure as code does not know about this change. Let’s fix that.

Open up the `~/tf-vm_in3csps/module/aws/network.tf` file and insert the following resource block at the end of the file:

```
resource "aws_security_group_rule" "sg-http" {
  type              = "ingress"
  from_port         = 80
  to_port           = 80
  protocol          = "tcp"
  cidr_blocks       = ["0.0.0.0/0"]
  security_group_id = aws_security_group.hitc-sg.id
}
```

What happens when we run `terraform apply`?

We get a message that includes the following at the end:

```
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_security_group_rule.sg-http will be created
  + resource "aws_security_group_rule" "sg-http" {
      + cidr_blocks              = [
          + "0.0.0.0/0",
        ]
      + from_port                = 80
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = "sg-03d802dd559b6dd49"
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 80
      + type                     = "ingress"
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
```

However, when we respond "yes" to the prompt, we get an error that reads:

```
╷
│ Error: [WARN] A duplicate Security Group rule was found on (sg-03d802dd559b6dd49). This may be
│ a side effect of a now-fixed Terraform issue causing two security groups with
│ identical attributes but different source_security_group_ids to overwrite each
│ other in the state. See https://github.com/hashicorp/terraform/pull/2376 for more
│ information and instructions for recovery. Error: InvalidPermission.Duplicate: the specified rule "peer: 0.0.0.0/0, TCP, from port: 80, to port: 80, ALLOW" already exists
│       status code: 400, request id: 40951575-b544-415c-ad51-6264ccb3aaf6
│
│   with aws_security_group_rule.sg-http,
│   on network.tf line 73, in resource "aws_security_group_rule" "sg-http":
│   73: resource "aws_security_group_rule" "sg-http" {
│
```

Although there is a note in the error message indicating a bug that Hashicorp fixed, we know the reason for this error--**_we caused it_**.

There are two possible fixes:
* One way to fix the issue is to undo the manual change. Sometimes this may be the most expedient approach.
* Alternatively, we can import the security group rule into the state file and that is what we will do.

Let's pause for a moment and discuss a generic approach to determining how to import a resource into terraform. First off, recognize that each cloud service requires a different **provider** (plugin) so that Terraform knows how to call that cloud provider's API. Also, each resource block may require different data to be added to the import command. Therefore, it is imperative to consult the online documentation for the provider to determine the syntax of the import command for the particular resource.

**TIP:** To find the Terraform provider documentation, use the following search terms "terraform import" and the resource that you need to import.

As an example, since we need to import a `aws_security_group_rule`, we would search for "terraform import aws_security_group_rule" and the top result will most likely lead you to the following page:

* [https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule#import](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule#import)

Scrolling down to the "import" section, we see several examples of import commands, but it looks like the top one is closest to what we are looking for:

```
terraform import aws_security_group_rule.ingress sg-6e616f6d69_ingress_tcp_8000_8000_10.0.3.0/24
```

Considering the above example, we need to substitute in our security group id as well as the desired port and CIDR block. This results in:

```
terraform import aws_security_group_rule.sg-http sg-03d802dd559b6dd49_ingress_tcp_80_80_0.0.0.0/0
```

Of course your security group ID will be different. Conveniently, the providers error message typically provides the information that you need to supply to the import statement, such as the security group id.

Running this command results in the following message:

```
aws_security_group_rule.sg-http: Importing from ID "sg-03d802dd559b6dd49_ingress_tcp_80_80_0.0.0.0/0"...
aws_security_group_rule.sg-http: Import prepared!
  Prepared aws_security_group_rule for import
aws_security_group_rule.sg-http: Refreshing state... [id=sg-03d802dd559b6dd49_ingress_tcp_80_80_0.0.0.0/0]

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
```

Running `terraform apply` shows that there are no changes needed.
Ok, run `terraform destroy` to remove the infrastructure in AWS

## An Azure Example

Next, lets play with Azure.

Change into the directory for the Azure module and if needed modify terraform.tfvars:

```
cd ../azure
cp terraform.tfvars.example terraform.tfvars
```

Next, as before with AWS, initialize terraform and deploy the Azure infrastructure.

```
terraform init
terraform apply
```

For fun, let's manually add a subnet. Fist let’s show the subnet in our VNet:

```
az network vnet subnet list --resource-group hitc --vnet-name hitc-vnet
```

This shows the following output:

```
[
  {
    "addressPrefix": "10.0.2.0/24",
    "addressPrefixes": null,
    "applicationGatewayIpConfigurations": null,
    "delegations": [],
    "etag": "W/\"d9404f52-c63e-45a4-ab0c-22c94828ddcb\"",
    "id": "/subscriptions/911d03ca-d335-4697-8254-024980a4f55a/resourceGroups/hitc/providers/Microsoft.Network/virtualNetworks/hitc-vnet/subnets/hitc-subnet",
    "ipAllocations": null,
    "ipConfigurationProfiles": null,
    "ipConfigurations": [
      {
        "etag": null,
        "id": "/subscriptions/911d03ca-d335-4697-8254-024980a4f55a/resourceGroups/hitc/providers/Microsoft.Network/networkInterfaces/hitc-nic/ipConfigurations/myNicConfiguration",
        "name": null,
        "privateIpAddress": null,
        "privateIpAllocationMethod": null,
        "provisioningState": null,
        "publicIpAddress": null,
        "resourceGroup": "hitc",
        "subnet": null
      }
    ],
    "name": "hitc-subnet",
    "natGateway": null,
    "networkSecurityGroup": null,
    "privateEndpointNetworkPolicies": "Enabled",
    "privateEndpoints": null,
    "privateLinkServiceNetworkPolicies": "Enabled",
    "provisioningState": "Succeeded",
    "purpose": null,
    "resourceGroup": "hitc",
    "resourceNavigationLinks": null,
    "routeTable": null,
    "serviceAssociationLinks": null,
    "serviceEndpointPolicies": null,
    "serviceEndpoints": [],
    "type": "Microsoft.Network/virtualNetworks/subnets"
  }
]
```

Good, now add a second subnet:

```
az network vnet subnet create --resource-group hitc --vnet-name hitc-vnet --address-prefixes 10.0.3.0/24 --name "hitc-subnet2"
```

This CLI command produces:

```
{
  "addressPrefix": "10.0.3.0/24",
  "addressPrefixes": null,
  "applicationGatewayIpConfigurations": null,
  "delegations": [],
  "etag": "W/\"de7f4cf4-0202-46bf-aecc-ba52152abd3a\"",
  "id": "/subscriptions/911d03ca-d335-4697-8254-024980a4f55a/resourceGroups/hitc/providers/Microsoft.Network/virtualNetworks/hitc-vnet/subnets/hitc-subnet2",
  "ipAllocations": null,
  "ipConfigurationProfiles": null,
  "ipConfigurations": null,
  "name": "hitc-subnet2",
  "natGateway": null,
  "networkSecurityGroup": null,
  "privateEndpointNetworkPolicies": "Enabled",
  "privateEndpoints": null,
  "privateLinkServiceNetworkPolicies": "Enabled",
  "provisioningState": "Succeeded",
  "purpose": null,
  "resourceGroup": "hitc",
  "resourceNavigationLinks": null,
  "routeTable": null,
  "serviceAssociationLinks": null,
  "serviceEndpointPolicies": null,
  "serviceEndpoints": null,
  "type": "Microsoft.Network/virtualNetworks/subnets"
}
```

Now lets modify `~/tf-vm_in3csps/modules/azure/network.tf`

Add the following block under the existing “hitc-subnet” resource block. (It should be the third resource block down in the file.)

```
resource "azurerm_subnet" "hitc-subnet2" {
  name                 = "hitc-subnet2"
  resource_group_name  = azurerm_resource_group.hitc.name
  virtual_network_name = azurerm_virtual_network.hitc-vnet.name
  address_prefixes     = ["10.0.3.0/24"]
}

```
Now, when we run `terraform apply` we get the following error:


```
╷
│ Error: A resource with the ID "/subscriptions/911d03ca-d335-4697-8254-024980a4f55a/resourceGroups/hitc/providers/Microsoft.Network/virtualNetworks/hitc-vnet/subnets/hitc-subnet2" already exists - to be managed via Terraform this resource needs to be imported into the State. Please see the resource documentation for "azurerm_subnet" for more information.
│
│   with azurerm_subnet.hitc-subnet2,
│   on network.tf line 15, in resource "azurerm_subnet" "hitc-subnet2":
│   15: resource "azurerm_subnet" "hitc-subnet2" {
│
╵
```


Do an internet search for “terraform import azurerm_subnet” leads you to [https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/subnet#import](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/subnet#import)

The Terraform provider documentation gives the following example:

```
terraform import azurerm_subnet.exampleSubnet /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/mygroup1/providers/Microsoft.Network/virtualNetworks/myvnet1/subnets/mysubnet1
```

Making the appropriate substitutions, we get:

```
terraform import azurerm_subnet.hitc-subnet2 /subscriptions/911d03ca-d335-4697-8254-024980a4f55a/resourceGroups/hitc/providers/Microsoft.Network/virtualNetworks/hitc-vnet/subnets/hitc-subnet2
```

Naturally, your subscription ID will be different than mine.

Great, running that import command produces the following result:

```
azurerm_subnet.hitc-subnet2: Importing from ID "/subscriptions/911d03ca-d335-4697-8254-024980a4f55a/resourceGroups/hitc/providers/Microsoft.Network/virtualNetworks/hitc-vnet/subnets/hitc-subnet2"...
azurerm_subnet.hitc-subnet2: Import prepared!
  Prepared azurerm_subnet for import
azurerm_subnet.hitc-subnet2: Refreshing state... [id=/subscriptions/911d03ca-d335-4697-8254-024980a4f55a/resourceGroups/hitc/providers/Microsoft.Network/virtualNetworks/hitc-vnet/subnets/hitc-subnet2]

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
```

As you can see, success is all about modifying the example contained in the documentation for the Terraform provider.

Well, that's our Azure example. We can tear it down using `terraform destroy`

## A GCP Example

Ok, to mix things up, let's mess with the GCP state file. Change into the "gcp" directory. Remember you may
need to update the PROJECT_ID in the terraform.tfvars file to match the project you created during the Lab Setup.

```
cd ../gcp
cp terraform.tfvars.example terraform.tfvars
```

Now,

Then:

```
terraform init
terraform apply
```

What happens if we remove the Google Compute Instance from the state file?

```
terraform state rm google_compute_instance.vm_instance
```

This outputs:

```
Removed google_compute_instance.vm_instance
Successfully removed 1 resource instance(s).
```

This command removes the resource from the state file but note that the VM is still running. To confirm, just take a look in the GCP console.

Now, lets make terraform throw an error by trying to run `terraform apply`

We get:

```
╷
│ Error: Error creating instance: googleapi: Error 409: The resource 'projects/tf-gcp-324518/zones/us-central1-a/instances/hitc-gcp' already exists, alreadyExists
│
│   with google_compute_instance.vm_instance,
│   on vm.tf line 1, in resource "google_compute_instance" "vm_instance":
│    1: resource "google_compute_instance" "vm_instance" {
│
```

Perform an internet search for “terraform import google_compute_instance”

The documentation gives the following example:

```
terraform import google_compute_instance.default projects/{{project}}/zones/{{zone}}/instances/{{name}}
```

Making the appropriate substitutions:

```
terraform import google_compute_instance.vm_instance projects/tf-gcp-324518/zones/us-central1-a/instances/hitc-gcp
```

And this produces the following result:

```
google_compute_instance.vm_instance: Importing from ID "projects/tf-gcp-324518/zones/us-central1-a/instances/hitc-gcp"...
google_compute_instance.vm_instance: Import prepared!
  Prepared google_compute_instance for import
google_compute_instance.vm_instance: Refreshing state... [id=projects/tf-gcp-324518/zones/us-central1-a/instances/hitc-gcp]

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.

```

Now when we run `terraform apply` it shows no changes need to be made and it does not throw an error.

Go ahead and run `terraform destroy`

## An Example using a Module

Ok, there is one more item I want to cover and that is how things change with importing when there is a module involved.

Change back to the root level directory of our terraform code.

```
cd ../..
```

If you customized your terraform.tfvars files for each module, you can create a root level tfvars file using the following command:

```
cat modules/aws/terraform.tfvars modules/azure/terraform.tfvars modules/gcp/terraform.tfvars | sort | uniq > terraform.tfvars
```

Good, now let's initialize terraform and deploy our resources:

```
terraform init
terraform apply
```

At the end of the deployment we should see something like:

```
Apply complete! Resources: 23 added, 0 changed, 0 destroyed.

Outputs:

aws_ip = "3.84.223.103"
aws_ssh_connection_string = "ssh -i ~/.ssh/id_rsa ubuntu@3.84.223.103"
az_ip = "52.173.16.243"
az_ssh_connection_string = "ssh -i ~/.ssh/id_rsa ubuntu@52.173.16.243"
gcp_ip = "35.184.117.84"
gcp_ssh_connection_string = "ssh -i ~/.ssh/id_rsa ubuntu@35.184.117.84"
```

Note that your IP addresses will be different.

Now that we have deployed everything. Lets remove the GCE instance from the state file.

```
terraform state rm module.gcp.google_compute_instance.vm_instance
```

Note that the Google cloud terraform code is in a directory called “gcp” and this is what gives the module its name. A resource in a module has "module" plus the module name added as part of the “address” to the object in the state file.

Therefore, we need to use "**module.gcp.**google_compute_instance.vm_instance" to reference the resource.  See [https://www.terraform.io/docs/cli/commands/state/rm.html](https://www.terraform.io/docs/cli/commands/state/rm.html) for more information.

The output of this command will be:

```
Removed module.gcp.google_compute_instance.vm_instance
Successfully removed 1 resource instance(s).
```

Now, if we run `terraform apply` we will get an error as expected:

```
╷
│ Error: Error creating instance: googleapi: Error 409: The resource 'projects/tf-gcp-324518/zones/us-central1-a/instances/hitc-gcp' already exists, alreadyExists
│
│   with module.gcp.google_compute_instance.vm_instance,
│   on modules/gcp/vm.tf line 1, in resource "google_compute_instance" "vm_instance":
│    1: resource "google_compute_instance" "vm_instance" {
│
╵
```

Notice anything different about the error message? The portion that reads `with module.gcp.google_compute_instance.vm_instance` indicates the “address” of where we want to import the object into the state file.

Hence, our import command will be:

```
terraform import module.gcp.google_compute_instance.vm_instance projects/tf-gcp-324518/zones/us-central1-a/instances/hitc-gcp
```

And the result of the command should look like:

```
module.gcp.google_compute_instance.vm_instance: Importing from ID "projects/tf-gcp-324518/zones/us-central1-a/instances/hitc-gcp"...
module.gcp.google_compute_instance.vm_instance: Import prepared!
  Prepared google_compute_instance for import
module.gcp.google_compute_instance.vm_instance: Refreshing state... [id=projects/tf-gcp-324518/zones/us-central1-a/instances/hitc-gcp]

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.

```

Now when we run `terraform apply` we can see that it shows that zero resources were added, changed, and destroyed.

## Wrap Up

Well, there you go. Hopefully this episode helps you understand the process of importing a resource into the Terraform state. Just remember to lookup the import syntax as it can vary based on the provider and the resource that you intend to import.

If you have thoughts or comments on today’s episode, feel free to chime in on the comments for this YouTube video.

If you appreciate this video and want to see more like it, be sure to give it a "thumbs up."

Stay tuned for another installment of “Head in the Clouds” as announcements of new episodes are made on the SANS Cloud Security Twitter feed.

Meanwhile, be sure to check out the other great videos on the [SANS Cloud Security YouTube Channel](https://www.youtube.com/c/SANSCloudSecurity).

Take care.
