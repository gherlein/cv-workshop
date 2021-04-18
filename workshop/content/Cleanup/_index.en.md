+++
title = "Cleanup"
chapter = true
weight = 40
+++

In this workshop, you 
* Configured an nVidia device 
* Installed Greengrass v2
* Learned to create and validate components
* Created and Deployed Components to run Inference
* Validated and explored the various log files and monitoring for Greengrass

If you wish to clean it up, you will need to delete some or all of these resources:
* Remove components from a deployment
* Cancel the deployment
* Delete components
* Remove objects from S3
* IoT Greengrass Groups, Things, and Thing Groups
* Certificates, Policies, and Roles

### Removing components from a deployment

From the [AWS IoT Console](https://us-west-2.console.aws.amazon.com/iot/home) click, **Greengrass**, **Deployments**, and Check the desired deployment (e.g. `cv-lab`). Click, **Revise** and unselect the components you no longer wish to have as part of the deployment.  This will leave the deployment in tact, but will delete the package from the the targeted devices. You can also **Cancel** the deployment, which will stop any future processing. Deployments **cannot** be deleted. 

### Deleting Components

Components are independent of the Deployments. As such, they can be retained for future use or deleted. To delete the, click **Greengrass**, **Components**, then Click on the link for the un-desired component (e.g. `aws.greengrass.JetsonDLRImageClassification`). Click, **Delete version**.  Note that each Component version can be deleted separately.

Deleting the component does NOT remove the artifacts from S3 storage.

### Remove objects from S3

Navigate to the S3 console and use the interface to delete objects, prefixes, or the entire bucket.