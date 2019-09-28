---
layout: post
title: "AWS Testing Part 1: Configuring the environment"
categories: AWS
---

With the continuous digital transformation we are experiencing, a lot of business are moving their assets to Cloud environments. One of them is **Amazon Web Services (AWS)**, with a huge number of clients getting there critical infrastuctures and so.

These will be a series of short posts with the purpose of being a reliable source to start with *Auditing or pentesting* Cloud assets on AWS. The first post will talk about configuring a command-line client provided by AWS that we will use to test these assets.

Cloud pentesting differs from classical pentesting on some points. For some of this points, we can use tools and techniques that we already know, but on the other hand, we will need some new tools to work with AWS specific items. With **awscli** tool we can test some of this points like S3 Buckets.

## AWS Command-line (AWSCLI)
### PreRequisites
1. AWS Account (Free tier is valid)
2. Python with Pip

### Installation

[Github repo with AWSCLI](https://github.com/aws/aws-cli)

We can install it on a virtual-env or doing a global install by issuing
```bash
$ pip install awscli
```

And by default, command completion is not enabled but we can enable it this way
```bash
$ complete -C aws_completer aws
```

### Configuration
#### Step One: Creating the IAM user
We will need to create first a IAM user on our AWS account with permissions to work with console tools.
To do this, on our AWS account we go to *My security credentials -> Users* and create a new User.

![IAM-Roles-1](/images/aws-testing/iam_1.png)

We will follow the user creation wizard, enabling **Console Access** to use the AWS Cli

![IAM-Roles-2](/images/aws-testing/iam_2.png)

Then, as this will be a Pentest environment only, we won't have any resources more than this user so we give him **Administrator** Access to AWS 

![IAM-Roles-3](/images/aws-testing/iam_3.png)
> If we are using a shared environment or we have assets on AWS that we will isolate from this IAM user, we can grant him only the permissions we want like EC2 Read only or S3 Related.

Last step is saving the keys associated to this user for later use on the AWSCLI. **IMPORTANT:** Save this on a safe location, if they get compromised they need to be regenerated and disable the compromised ones.

![IAM-Roles-4](/images/aws-testing/iam_4.png)
> Don't try this creds, this user was for testing purposes only :)

#### Step Two: Setting up the AWSCLI
Now that we have an IAM console user, we go back to our terminal with awscli installed and we run this
```bash
$ aws configure 
AWS Access Key ID [None]: #<Access Key ID for our User>
AWS Secret Access Key [None]: #<Secret Key for our User> 
Default region name [None]: #<Region where we want to work> Example: eu-west-3 
Default output format [None]: #Possible values text,json,table
```

We already have configured our AWSCLI! In the following posts, we will use this tool to check permissions on S3 Buckets, and checking meta-data for a compromised EC2 Machines.