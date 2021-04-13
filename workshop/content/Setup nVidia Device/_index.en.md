+++
title = "Setup nVidia Device"
chapter = true
weight = 10
+++

In this section you will set up an nVidia Jetson Nano, TX2, or AGX device with 
* Latest Jetpack and other libraries

And then you will install and configure AWS IoT Greengrass v2 on the device.

## Prerequisites

You will need 
1. host computer (Linux, MacOS, or Windows) -- referred to as host -- with administrator or root access
2. an AWS Account with Access Key and Secret Key credentials that has the ability to create IAM roles, IoT Resources, and S3 objects
3. as AWS Console login to the same account
4. an nVidia device (Jetson Nano, TX2, or AGX)
5. internet access

Depending on the device, you may also need
1. SD Card (32GB or larger recommended) -- the device will run from this card
2. a USB SD Card reader for use in the host computer to write the device software

To access the device, network connectivity between the host and the device are recommended so as to allow for SSH from the host to the device. Alternatively, an HDMI display, mouse, and keyboard can be directly attached to the device for access. Yet another option for some devices is to configure the device upon First Boot using a USB cable.

See the apporpriate nVidia guide for your device:
* [Jetson Nano](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit#setup)

While these labs are written to primarily use the host computer, it is possible to execute all the commands directly on the target device. If you wish to do that, you may need to modify the lab guides slightly.

