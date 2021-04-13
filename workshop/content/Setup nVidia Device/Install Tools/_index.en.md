+++
title = "Install Tools"
weight = 12
+++

These steps will guide you through installing the necessary tools on your development host and the target device. You will install:

* AWS Command Line Interface (CLI)

### Step 1: Install AWS CLI on developmet host

On your host, install the [AWS CLI version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)

> **Linux (x86_64)**

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

> **MacOS**

```bash
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```

### Step 2: Configure the AWS CLI

1. Find the ACCESS KEY ID and SECRET ACCESS KEY for a role in your AWS account that has permissions to create IAM, IoT, and S3 resources. These are typically provided when the account is created and often delivered in a CSV file. If you cannot locate them, contact your account administrator or create new credentials from the IAM console.

2. Add the credentials to your AWS profile by typing

```bash
aws configure
# optionally add -profile <profile-name> 
```

When prompted provide the Access Key and Secret Access Key. For the purposes of this workshop, set the region to `us-west-2` and `json` for the output format. The `aws configure` command can be reused to verify that values have been set properly.

```bash
aws configure                                                            

# at the prompt, provide a new value or press return to accept the current value
AWS Access Key ID [****************MNO5]: 
AWS Secret Access Key [****************RgDp]: 
Default region name [us-west-2]: 
Default output format [json]: 
```

### Step 3: Verify configuration

Verify the functionality of the CLI and the configuration with the command:

```bash
aws sts get-caller-identity
```

Verify the UserId and Account values are correct in the output.

```bash
{
    "UserId": "AID----BMW",
    "Account": "11-----16",
    "Arn": "arn:aws:iam::11-----16:user/s---c"
}
```