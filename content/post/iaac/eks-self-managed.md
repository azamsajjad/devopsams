+++
title = 'EKS Self Managed NodeGroups'
date = 2024-10-12T10:00:02Z
draft = false
description = "Sample article showcasing basic Markdown syntax and formatting for HTML elements."
featured = true
tags = [
    "terraform",
    "kubernetes",
    "featured"
]
categories = ["containers", "iaac"]
thumbnail = "images/kubernetes.png"
+++
### Self-Managed Node Group in EKS to Handle Spot Instance Unavailability
> **USE CASE**:
> Ensure that On-Demand Instances are automatically used if Spot Instances become unavailable.

<!--more-->
In this post, I’ll explain how I created a self-managed node group in Amazon EKS (Elastic Kubernetes Service). My goal was to ensure that **on-demand instances** are automatically used if **spot instances** become unavailable. Let’s dive into the details!
### Limitations of Managed Node Groups in EKS

When using **Amazon EKS Managed Node Groups**, AWS handles many of the heavy-lifting tasks involved in managing worker nodes for your Kubernetes cluster. While this abstraction simplifies node management, it also introduces certain **limitations**, particularly when you need more granular control over your infrastructure.

#### 1. Lack of Access to the Underlying Auto Scaling Group (ASG)

One of the key limitations of EKS managed node groups is that **you don’t have direct access to the Auto Scaling Group (ASG)** backing your node group. AWS manages this ASG for you, and while it’s convenient for standard use cases, this abstraction restricts your ability to configure certain ASG settings directly. Here's what you cannot control:

- **Custom Scaling Policies**: You can't modify scaling policies like specifying scaling triggers based on custom CloudWatch metrics or adjusting scaling cooldown times.
- **Capacity Rebalance**: In a managed node group, you cannot configure **Capacity Rebalancing**, which would automatically replace spot instances if they are interrupted by AWS.
- **On-Demand and Spot Instance Allocation**: Unlike self-managed node groups, you can’t directly configure **mixed instance policies** (i.e., adjusting how many spot or on-demand instances should be used) in the managed node group's ASG.
  
#### 2. Limited Control Over ASG Overrides

Managed node groups provide only basic options for configuring instance types and limits. You cannot fine-tune **instance overrides**, which means less flexibility when specifying multiple instance types or controlling **instance weighting** to distribute workloads more effectively across varying instance sizes.

#### 3. Lack of Direct Access to ASG Lifecycle Hooks

For more advanced lifecycle management, such as running scripts during instance launch or termination, EKS managed node groups don’t give you access to the underlying ASG lifecycle hooks. These hooks are crucial when you need to perform custom actions during these lifecycle events.

#### 4. No Control Over ASG Tags
n
In a self-managed node group, you can apply **custom tags** to your ASG, which can be useful for cost allocation, logging, or monitoring purposes. With managed node groups, this flexibility is lost, as AWS automatically manages the tags for you.

#### 5. No Support for Advanced Networking or Instance Features

With managed node groups, you can’t configure certain advanced networking options like **custom network interfaces** or **Elastic Fabric Adapter (EFA)** for high-performance computing workloads. These options are typically available when you manage the underlying EC2 instances yourself.

While **EKS Managed Node Groups** provide a high level of abstraction and ease of use, they come with trade-offs in terms of flexibility and configurability. For teams or workloads that require fine-tuned control over the underlying infrastructure, especially at the Auto Scaling Group level, **self-managed node groups** are a better option despite the additional operational overhead.

If you require custom configurations such as advanced scaling policies, mixed instance types, or direct ASG lifecycle management, you might find these limitations of managed node groups restrictive.

### Why Self-Managed Node Groups?

The primary reason for creating a self-managed node group is to have greater control over the configuration and management of EC2 instances. Although **spot instances** are cost-effective, they are not guaranteed, and AWS can reclaim them at any time. By setting up a self-managed node group, I ensure that if spot instances are unavailable, **on-demand instances** are automatically utilized, maintaining the desired capacity of my EKS cluster.

---
## Node Group Configuration

Below is the Terraform configuration that sets up the self-managed node group.

### Scripts

#### 1. Join Cluster Script
The provided cloud-init configuration script ensures that the EC2 instances launched in the EKS node group are initialized correctly with the required packages and scripts, ensuring seamless node setup and integration into the Kubernetes cluster. Below is a breakdown of what the script does
```yaml
#cloud-config
package_update: true
package_upgrade: true
packages:
    - jq

runcmd:
    - [ sh, -c, "set -o xtrace"]
    - [ sh, -c, "sleep 60"]
    - [ sh, -c, "/etc/eks/bootstrap.sh ${CLUSTER_NAME}"]
    - curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    - curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
    - sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    - [ sh, -c, "echo 'kubectl has been installed successfully.'"]
    - [ sh, -c, "aws eks update-kubeconfig --name ${CLUSTER_NAME} --region ${REGION}"]
    - [ sh, -c, "echo '[Unit]' > /etc/systemd/system/init.service"]
    - [ sh, -c, "echo 'Description=Init Service' >> /etc/systemd/system/init.service"]
    - [ sh, -c, "echo '[Service]' >> /etc/systemd/system/init.service"]
    - [ sh, -c, "echo 'ExecStart=/bin/bash /var/lib/cloud/instance/scripts/part-002' >> /etc/systemd/system/init.service"]
    - [ sh, -c, "echo '' >> /etc/systemd/system/init.service"]
    - [ sh, -c, "echo '[Install]' >> /etc/systemd/system/init.service"]
    - [ sh, -c, "echo 'WantedBy=multi-user.target' >> /etc/systemd/system/init.service"]
    - [ sh, -c, "sudo systemctl daemon-reload"]
    - [ sh, -c, "sudo systemctl enable init.service"]
    - [ sh, -c, "sudo systemctl start init.service"]

output:
    all: '| tee -a /var/log/cloud-init-output.log'
```
#### 2. User Data Script (Labelling Nodes for Affinity)

The user data script is responsible for configuring the nodes at startup. It does the following:
1. **Exports the Kubeconfig** for the Kubernetes API.
2. Fetches the **instance ID** and **node name**.
3. Labels the node based on the node group.

```bash
locals {
  labelling_script = <<-EOF
    #!/bin/bash
    export KUBECONFIG=/root/.kube/config
    REGION=us-east-1
    sleep 60
    INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
    NODE_NAME=$(aws ec2 describe-instances --instance-ids $INSTANCE_ID --region $REGION --query 'Reservations[0].Instances[0].PrivateDnsName' --output text)
    sleep 60
    /usr/local/bin/kubectl label nodes $NODE_NAME ${var.NODE_GROUP}=exists
  EOF
}
```
```hcl
data "template_file" "template_userdata" {
  template = "${file("${path.module}/userdata/cloud-init.yaml")}"
vars = {
  "CLUSTER_NAME" = "${var.CLUSTER_NAME}"
  "REGION" = data.aws_region.current.name
    }
}
data "template_cloudinit_config" "userdata" {
  gzip          = true
  base64_encode = true
  part {
    filename     = "cloud-init.yaml"
    content_type = "text/cloud-config"
    content      = data.template_file.template_userdata.rendered
  }
  part {
    content_type = "text/x-shellscript"
    content      = local.labelling_script
  }
}
resource "aws_iam_instance_profile" "this" {
  name = "${var.NODE_GROUP}"
  role = "EKS_NodeGroup_Role"
}
```
The labelling_script labels the nodes as soon as they are provisioned, making them ready for use in the cluster.

### Launch Template
The launch template defines the EC2 instances that will be created as part of the node group. It includes details like:

Instance type
EBS volume size
IAM instance profile
User data
```hcl
resource "aws_launch_template" "this" {
  name                    = "${var.NODE_GROUP}_launch_template"
  vpc_security_group_ids  = var.SECURITY_GROUP_IDS
  iam_instance_profile {
    name = aws_iam_instance_profile.this.name
  }

  block_device_mappings {
    device_name = "/dev/xvda"
    ebs {
      volume_size           = var.VOLUME_SIZE
      volume_type           = "gp2"
      delete_on_termination = true
    }
  }

  image_id   = var.IMAGE_ID
  user_data  = data.template_cloudinit_config.userdata.rendered
  monitoring {
    enabled = var.ENABLE_MONITORING
  }

  tag_specifications {
    resource_type = "instance"
    tags = merge(
      var.COMMON_TAGS,
      var.TAGS,
      {
        Name = "EKS-SELF-MANAGED-NODE-${var.NODE_GROUP}"
        "k8s.io/cluster-autoscaler/enabled"              = "true"
        "k8s.io/cluster-autoscaler/${var.CLUSTER_NAME}"  = "true"
        "kubernetes.io/cluster/${var.CLUSTER_NAME}"      = "owned"
        NodeGroup                                        = var.NODE_GROUP
      }
    )
  }
}
```

This template makes sure that all instances launched as part of this node group are configured with the correct parameters.

### Auto Scaling Group (ASG)
The Auto Scaling Group (ASG) is configured to handle both on-demand and spot instances using a mixed instance policy. This ensures that when spot instances are unavailable, the ASG automatically provisions on-demand instances to maintain the desired capacity.


```hcl
resource "aws_autoscaling_group" "this" {
  name                = "eks-${var.NODE_GROUP}-asg"
  capacity_rebalance  = true
  desired_capacity    = var.DESIRED_CAPACITY
  max_size            = var.MAX_SIZE
  min_size            = var.MIN_SIZE
  vpc_zone_identifier = var.SUBNET_IDS
  health_check_type         = "EC2"
  health_check_grace_period = 300

  mixed_instances_policy {
    instances_distribution {
      on_demand_allocation_strategy            = "prioritized"
      on_demand_base_capacity                  = var.ON_DEMAND_BASE_CAPACITY
      on_demand_percentage_above_base_capacity = var.ON_DEMAND_PERCENTAGE_ABOVE_BASE
      spot_allocation_strategy                 = "capacity-optimized"
    }

    launch_template {
      launch_template_specification {
        launch_template_id = aws_launch_template.this.id
      }
      override {
        instance_type     = var.INSTANCE_TYPE_1_W1
        weighted_capacity = "1"
      }
      override {
        instance_type     = var.INSTANCE_TYPE_2_W1
        weighted_capacity = "1"
      }
      override {
        instance_type     = var.INSTANCE_TYPE_3_W1
        weighted_capacity = "1"
      }
      override {
        instance_type     = var.INSTANCE_TYPE_1_W2
        weighted_capacity = "2"
      }
      override {
        instance_type     = var.INSTANCE_TYPE_2_W2
        weighted_capacity = "2"
      }
      override {
        instance_type     = var.INSTANCE_TYPE_3_W2
        weighted_capacity = "2"
      }
    }
  }

  tag {
    key                 = "k8s.io/cluster-autoscaler/${var.CLUSTER_NAME}"
    value               = "owned"
    propagate_at_launch = true
  }
  tag {
    key                 = "k8s.io/cluster-autoscaler/enabled"
    value               = "true"
    propagate_at_launch = true
  }
}
```
The mixed_instances_policy allows for multiple instance types to be used. It also prioritizes on-demand instances in case spot instances are not available.

## Calling the Module
Finally, I call the module with specific parameters, including the instance types, cluster name, desired capacity, and more.

```hcl
module "NODEGROUP" {
  source = "../../modules/NODEGROUP"

  CLUSTER_NAME                     = var.CLUSTER_NAME
  NODE_GROUP                       = "backend"
  SUBNET_IDS                       = data.terraform_remote_state.vpc.outputs.PUBLIC_SUBNET_IDS
  SECURITY_GROUP_IDS               = ["${data.terraform_remote_state.eks_sg.outputs.EKS_SG_ID_LT}"]
  VOLUME_SIZE                      = "20"
  IMAGE_ID                         = "ami-0147d3ab35a9exxxx"
  INSTANCE_TYPE_1_W1               = "t3a.medium"     #weight 1
  INSTANCE_TYPE_2_W1               = "t3.medium"      #weight 1
  INSTANCE_TYPE_3_W1               = "t2.medium"      #weight 1
  INSTANCE_TYPE_1_W2               = "t3.large"       #weight 2
  INSTANCE_TYPE_2_W2               = "t3a.large"      #weight 2
  INSTANCE_TYPE_3_W2               = "t4g.large"      #weight 2
  MAX_SIZE                         = 40
  DESIRED_CAPACITY                 = 30
  MIN_SIZE                         = 25
  ON_DEMAND_BASE_CAPACITY          = 0
  ON_DEMAND_PERCENTAGE_ABOVE_BASE  = 25
  ENABLE_MONITORING                = true
  COMMON_TAGS                      = local.common_tags
  TAGS                             = var.TAGS
}
```
## Variables

```hcl
variable "NODE_GROUP" {
  type = string
}

variable "SECURITY_GROUP_IDS" {
  type = list(string)
}

variable "VOLUME_SIZE" {
  type = number
}

variable "IMAGE_ID" {
  type = string
}

variable "DESIRED_CAPACITY" {
  type = number
}

variable "MAX_SIZE" {
  type = number
}

variable "MIN_SIZE" {
  type = number
}

variable "INSTANCE_TYPE_1_W1" {
  type = string  
}
variable "INSTANCE_TYPE_2_W1" {
  type = string  
}
variable "INSTANCE_TYPE_3_W1" {
  type = string  
}
variable "INSTANCE_TYPE_1_W2" {
  type = string  
}
variable "INSTANCE_TYPE_2_W2" {
  type = string  
}
variable "INSTANCE_TYPE_3_W2" {
  type = string  
}

variable "SUBNET_IDS" {
  type = list(string)
}

variable "ON_DEMAND_BASE_CAPACITY" {
  type = number
}

variable "ON_DEMAND_PERCENTAGE_ABOVE_BASE" {
  type = number
}

variable "ENABLE_MONITORING" {
  type = bool
}

variable "CLUSTER_NAME" {
  type = string
}

variable "COMMON_TAGS" {
  type = map(string)
}

variable "TAGS" {
  type = map(string)
}
```