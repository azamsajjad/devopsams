+++
title = 'AWS IAM Permissions For SSM'
date = 2024-10-12T09:58:37Z
draft = false
description = "Article explaining Basic Concept of Least Previlige."
featured = false
tags = [
    "aws",
    "iam"
]
categories = ["aws"]
thumbnail = "images/aws.png"
+++
# Minimal IAM Permissions for AWS SSM
> **USE CASE**:
> Principle of least privilege when assigning permissions for SSM as EC2 Instance Profile.

<!--more-->
When using AWS Systems Manager (SSM), it's important to only provide the minimal permissions necessary for the service to function properly. Assigning overly broad or full access can pose security risks, and it's often unnecessary for standard use cases. Below is a minimal IAM policy that ensures SSM can work while limiting permissions to the essential actions required for basic functionality.

### Policy

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "ssmmessages:CreateDataChannel",
                "ssmmessages:OpenDataChannel",
                "ssmmessages:CreateControlChannel",
                "ssmmessages:OpenControlChannel",
                "ssm:UpdateInstanceInformation",
                "ec2:Describe*"
            ],
            "Effect": "Allow",
            "Resource": "*"
        }
    ]
}
```
## Explanation

- **`ssmmessages:CreateDataChannel` & `ssmmessages:OpenDataChannel`**: These actions allow SSM to create and open the data channel necessary for communication between the instance and SSM.

- **`ssmmessages:CreateControlChannel` & `ssmmessages:OpenControlChannel`**: These actions permit SSM to manage the control channel for sending commands and receiving responses.

- **`ssm:UpdateInstanceInformation`**: This action is required for updating the instance’s status and ensuring SSM maintains up-to-date information about managed instances.

- **`ec2:Describe*`**: The Describe permissions are required to retrieve information about the instance’s environment, such as instance metadata and status, which helps SSM manage the instance correctly.

### Key Takeaway

This policy provides the necessary permissions for SSM to work properly without granting full access to SSM, EC2, or other resources. By applying the principle of least privilege, we ensure that the instance can communicate with SSM efficiently and securely, without exposing excessive permissions.
