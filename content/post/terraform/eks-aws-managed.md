+++
title = 'EKS AWS Managed NodeGroups'
date = 2024-10-12T09:58:37Z
draft = false
description = "Sample article showcasing basic Markdown syntax and formatting for HTML elements."
featured = true
tags = [
    "terraform",
    "featured"
]
categories = ["terraform"]
thumbnail = "images/building.png"
+++

## Creating a EKS Cluster with AWS-Managed Node Groups

Creating an Amazon EKS (Elastic Kubernetes Service) cluster can be streamlined using Terraform. Below is a comprehensive example of how to set up an EKS cluster along with necessary configurations for logging, IAM roles, and access entries.

### Terraform Code

Here's how to create an EKS cluster with essential configurations:

```hcl
data "tls_certificate" "demo" {
  count = var.ENVIRONMENT != "PROD" ? 1 : 0
  url = aws_eks_cluster.eks.identity.0.oidc.0.issuer
}

data "aws_caller_identity" "current" {}
data "aws_region" "current" {}

resource "aws_iam_openid_connect_provider" "demo" {
  count = var.ENVIRONMENT != "PROD" ? 1 : 0
  client_id_list = ["sts.amazonaws.com"]
  thumbprint_list = [data.tls_certificate.demo[count.index].certificates[0].sha1_fingerprint]
  url = aws_eks_cluster.eks.identity.0.oidc.0.issuer
}

resource "aws_cloudwatch_log_group" "cluster-logs" {
  name = "${var.CLUSTER_NAME}-Log-Group"
  retention_in_days = var.LOGS_RETENTION_IN_DAYS

  tags = {
    Name = "${var.CLUSTER_NAME}-Log-Group"
  }
}

resource "aws_eks_cluster" "eks" {
  name     = var.CLUSTER_NAME
  version  = var.KUBERNETES_VERSION
  role_arn = var.CLUSTER_ROLE_ARN
  enabled_cluster_log_types = var.LOGGING_TYPE

  encryption_config {
    resources = ["secrets"]
    provider {
      key_arn = var.KMS_ARN
    }
  }

  access_config {
    authentication_mode = var.ACCESS_CONFIG
    bootstrap_cluster_creator_admin_permissions = false
  }

  vpc_config {
    security_group_ids = var.SECURITY_GROUP_ID_CLUSTER
    subnet_ids         = var.CLUSTER_SUBNET_IDS
    endpoint_private_access = var.ENDPOINT_PRIVATE_ACCESS
    endpoint_public_access  = var.ENDPOINT_PUBLIC_ACCESS
    public_access_cidrs     = var.PUBLIC_ACCESS_CIDRS
  }

  tags = merge(
    var.COMMON_TAGS,
    var.TAGS,
    {
      "Name" = var.CLUSTER_NAME
    },
    {
      "kubernetes.io/cluster/${var.CLUSTER_NAME}" = "owned"
    }
  )
}
```
# Creating AWS Managed Node Groups with Terraform

AWS Managed Node Groups simplify the management of worker nodes in your Amazon EKS (Elastic Kubernetes Service) cluster. This guide demonstrates how to create an AWS managed node group using Terraform.

## Terraform Code

Below is Terraform configuration for creating a managed node group named `backend-node-group`:

```hcl
resource "aws_eks_node_group" "backend_node_group" {
  depends_on = [ aws_eks_cluster.eks]

  cluster_name    = var.CLUSTER_NAME
  node_group_name = "backend-node-group"
  node_role_arn   = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:role/EKS_NodeGroup_Role"

  subnet_ids = var.CLUSTER_SUBNET_IDS  # Specify your subnet IDs
  
  capacity_type = var.CAPACITY_TYPE  # On-Demand or Spot Instances
  scaling_config {
    desired_size = var.BACKEND_DESIRED_SIZE  # Desired number of nodes
    max_size     = var.BACKEND_MAX_SIZE      # Maximum number of nodes
    min_size     = var.BACKEND_MIN_SIZE      # Minimum number of nodes
  }

  taint {
    key    = "requestor"
    value  = "backend"
    effect = "NO_EXECUTE"  # Prevents pods that do not tolerate this taint from being scheduled on the nodes
  }

    launch_template {
      name    = aws_launch_template.backend_launch_template.name
      version = aws_launch_template.backend_launch_template.latest_version
    }

  labels = {
    wazuh   = true
    backend = true
    datadog = true
  }

  instance_types = var.INSTANCE_TYPES  # Specify your desired instance type(s)
}
```

## Access Entries and Policies
Access to the EKS cluster can be managed through IAM roles and Kubernetes RBAC. The following resources define access entries for Admin role:

```hcl
resource "aws_eks_access_entry" "admin" {
  depends_on = [ aws_eks_cluster.eks]
  cluster_name      = var.CLUSTER_NAME
  principal_arn     = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:role/aws-reserved/sso.amazonaws.com/${var.ADMIN_SSO_ROLE}"
  kubernetes_groups = ["admin"]
  type              = "STANDARD"
}

# Similar blocks for devops, oidc_infra, and oidc roles...

resource "aws_eks_access_policy_association" "admin" {
  depends_on = [aws_eks_access_entry.admin]
  cluster_name  = var.CLUSTER_NAME
  policy_arn    = "arn:aws:eks::aws:cluster-access-policy/AmazonEKSClusterAdminPolicy"
  principal_arn = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:role/aws-reserved/sso.amazonaws.com/${var.ADMIN_SSO_ROLE}"

  access_scope {
    type = "cluster"
  }
}
```
## Calling the Module
Finally, I call the module with specific parameters, including the instance types, cluster name, desired capacity, and more.

```hcl
module "EKS" {

  source      = "../../modules/EKS"
  TAGS        = var.TAGS
  COMMON_TAGS = local.common_tags

  ###########################
  ##### Control Plane Logging
  LOGS_RETENTION_IN_DAYS = var.LOGS_RETENTION_IN_DAYS

  ##########################
  ############## EKS Cluster
  ENVIRONMENT                  = "DEV"
  CLUSTER_NAME                 = "dev-eks"
  KUBERNETES_VERSION           = "1.30"
  ACCESS_CONFIG                = "API_AND_CONFIG_MAP"
  CLUSTER_ROLE_ARN             = module.IAM_Role_cluster.IAM_ROLE_ARN_FOR_STATEMENT
  ENDPOINT_PUBLIC_ACCESS       = var.ENDPOINT_PUBLIC_ACCESS
  PUBLIC_ACCESS_CIDRS          = var.PUBLIC_ACCESS_CIDRS
  ENDPOINT_PRIVATE_ACCESS      = var.ENDPOINT_PRIVATE_ACCESS
  LOGGING_TYPE                 = ["api", "audit", "authenticator", "controllerManager", "scheduler"]
  KMS_ARN                      = data.terraform_remote_state.kms_iam_sg.outputs.EKS_KMS_KEY_ARN
  SECURITY_GROUP_IDS           = ["${data.terraform_remote_state.kms_iam_sg.outputs.EKS_SECURITY_GROUP_ID}"]
  CLUSTER_SUBNET_IDS           = data.terraform_remote_state.vpc.outputs.PUBLIC_SUBNET_IDS
  INSTANCE_TYPES               = ["t3a.medium","t3.medium","t2.medium"]
  CAPACITY_TYPE                = "SPOT"
  BACKEND_MAX_SIZE             = 45
  BACKEND_DESIRED_SIZE         = 35
  BACKEND_MIN_SIZE             = 25
  CODEBUILD_ROLE               = "xxxxxxxxxxxxxxxxx-AdministratorAccess"
  ADMIN_SSO_ROLE               = "AWSReservedSSO_Admin_xxxxxxxxxxxxxxxx"
  DEVOPS_SSO_ROLE              = "AWSReservedSSO_Devops_xxxxxxxxxxxxxxx"
  FULLACCESS_SSO_ROLE          = ""
  READACCESS_SSO_ROLE          = ""
  GITHUB_OIDC_ROLE             = "githubactions_oidc"
  VPC_ID                       = data.terraform_remote_state.vpc.outputs.VPC_ID
  INGRESS_ROLE_ARN             = module.IAM_Role.IAM_ROLE_ARN
}
```
