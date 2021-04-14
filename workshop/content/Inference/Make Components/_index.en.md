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
zip -rm installer.zip *

cd ../..
cd variant.Jetson.ImageClassification.ModelStore/0.1.1/
zip -rm resnet18_v1-jetson.zip *

cd ../..
cd aws.greengrass.JetsonDLRImageClassification/1.0.0/
zip -rm image_classification.zip *
```
You should now have a zip file for each component like this
```
.
├── aws.greengrass.JetsonDLRImageClassification
│   └── 1.0.0
│       └── image_classification.zip
├── variant.Jetson.DLR
│   └── 1.0.0
│       └── installer.zip
└── variant.Jetson.ImageClassification.ModelStore
    └── 0.1.1
        └── resnet18_v1-jetson.zip
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

3. Copy the artifacts to the bucket

```bash
cd ~/aws-iot-greengrass-v2-deploy-nvidia-deepstream/jetson_inference/artifacts

aws s3 sync ./ s3://$bucket_name/artifacts/
```

Verify the layout of the artifacts in S3

{{< img "s3-artifacts.png" "Artifacts in S3" >}}

### Step 3: Create the components

1. Modify the recipe files to use the bucket and artifacts you just uploaded

```bash
cd ~/aws-iot-greengrass-v2-deploy-nvidia-deepstream/jetson_inference/recipes

sed -i "s/BUCKET_NAME/$bucket_name/g" *.json
```

2. Create the components from the modified recipe files

```bash
cd ~/aws-iot-greengrass-v2-deploy-nvidia-deepstream/jetson_inference/recipes

aws greengrassv2 create-component-version --inline-recipe fileb://variant.Jetson.DLR-1.0.0.json
aws greengrassv2 create-component-version --inline-recipe fileb://variant.Jetson.ImageClassification.ModelStore-1.0.0.json
aws greengrassv2 create-component-version --inline-recipe fileb://aws.greengrass.JetsonDLRImageClassification-1.0.0.json
```

Verify the components are shown in the [AWS IoT Management Console](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/greengrass/v2/cores) under **Greengrass** and **Components**.

{{< img "components.png" "Components" >}}

**Note**: When the components are created, a integrity checksum is created for the artifacts and added to the recipe (view the recipe on the console to see the digest). Due to this, if you change an artifact, you must update the version for the component or recreate it.
