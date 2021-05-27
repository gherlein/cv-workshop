+++
title = "Validate"
date = 2021-04-13T11:06:22-07:00
weight = 22
chapter = true
+++

Use this lab to validate an installed Greengrass core through inspection of the Management Console and deploying a simple Hello World component.

_(This guide adapted from the [Greengrass Getting Started Guide](https://docs.aws.amazon.com/greengrass/v2/developerguide/getting-started.html#create-first-component))_


### Step 1: Inspect the Console

 Navigate to the [AWS IoT Management Console](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/greengrass/v2/cores) and Click **Greengrass** and then **Core devices** from the left side panel.

{{< img "gg-console.png" "Greengrass Console" >}}

**Verify the core you created in the previous lab is shown with a 'Healthy' status before proceeding.**

### Step 2: Create an S3 bucket and the component artifacts

1. On your development **host**, set the same environment variables as you did on the deployment target:

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

# useful variables here
export acct_num=$(aws sts get-caller-identity --query "Account" --output text)
export bucket_name=greengrass-component-artifacts-$acct_num-$AWS_DEFAULT_REGION
```

2. Create the S3 bucket

```bash
aws s3 mb s3://$bucket_name
```

**NB-** Alternatively, you can use an existing bucket. If so, ensure that you set the environment variable `$bucket_name` to your existing bucket for the remainder of this lab.

3. Create the artifact for the component using `vim` or the text editor of your choice

```bash
mkdir -p ~/GreengrassCore/artifacts/com.example.HelloWorld/1.0.0

vim ~/GreengrassCore/artifacts/com.example.HelloWorld/1.0.0/hello_world.py
```
and enter this content
```python3
import sys
import datetime

message = f"Hello, {sys.argv[1]}! Current time: {str(datetime.datetime.now())}."

# Print the message to stdout.
print(message)

# Append the message to the log file.
with open('/tmp/Greengrass_HelloWorld.log', 'a') as f:
    print(message, file=f)
```

You should now have a file layout similar to this

```bash
~/GreengrassCore/
├── artifacts
│   └── com.example.HelloWorld
│       └── 1.0.0
│           └── hello_world.py
```

4. Copy the artifact to the S3 bucket

```bash
aws s3 sync ~/GreengrassCore/ s3://$bucket_name/
```
and verify the object layout is correct with
```bash
aws s3 ls s3://$bucket_name/ --recursive
```
You should have a result simliar to
```bash
2021-04-13 16:17:54        278 artifacts/com.example.HelloWorld/1.0.0/hello_world.py
```

5.  Set a policy on the bucket allowing Greengrass to read from it

Run this bash script by pasting it directly into a terminal or saving it as a file and running it at the command line.

```bash
#!/bin/bash
(
cat <<POLICY
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::$bucket_name/*"
    }
  ]
}
POLICY
) > policy.json
aws iam create-policy --policy-name MyGreengrassV2ComponentArtifactPolicy --policy-document file://policy.json
aws iam attach-role-policy --role-name $ROLE_NAME --policy-arn arn:aws:iam::$acct_num:policy/MyGreengrassV2ComponentArtifactPolicy
aws iam list-attached-role-policies --role-name $ROLE_NAME
```

You should see something like this:

```
{
    "AttachedPolicies": [
        {
            "PolicyName": "MyGreengrassV2RoleAccess",
            "PolicyArn": "arn:aws:iam::<accountnum>:policy/MyGreengrassV2RoleAccess"
        },
        {
            "PolicyName": "MyGreengrassV2ComponentArtifactPolicy",
            "PolicyArn": "arn:aws:iam::<accountnum>:policy/MyGreengrassV2ComponentArtifactPolicy"
        }
    ]
}
```

### Step 3: Create the Hello World Component

1. Create a directory for the component artifacts.

```bash
mkdir -p ~/GreengrassCore/recipes
```

2.  You can run one script to create the component at once, or do each step one at a time manually.  To run it automatically, paste the following
into a terminal or save it to a file and exectute it:

```bash
#!/bin/bash
json=com.example.HelloWorld-1.0.0.json
(
cat <<RECIPE
{
  "RecipeFormatVersion": "2020-01-25",
  "ComponentName": "com.example.HelloWorld",
  "ComponentVersion": "1.0.0",
  "ComponentDescription": "My first AWS IoT Greengrass component.",
  "ComponentPublisher": "Amazon",
  "ComponentConfiguration": {
    "DefaultConfiguration": {
      "Message": "world"
    }
  },
  "Manifests": [
    {
      "Platform": {
        "os": "linux"
      },
      "Lifecycle": {
        "Run": "python3 -u {artifacts:path}/hello_world.py '{configuration:/Message}'"
      },
      "Artifacts": [
        {
          "URI": "s3://$bucket_name/artifacts/com.example.HelloWorld/1.0.0/hello_world.py"
        }
      ]
    }
  ]
}
RECIPE
) > $json
aws greengrassv2 create-component-version --inline-recipe fileb://$json
```

Or, to manually create a recipe file for the 'HelloWorld' component 

```bash
vim ~/GreengrassCore/recipes/com.example.HelloWorld-1.0.0.json
```

Enter the following content for the recipe, replacing `<paste_bucket_name_here>` with the name of the bucket you created earlier

```json
{
  "RecipeFormatVersion": "2020-01-25",
  "ComponentName": "com.example.HelloWorld",
  "ComponentVersion": "1.0.0",
  "ComponentDescription": "My first AWS IoT Greengrass component.",
  "ComponentPublisher": "Amazon",
  "ComponentConfiguration": {
    "DefaultConfiguration": {
      "Message": "world"
    }
  },
  "Manifests": [
    {
      "Platform": {
        "os": "linux"
      },
      "Lifecycle": {
        "Run": "python3 -u {artifacts:path}/hello_world.py '{configuration:/Message}'"
      },
      "Artifacts": [
        {
          "URI": "s3://<paste_bucket_name_here>/artifacts/com.example.HelloWorld/1.0.0/hello_world.py"
        }
      ]
    }
  ]
}
```

Then manually create the component:

```bash
aws greengrassv2 create-component-version \
  --inline-recipe fileb://~/GreengrassCore/recipes/com.example.HelloWorld-1.0.0.json
```

3.  Regradless of which of the above you did, Verify the command returns without errors similar to

```json
{
    "arn": "arn:aws:greengrass:us-west-2:11-----16:components:com.example.HelloWorld:versions:1.0.0",
    "componentName": "com.example.HelloWorld",
    "componentVersion": "1.0.0",
    "creationTimestamp": "2021-04-13T16:32:48.936000-07:00",
    "status": {
        "componentState": "REQUESTED",
        "message": "NONE",
        "errors": {}
    }
}
```


### Step 4: Deploy the HelloWorld component using the AWS Console

1.  Navigate to the [AWS IoT Management Console](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/greengrass/v2/cores) and Click **Greengrass** and then **Components** from the left side panel.

{{< img "gg-components.png" "Greengrass Components" >}}

**Verify the presence of the component you just created**

2. Click on the **com.example.HelloWorld** link from the My components list, then click, **Deploy**

3. Select **Create new deployment** then click, **Next**

4. Name this deployment, `cv-lab`, select **Core device** for the Target Type, and provide the name of your Greengrass Core for the Target name--e.g. `jetson`.
Click, **Next**

_NB- if you see the error **Target has a configuration**, click the **Revise deployment**. There can be only ONE device-specific deployment, however Thing group deployments can be layered._

5. Verify **com.example.HelloWorld** is checked and click, **Next** and then click, **Next** twice more to the Review screen

6. On the Review screen, verify all looks correct, then click, **Deploy**

### Step 5: Validate the deployment

1. Navigate to the [AWS IoT Management Console](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/greengrass/v2/cores) and Click **Greengrass** and then **Deployments** from the left side panel. Refresh the screen after a couple minutes.

{{< img "deployed.png" "Greengrass Deployments" >}}

**Verify the 'cv-lab' deployment has 'Completed'**

_You can also verify the status of your core is 'Healthy' on the 'Core devices' screen_

2. From the device shell, check the HelloWorld log

> **Using SSH**

```bash
ssh <user>@<device>
```

> **With Keybord and Monitor**

Open the terminal app.

Then
```bash
cat /tmp/Greengrass_HelloWorld.log
```

**Verify the message and timestamp**

### (Optional) Step 6: Clean up

It can be handy to leave the Hello World component in your account to verify basic Greengrass Core communications, but you may wish to remove it from the device.

* To remove the Hello World component from the deployment:

  * From the [AWS IoT Management Console](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/greengrass/v2/cores) and Click **Greengrass** and then **Deployments**. Check **cv-lab** (or whatever name you gave your deployment) and click **Revise**. Click, **Next** then un-check the `com.example.HelloWorld` component and click, **Next**.  Click, **Next** twice more, make sure everything is correct on the Review screen and click, **Deploy**.

  * Verify the python script is removed from the `/greengrass/v2/packages/artifacts-unarchived/com.example.Helloworld` directory on the target device. _You may need to be root to access this directory_  It is normal for the directory structure to remain after components are removed.

  * You can revise the deployment to add this component back at any time if desired or add this component to other deployments as a check of the deployment function.

* To **delete** the component from your account
_(This will remove the ability to re-deploy the component without re-creating it)_:

  * From the [AWS IoT Management Console](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/greengrass/v2/cores) and Click **Greengrass** and then **Components**. Click `com.example.HelloWorld` link, then **Delete version**.

  * _NB- multiple versions of the component can be created and deleted_

  * The artifact will remain present in your S3 bucket

* You can use the S3 console or AWS CLI to delete the artifacts and the bucket if desired.

**Congratulations!! You now created functional Greengrass core and a test component** 
