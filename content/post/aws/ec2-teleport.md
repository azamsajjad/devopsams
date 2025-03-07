+++
title = 'AWS EC2 Instance Connect & Bastion Host'
date = 2025-03-08T09:58:37Z
draft = false
description = "AWS EC2 Bastion Host."
featured = false
tags = [
    "aws",
    "ec2"
]
categories = ["aws"]
thumbnail = "images/aws.png"
+++
# Manage Application Access With Zero Trust Bastion - Teleport 🚀

As organizations expand their cloud environments, secure access to infrastructure becomes a critical challenge. Traditional access methods, such as VPNs and SSH keys, introduce security risks and operational inefficiencies. This is where **Teleport Bastion**, an open-source access management platform, stands out by providing secure, auditable, and user-friendly access to servers, Kubernetes clusters, databases, and internal applications.
<!--more-->

## Key Advantages of Teleport Bastion

### 1. **Zero Trust Security Model**
Teleport follows a **zero-trust architecture**, ensuring that access is granted only to authenticated and authorized users. It integrates with identity providers (IdPs) such as **Okta, AWS IAM Identity Center, and Active Directory**, eliminating the need for static credentials.

### 2. **Role-Based and Just-in-Time Access**
With Teleport, access control is enforced using **role-based access control (RBAC)** and **attribute-based access control (ABAC)**. This allows organizations to provide granular permissions based on user roles and context, reducing the risk of unauthorized access. Additionally, Teleport enables **just-in-time (JIT) access requests**, ensuring that privileged access is temporary and audited.

### 3. **Passwordless and Keyless Authentication**
Teleport eliminates the need for SSH keys, passwords, or VPN credentials by leveraging modern authentication mechanisms such as **SSO (Single Sign-On), MFA (Multi-Factor Authentication), and biometrics**. This removes the risks associated with credential leaks and unauthorized key sharing.

### 4. **Session Recording and Audit Logging**
One of the biggest advantages of Teleport is its built-in **session recording** and **audit logging** capabilities. Every session, command, and access request is logged and stored securely, providing complete visibility and compliance with security policies and regulatory requirements.

### 5. **Secure Kubernetes and Database Access**
Teleport extends its secure access capabilities beyond SSH to **Kubernetes clusters and databases**. It enables developers and operations teams to connect securely to Kubernetes workloads and databases (PostgreSQL, MySQL, etc.) without exposing them to the public internet, reducing the attack surface.

### 6. **Simplified Access for DevOps and SRE Teams**
For DevOps and Site Reliability Engineering (SRE) teams, managing multiple access tools can be cumbersome. Teleport **consolidates SSH, Kubernetes, and database access into a single gateway**, reducing complexity and improving operational efficiency.

### 7. **Seamless Integration with Cloud and On-Prem Infrastructure**
Teleport is designed to work across hybrid cloud environments, supporting **AWS, Azure, Google Cloud, and on-premises data centers**. This flexibility makes it an ideal choice for organizations managing multi-cloud and hybrid infrastructure.

### 8. **Compliance and Regulatory Benefits**
For industries subject to compliance regulations such as **SOC 2, HIPAA, PCI-DSS, and GDPR**, Teleport provides an auditable access trail and fine-grained access controls that help meet compliance requirements effortlessly.

## Setting Up Teleport Bastion on an EC2 Instance in a Public Subnet

### Prerequisites
- An AWS account with IAM permissions to create EC2 instances, security groups, and IAM roles.
- An Amazon Linux 2 or Ubuntu 22.04 EC2 instance in a public subnet.
- A domain or subdomain for accessing Teleport via a browser (optional but recommended).

### Steps to Install and Configure Teleport

1. **Launch an EC2 Instance**
   - Choose an instance type such as `t3.medium` (recommended for small deployments).
   - Assign it to a public subnet with an Elastic IP (EIP) attached.
   - Open necessary ports in the security group (e.g., `22` for SSH, `443` for web UI, and `3022-3025` for Teleport services).

2. **Install Teleport**
   ```bash
   curl -O https://get.gravitational.com/teleport-17.2.0-linux-amd64-bin.tar.gz
   tar -xzf teleport-17.2.0-linux-amd64-bin.tar.gz
   sudo mv teleport tctl tsh /usr/local/bin/
   teleport version
   ```

3. **Configure Teleport**
   - Create a configuration file:
   ```bash
   sudo mkdir -p /etc/teleport
   sudo tee /etc/teleport/teleport.yaml > /dev/null <<EOL
   version: v2
   teleport:
     nodename: "bastion-server"
     log:
       output: stderr
       severity: INFO
   auth_service:
     enabled: yes
     cluster_name: "example.com"
   ssh_service:
     enabled: yes
   proxy_service:
     enabled: yes
     public_addr: "your-public-ip:443"
     acme:
       enabled: yes
   EOL
   ```

4. **Start Teleport**
   ```bash
   sudo teleport start --config=/etc/teleport/teleport.yaml
   ```

5. **Access Teleport Web UI**
   - Navigate to `https://your-public-ip` in a browser.
   - Follow the on-screen instructions to complete the setup.

6. **Integrate with IAM Identity Center (Optional)**
   - Configure Teleport to use SSO authentication for passwordless access.
---
## Setting Up Teleport Bastion in a Private Subnet with a Private Hosted Zone and Let's Encrypt Certificates

### Prerequisites
- A private VPC with a **private subnet**.
- An EC2 instance launched in the private subnet.
- AWS Route 53 **private hosted zone** set up for internal domain resolution.
- AWS Certificate Manager (ACM) or Let's Encrypt for SSL/TLS.

### Steps to Deploy

1. **Create a Private Hosted Zone in Route 53**
   - Navigate to **Route 53** → **Hosted Zones** → **Create Hosted Zone**.
   - Set the domain name (e.g., `internal.example.com`) and choose **Private Hosted Zone**.
   - Associate it with the VPC where the EC2 instance is deployed.

2. **Set Up Teleport Configuration**
   - Modify `teleport.yaml`:
   ```yaml
   proxy_service:
     enabled: yes
     public_addr: "bastion.internal.example.com:443"
     acme:
       enabled: yes
       email: "admin@example.com"
   ```

3. **Generate Let's Encrypt Certificates**
   - Install `certbot`:
   ```bash
   sudo apt install certbot
   ```
   - Request a certificate:
   ```bash
   sudo certbot certonly --dns-route53 -d bastion.internal.example.com
   ```
   - Update Teleport to use the generated certificates.

4. **Access Teleport via Internal DNS**
   - Use `https://bastion.internal.example.com` within the VPC.

## Conclusion
Teleport Bastion is a modern, secure, and scalable solution for infrastructure access management. By implementing Teleport, organizations can significantly enhance security, simplify access controls, and ensure compliance while improving developer productivity. With features like **zero-trust authentication, session recording, and just-in-time access**, Teleport is an essential tool for securing infrastructure access in cloud-native environments.

Organizations looking to strengthen their security posture should consider **adopting Teleport** to eliminate risks associated with traditional access methods while embracing a **more secure and efficient access management strategy**.

