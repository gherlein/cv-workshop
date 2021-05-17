+++
title = "Install"
date = 2021-04-13T11:06:06-07:00
weight = 21
chapter = true
+++


This lab will guide you to set up AWS IoT Greengrass v2 on the device you previously set up. 

### Step 1: Connect to the target device

> **Using SSH**

```bash
ssh <user>@<device>
```

> **With Keybord and Monitor**

Open the terminal app.

### Step 2: Install Corretto (Java)

Enter these commands into the device shell:

(Copy and paste one line at a time and enter password for `sudo` when prompted.)

```bash
wget -O- https://apt.corretto.aws/corretto.key | sudo apt-key add - 

sudo add-apt-repository 'deb https://apt.corretto.aws stable main'

sudo apt-get update; sudo apt-get install -y java-1.8.0-amazon-corretto-jdk
```

Validate the successful installation of Java with

```bash
java -version
```

The output should look similar to:

```bash
openjdk version "1.8.0_282"
OpenJDK Runtime Environment Corretto-8.282.08.1 (build 1.8.0_282-b08)
OpenJDK 64-Bit Server VM Corretto-8.282.08.1 (build 25.282-b08, mixed mode)
```

_NB- This workshop was developed with Corretto v 1.8, but other JREs may work just as well_

### Step 3: Download Greengrass v2

```bash
curl -s https://d2s8p88vqu9w66.cloudfront.net/releases/greengrass-nucleus-latest.zip > greengrass-nucleus-latest.zip

unzip greengrass-nucleus-latest.zip -d GreengrassCore && rm greengrass-nucleus-latest.zip
```

### Step 4: Set environment variables

Greengrass v2 has the ability to self-install and configure, but needs access credentials. Additionally, this installer will create an IoT thing, thing group, and IAM roles. Provide these values through shell variables. The AWS CLI credentials can be the same as you used in the host setup or can be newly created for just this role. The installer will not save these values. Using shell variables eliminates the need to install the CLI on the device.

| shell var | description |
| --- | --- |
| AWS_ACCESS_KEY_ID | ID for account to create resources |
| AWS_SECRET_ACCESS_KEY | secret key |
| AWS_DEFAULT_REGION | region to use -- `us-west-2` for this workshop |
| _name for things created by the installer_ |
| THING_NAME | device name for this Greengrass Core |
| GROUP_NAME | name for the Greengrass Group |
| ROLE_NAME | name of IAM role for the core |
| ROLE_ALIAS | alias used to refer to role to allow role to change |


```bash
# replace these with your credentials
export AWS_ACCESS_KEY_ID=<paste_your_id_here>
export AWS_SECRET_ACCESS_KEY=<paste_your_key_here>

# modify any of these as desired, or copy/paste as is
export AWS_DEFAULT_REGION=us-west-2

# the installer will create the resources with these names
export THING_NAME=jetson
export GROUP_NAME=jetsonGroup
export ROLE_NAME=MyGreengrassV2Role
export ROLE_ALIAS=MyGreengrassCoreTokenExchangeRoleAlias
```

### Step 5: Run the installer

```bash
sudo -E java -Dlog.store=FILE \
  -jar ./GreengrassCore/lib/Greengrass.jar \
  --aws-region $AWS_DEFAULT_REGION \
  --root /greengrass/v2 \
  --thing-name $THING_NAME \
  --thing-group-name $GROUP_NAME \
  --tes-role-name $ROLE_NAME \
  --tes-role-alias-name $ROLE_ALIAS \
  --component-default-user ggc_user:ggc_group \
  --provision true \
  --setup-system-service true

sudo chmod 755 /greengrass/v2 && sudo chmod 755 /greengrass
```

Your Greengrass core is now set up and ready to go. Continue on to the next step to validate the installation.