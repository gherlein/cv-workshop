+++
title = "Deploy and Monitor"
chapter = false
weight = 22
+++

In this section, you will deploy the components created in the previous section. As deployment can take some time (15 minutes is not unusual), this guide will also describe some basic monitoring of the Greengrass core and the deployment process.

### Step 1: Deploy the new components

1.  Navigate to the [AWS IoT Management Console](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/greengrass/v2/cores) and Click **Greengrass** and then **Deployments** from the left side panel. Check the box next to `cv-lab` and click, **Revise**.

2. Click, **Next** and check the boxes for the components you just created.

{{< img "sel-comps.png" "Select Components" >}}

3.  Click, **Next** two more times to advance to the Review screen, verify the information is correct and click, **Deploy**.

**Deployment can take 15 minutes or more**

### Step 2: Monitor the progress of the deployment

1. Connect to the target device

```bash
ssh <user>@<dev>
```
You may find it helpful to open multiple shells and/or use  `tmux` or other tool.

2. Become su

```bash
sudo su -
```
Enter your password when prompted.

3. Navigate to the Greengrass root

```bash
cd /greengrass/v2
```
The specific directory on your device may be different i]f you installed greengrass in some other location.

4. Observe and tail some status directories and files

```bash
tail -f logs/greengrass.log
```
Will show the status of the Greengrass core as it processes the deployment.

```bash
tree deployments/
```
Shows the status of the most recent deployment and any on-going deployments
```
deployments/
├── arn.aws.greengrass.us-west-2.119690479916.configuration.thing+jetson.26
│   ├── deployment_metadata.json
│   └── rollback_snapshot.tlog
└── previous-success -> /greengrass/v2/deployments/arn.aws.greengrass.us-west-2.11-----16.configuration.thing+jetson.26
```

Note that in this example, the deployment is version 26 and it was successfully deployed.

5. Monitor MQTT topics on the [AWS IoT Console](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/test) Test client by clicking, **Test** and subscribing to the topic `demo/topic`. The inference results are also logged in the file `/greengrass/v2/work/greengrass_ml/inference_log/resnet18_v1-jetson_nano.log`.
