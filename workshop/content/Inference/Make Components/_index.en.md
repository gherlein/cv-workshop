+++
title = "Create the Inference Components"
weight = 21
+++

Inference on the device is implemented with THREE components:
* `variant.Jetson.DLR` provides the Deep Learning Runtime (DLR) libraries needed to run the models on the target device
* `variant.Jetson.ImageClassification.ModelStore` delivers the collection (or zoo) of Image Classifications models that will be used
* `aws.greengrass.JetsonDLRImageClassification` is the actual 'app' that runs the inference model on the device

The recipes and artifacts have been pre-created and stored in the Github repository:

https://github.com/scottrfrancis/aws-iot-greengrass-v2-deploy-nvidia-deepstream

### Step 1: Clone the repository

1. On your **development host**, clone the repository

```bash
cd ~

git clone git@github.com:scottrfrancis/aws-iot-greengrass-v2-deploy-nvidia-deepstream.git
```

2. Observe you have the artifacts and recipe files in an arrangement like this:

```
.
├── artifacts
│   ├── aws.greengrass.JetsonDLRImageClassification
│   │   ├── 1.0.0
│   │   │   ├── inference.py
│   │   │   ├── init.sh
│   │   │   ├── IPCUtils.py
│   │   │   ├── jetson_inference.py
│   │   │   ├── labels.py
│   │   │   └── samples_images
│   │   │       ├── dog.jpg
│   │   │       └── dog.npy
│   │   ├── inference.py
│   │   ├── init.sh
│   │   ├── IPCUtils.py
│   │   ├── jetson_inference.py
│   │   ├── labels.py
│   │   ├── LICENSE
│   │   ├── sample_images
│   │   │   ├── cat2.npy
│   │   │   ├── cat.jpeg
│   │   │   ├── cat.npy
│   │   │   ├── dog.jpg
│   │   │   └── dog.npy
│   │   └── samples_images
│   │       ├── cat2.npy
│   │       ├── dog.jpg
│   │       └── dog.npy
│   ├── variant.Jetson.DLR
│   │   └── 1.0.0
│   │       ├── installer.sh
│   │       └── LICENSE
│   └── variant.Jetson.ImageClassification.ModelStore
│       ├── 0.1.1
│           ├── resnet18_v1-jetson_nano
│           │   ├── compiled.meta
│           │   ├── compiled_model.json
│           │   ├── compiled.params
│           │   ├── compiled.so
│           │   ├── dlr.h
│           │   ├── libdlr.so
│           │   └── manifest
│           ├── resnet18_v1-jetson_tx2
│           │   ├── compiled.meta
│           │   ├── compiled_model.json
│           │   ├── compiled.params
│           │   ├── compiled.so
│           │   ├── dlr.h
│           │   ├── libdlr.so
│           │   └── manifest
│           └── resnet18_v1-jetson_xavier
│               ├── compiled.meta
│               ├── compiled_model.json
│               ├── compiled.params
│               ├── compiled.so
│               ├── dlr.h
│               ├── libdlr.so
│               └── manifest
└── recipes
    ├── aws.greengrass.JetsonDLRImageClassification-1.0.0.json
    ├── variant.Jetson.DLR-1.0.0.json
    └── variant.Jetson.ImageClassification.ModelStore-1.0.0.json

14 directories, 47 files
```

### Step 2: Zip and upload artifacts to S3

1. Zip the artifacts for each component

```bash

cd ~/aws-iot-greengrass-v2-deploy-nvidia-deepstream/jetson_inference/artifacts

cd variant.Jetson.DLR/1.0.0/
zip -m installer.zip *

cd ../..


```





2. Set the `bucket_name` environment variable for where you will host the artifact files. On your development **host**, extract your account number to an environment variable

```bash
export region='us-west-2'
export acct_num=$(aws sts get-caller-identity --query "Account" --output text)
export bucket_name=greengrass-component-artifacts-$acct_num-$region
```

**This bucket should have previously been created in the Greengrass installation lab**. If this bucket does **NOT** exist, you can create it with:

```bash
aws s3 mb s3://$bucket_name
```

**NB-** Alternatively, you can use an existing bucket. Set the environment variable `$bucket_name` for the remainder of this lab.

2. 