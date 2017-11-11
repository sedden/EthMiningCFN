# Automated build of the ethminer AMI

The purpose of this build description is to automate the creation of the ethminer Amazon Machine Image (AMI)
as well as its distribution across multiple regions.

## Prerequisites

**Step 1:** Install [Packer](https://www.packer.io/downloads.html)

**Step 2:** Create an IAM user for Packer with the following inline policy attached ([source](https://www.packer.io/docs/builders/amazon.html#specifying-amazon-credentials))

    {
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Action" : [
            "ec2:AttachVolume",
            "ec2:AuthorizeSecurityGroupIngress",
            "ec2:CopyImage",
            "ec2:CreateImage",
            "ec2:CreateKeypair",
            "ec2:CreateSecurityGroup",
            "ec2:CreateSnapshot",
            "ec2:CreateTags",
            "ec2:CreateVolume",
            "ec2:DeleteKeypair",
            "ec2:DeleteSecurityGroup",
            "ec2:DeleteSnapshot",
            "ec2:DeleteVolume",
            "ec2:DeregisterImage",
            "ec2:DescribeImageAttribute",
            "ec2:DescribeImages",
            "ec2:DescribeInstances",
            "ec2:DescribeRegions",
            "ec2:DescribeSecurityGroups",
            "ec2:DescribeSnapshots",
            "ec2:DescribeSubnets",
            "ec2:DescribeTags",
            "ec2:DescribeVolumes",
            "ec2:DetachVolume",
            "ec2:GetPasswordData",
            "ec2:ModifyImageAttribute",
            "ec2:ModifyInstanceAttribute",
            "ec2:ModifySnapshotAttribute",
            "ec2:RegisterImage",
            "ec2:RunInstances",
            "ec2:StopInstances",
            "ec2:TerminateInstances"
        ],
        "Resource" : "*"
    }]
    }

## Building the image

**Step 1:** Set the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables according to the Packer IAM users credentials:

    export AWS_ACCESS_KEY_ID=PACKER_IAM_USER_ACCESS_KEY_ID
    export AWS_SECRET_ACCESS_KEY=PACKER_IAM_USER_SECRET_ACCESS_KEY

**Step 2:** Validate the build description:

    packer validate eth-mining-ami.json

**Step 3:** Start building the AMI:

    packer build eth-mining-ami.json

**Step 4:** Verify the build process finishes successfully:

    ...
        amazon-ebs: Install the project...
        amazon-ebs: -- Install configuration: "Release"
        amazon-ebs: -- Installing: /usr/local/bin/ethminer
        amazon-ebs: Done
    ==> amazon-ebs: Stopping the source instance...
        amazon-ebs: Stopping instance, attempt 1
    ==> amazon-ebs: Waiting for the instance to stop...
    ==> amazon-ebs: Creating the AMI: ethminer-1510348061
        amazon-ebs: AMI: ami-0b5ed864
    ==> amazon-ebs: Waiting for AMI to become ready...
    ==> amazon-ebs: Copying AMI (ami-0b5ed864) to other regions...
        amazon-ebs: Copying to: us-east-1
        amazon-ebs: Copying to: us-east-2
        amazon-ebs: Copying to: us-west-1
        amazon-ebs: Copying to: us-west-2
        amazon-ebs: Copying to: ap-northeast-1
        amazon-ebs: Copying to: eu-west-1
        amazon-ebs: Copying to: eu-west-2
        amazon-ebs: Waiting for all copies to complete...
    ==> amazon-ebs: Terminating the source AWS instance...
    ==> amazon-ebs: Cleaning up any extra volumes...
    ==> amazon-ebs: No volumes to clean up, skipping
    ==> amazon-ebs: Deleting temporary security group...
    ==> amazon-ebs: Deleting temporary keypair...
    Build 'amazon-ebs' finished.

    ==> Builds finished. The artifacts of successful builds are:
    --> amazon-ebs: AMIs were created:
    ap-northeast-1: ami-16fc4970
    eu-central-1: ami-0b5ed864
    eu-west-1: ami-1442ed6d
    eu-west-2: ami-7f968a1b
    us-east-1: ami-8cd56cf6
    us-east-2: ami-ae89a6cb
    us-west-1: ami-a8cbf5c8
    us-west-2: ami-9674a3ee

**Step 5:** Update the `eth-mining-cfn.template` with the new AMI image IDs (see `AWSRegion2AMI` mapping at line 32) and
upload the CloudFormation template for the desired regions.