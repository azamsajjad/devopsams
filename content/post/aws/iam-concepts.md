+++
title = 'IAM Concepts'
date = 2024-10-12T09:58:37Z
draft = false
description = "Sample article showcasing basic Markdown syntax and formatting for HTML elements."
featured = false
tags = [
    "aws",
    "iam"
]
categories = ["aws"]
thumbnail = "images/aws.png"
+++
# AWS Identity and Access Management (IAM)

IAM allows you to manage users and their level of access to the AWS console. It provides centralized control over your AWS account and offers various features that help secure and manage access to AWS resources.
<!--more-->
## Key Features of IAM

1. **Centralized Control**
   - IAM gives you centralized control over your AWS account.
   - Manage who can access what resources within your account.

2. **Shared Access**
   - IAM allows you to provide shared access to your AWS account, enabling different users to access resources.

3. **Granular Permissions**
   - IAM provides granular permissions, enabling you to control which resources each user or group can access.
   - You can assign different levels of access to different users within your organization.

4. **Identity Federation**
   - IAM supports identity federation, allowing users to log in with credentials from external systems like Active Directory, Facebook, or LinkedIn.

5. **Multi-Factor Authentication (MFA)**
   - IAM allows for multi-factor authentication. Users are granted access only after successfully completing multiple authentication mechanisms.
   - Example: A user provides a username/password and a software token (like Google Authenticator) as a second factor.

6. **Temporary Access**
   - IAM enables you to provide temporary access to users, devices, or services as needed.
   - Example: A web or mobile application can access AWS resources (e.g., S3 or DynamoDB) temporarily using IAM roles and temporary credentials.

7. **Password Rotation Policy**
   - IAM allows you to enforce password policies, including password rotation, ensuring better security practices.

8. **Service Integration**
   - IAM integrates with many AWS services, offering seamless access control across the AWS ecosystem.

9. **PCI DSS Compliance**
   - IAM supports PCI DSS compliance, which is essential for applications dealing with the payment card industry.

## Core IAM Concepts

1. **Users**
   - These are end-users or individuals who log in to the AWS console or interact with AWS via API commands.
   - New users have no permissions by default and must be granted access.

2. **Groups**
   - Groups are collections of users that share a common set of permissions.
   - Example: A marketing team group might have permission to access certain S3 buckets containing logos or images.
   - Groups simplify permissions management by applying the same policies to multiple users.

3. **Roles**
   - Roles define a set of permissions that can be assumed by users or AWS resources (like EC2 instances).
   - Example: An EC2 instance needing access to an S3 bucket can assume a role with the necessary permissions.
   - Roles are often used to allow AWS services to interact with each other securely.

4. **Policies**
   - Policies are documents that define one or more permissions.
   - A policy can be attached to users, groups, or roles.
   - When attached, the users, groups, or roles inherit the permissions specified in the policy.

## Permissions Management

1. **Explicit Deny**
   - Explicit Deny overrides any Allow permissions.
   - Example: If a user has an explicit Deny for accessing an S3 bucket, it will override an Allow permission given to the group they belong to.

2. **Implicit Deny**
   - All permissions are implicitly denied until they are explicitly granted.

3. **Principle of Least Privilege**
   - Always follow the principle of least privilege by assigning only the permissions required for a user to perform their job.

4. **IAM Policy Simulator**
   - You can validate your policies using the [IAM Policy Simulator](https://policysim.aws.amazon.com) to ensure they work as expected.

## Example IAM Scenario

- A marketing team needs read and write access to a specific S3 bucket containing logos and images. You can create a group called `Marketing` and assign the necessary permissions to that group. All users added to the `Marketing` group will automatically inherit the required permissions to access the S3 bucket.
  
- Similarly, for a team of database administrators who need to create DynamoDB tables and run queries, you can create a group called `DBAdmins` with permissions for DynamoDB and add the relevant users.

# Three Types of AWS Policies

## 1. **AWS Managed Policies**
   - Predefined by AWS.
   - Example policies:
     - `AmazonEC2ReadOnlyAccess`
     - `AmazonDynamoDBFullAccess`
     - `AWSCodeCommitPowerUser`
   - **Key points**:
     - You **cannot** change the permissions defined in an AWS Managed Policy.
     - Reusable across different entities (users, groups, or roles).
   
## 2. **Customer Managed Policies**
   - You can create and customize your own policies.
   - Example: You can copy an AWS Managed Policy and modify it according to your needs.
   - **Key points**:
     - Reusable across different entities.
     - Provides greater flexibility for tailored permissions.

## 3. **Inline Policy**
   - A one-to-one relationship between an entity (user, group, or role) and the policy.
   - When you delete the entity, the policy gets deleted as well.
   - AWS recommends using Managed Policies over Inline Policies.
   - **Key points**:
     - Not reusable.
     - Option to add inline policies in the user, group, or role sections.

---

# Security Token Service (STS) - AssumeRoleWithWebIdentity

## Overview
The **AssumeRoleWithWebIdentity** API, provided by the Security Token Service (STS), returns temporary security credentials for users authenticated via a web identity provider (such as Amazon, Facebook, or Google). This API is used to allow users, authenticated by external identity providers, to access AWS resources temporarily.

For mobile applications, AWS recommends using **Cognito**, which internally makes calls to the `AssumeRoleWithWebIdentity` API on your behalf. However, for web applications not using Cognito, you can directly use this API.

### JWT Flow
1. **User logs in** to Facebook (or another web identity provider).
2. Facebook **returns a JWT token** to the user.
3. The user sends the JWT token to **STS** via the `AssumeRoleWithWebIdentity` API.
4. STS responds with **temporary AWS credentials**.
5. The user can now access AWS resources (e.g., S3, DynamoDB) using those temporary credentials.

### Key Points:
- The **ARN** and **AssumedRoleID** are used to programmatically reference the temporary credentials.
- This process **does not involve IAM users or roles**; instead, it deals with temporary security credentials.

---

# Cross-Account Roles

Cross-account access allows you to **delegate access** to AWS resources in one account to users in another AWS account. This is useful for sharing resources across different accounts, such as production and development accounts, without needing to create IAM users in every account.

### How it Works:
1. Create an **IAM Role** in the account that owns the resource (e.g., production).
2. **Grant permission** to users in a different AWS account (e.g., development) to assume that role.
3. Users from the development account can **switch roles** in the AWS Management Console to access resources in the production account.

### Example Use Case:
Imagine you have:
- A **development account** with your development team.
- A **production account** with an S3 bucket storing historical production data.
  
You want the developers in the development account to have read access to the S3 bucket in the production account for testing purposes.

### Steps:
1. Create a **role** in the production account that grants access to the S3 bucket.
2. Create a **policy** in the development account that allows users to assume the role in the production account.
3. Developers can now switch roles to access the S3 bucket in the production account without needing a separate login.

### Visual Representation:

| **Production Account** | **Development Account** |
| ---------------------- | ----------------------- |
| S3 Bucket               | Developer IAM Group     |
| IAM Role               | IAM Policy              |

The role in the production account allows the developers in the development account to access the S3 bucket.

---
# AWS Certification Questions and Answers

## Question 1
You are working on a mobile phone app for an online retailer that stores customer data in DynamoDB. You would like to allow new users to sign-up using their Facebook credentials. What is the recommended approach?

- **A.** After the user has successfully logged in to Facebook and received an authentication token, Cognito should be used to exchange the token for temporary access to DynamoDB.
- **B.** Embed encrypted AWS credentials into the application code, so that the application can access DynamoDB on the user's behalf.
- **C.** Write your own custom code which allows the user to log in via Facebook and receive an authentication token, then calls the AssumeRoleWithWebIdentity API and exchanges the authentication tokens for temporary access to DynamoDB.
- **D.** After the user has authenticated with Facebook, allow them to download encrypted AWS credentials to their device so that the mobile app can access DynamoDB.

**Correct Answer:** A. After the user has successfully logged in to Facebook and received an authentication token, Cognito should be used to exchange the token for temporary access to DynamoDB.

---

## Question 2
The most cost-effective solutions to allow existing Active Directory users access to AWS without having to recreate AWS IAM user accounts are:

- **A.** SAML 2.0
- **B.** Amazon Directory Services

**Correct Answer:** Both A and B. SAML 2.0 enables web-based, single sign-on (SSO) authentication, while Amazon Directory Services allows you to connect your AWS resources to an existing Microsoft AD.

---

## Question 3
Your company uses an Identity Provider (IdP) for Single-sign on (SSO) and has tasked their solutions architect with connecting their AWS Account to the IdP so their users can leverage their corporate identity to access the environment. What actions should the solutions architect take to meet these requirements? (Select TWO)

- **A.** Create an AWS IAM Role with a trust relationship with the IdP.
- **B.** Create an AWS IAM User Group, associate the User Group with the IdP and add users to the User Group.
- **C.** Create an AWS IAM Identity Provider by uploading the SAML metadata document from your IdP.
- **D.** Create an AWS IAM User with AWS Management Console Access, attach a policy with a trust relationship with the IdP.
- **E.** Create an AWS IAM Identity Provider by uploading the JSON metadata document from your IdP.

**Correct Answer:** A and C. Create an AWS IAM Role with a trust relationship with the IdP and create an AWS IAM Identity Provider by uploading the SAML metadata document.

---

## Question 4
You are developing a meme-sharing application that runs business login on a fleet of EC2 instances behind an Application Load Balancer. Your website is served by a CloudFront distribution. The security architect has asked you to decouple the user authentication process from the application servers, enable users to easily sign up and sign in to the website and use proper Identity and Access Management controls to allow access to a number of S3 buckets where they can upload images. Which of the following solutions would you recommend to address this use-case?

- **A.** Use Cognito for sign-up, and use resource policies to manage user permissions.
- **B.** Use MFA for sign-up and sign-in, and use IAM to manage user permissions.
- **C.** Use IAM for sign-up and use Cognito to manage user permissions.
- **D.** Use Cognito for sign-up and use IAM to manage user permissions.

**Correct Answer:** D. Use Cognito for sign-up and use IAM to manage user permissions.

---

## Question 5
Your application uses the STS API call AssumeRoleWithWebIdentity to enable access for users who have authenticated using a Web ID provider. Which of the following best describes what a successful call to AssumeRoleWithWebIdentity returns?

- **A.** AssumeRoleWithWebIdentity returns a set of temporary credentials (access key ID, secret access key, and security token) that give temporary access to AWS services.
- **B.** AssumeRoleWithWebIdentity returns an access key ID, a secret access key, and a security token as temporary security credentials. These credentials can be used by applications to sign calls for AWS service API operations.
- **C.** AssumeRoleWithWebIdentity returns an ARN of the IAM role that the user is allowed to assume temporarily.
- **D.** AssumeRoleWithWebIdentity returns an assumed role ID that the user is allowed to assume temporarily.

**Correct Answer:** B. AssumeRoleWithWebIdentity returns an access key ID, a secret access key, and a security token as temporary security credentials.

---

## Question 6
A company selling smart security cameras uses an S3 bucket behind a CloudFront web distribution to store its static content, which it shares with customers worldwide. The company has recently released a new firmware update intended only for its premium customers, and unauthorized access should be denied with a user authentication process that has minimal latency. How can a developer refactor the current setup to achieve this requirement with the MOST efficient solution?

- **A.** Restrict access to the S3 bucket only to premium customers using an Origin Access Control (OAC).
- **B.** Use Signed URLs and Signed Cookies in CloudFront to distribute the firmware update file.
- **C.** Use Lambda@Edge and Amazon Cognito to authenticate and authorize premium customers to download the firmware update.
- **D.** Use the AWS Serverless Application Model (AWS SAM) and Amazon Cognito to authenticate the premium customers.

**Correct Answer:** C. Use Lambda@Edge and Amazon Cognito to authenticate and authorize premium customers to download the firmware update.

---

## Question 7
A company has an AWS Amplify application, relying on Amazon Cognito for user authentication. Multi-factor authentication (MFA) is disabled for their User Pool. There has been a recent data breach in a popular website. The company is worried that attackers might exploit compromised email addresses and passwords to sign into their applications. For this reason, they want to enforce MFA only on users with suspicious login attempts. How can the company satisfy these requirements?

- **A.** Enable Adaptive Authentication for the User Pool.
- **B.** Create a subscription filter Lambda function that monitors for the CompromisedCredentialRisk metric from Advanced Security Metrics in CloudWatch Logs and triggers MFA when detected.
- **C.** Enable the Time-based one-time password (TOTP) software token MFA for the User Pool.
- **D.** Recreate the User Pool and enable SMS text message MFA.

**Correct Answer:** A. Enable Adaptive Authentication for the User Pool.

---

## Question 8
A developer wants to use multi-factor authentication (MFA) to protect programmatic calls to specific AWS API operations like Amazon EC2 StopInstances. He needs to call an API where he can submit the MFA code that is associated with his MFA device. Using the temporary security credentials that are returned from the call, he can then make programmatic calls to API operations that require MFA authentication. Which API should the developer use to properly implement this security feature?

- **A.** AssumeRoleWithWebIdentity
- **B.** GetSessionToken
- **C.** GetFederationToken
- **D.** AssumeRoleWithSAML

**Correct Answer:** B. GetSessionToken.

---

## Question 9
A company has a static website running in an Auto Scaling group of EC2 instances which they want to convert as a dynamic e-commerce web portal. One of the requirements is to use HTTPS to improve the security of their portal and also improve their search ranking as a reputable and secure site. A developer recently requested an SSL/TLS certificate from a third-party certificate authority (CA) which is ready to be imported to AWS. Which of the following services can the developer use to safely import the SSL/TLS certificate? (Select TWO.)

- **A.** CloudFront
- **B.** A private S3 bucket with versioning enabled
- **C.** IAM certificate store
- **D.** Amazon Cognito
- **E.** AWS Certificate Manager

**Correct Answer:** C and E. IAM certificate store and AWS Certificate Manager.

---

## Question 10
You are developing an online game where the app preferences and game state of the player must be synchronized across devices. It should also allow multiple users to synchronize and collaborate shared data in real time. Which of the following is the MOST appropriate solution that you should implement in this scenario?

- **A.** Integrate AWS Amplify to your mobile app.
- **B.** Integrate Amazon Pinpoint to your mobile app.
- **C.** Integrate AWS AppSync to your mobile app.
- **D.** Integrate Amazon Cognito Sync to your mobile app.

**Correct Answer:** C. Integrate AWS AppSync to your mobile app.

---

## Question 11
A developer is building a ReactJS application that will be hosted on Amazon S3. Amazon Cognito handles the registration and signing of users using the AWS Software Development Kit (SDK) for JavaScript. The JSON Web Token (JWT) received upon authentication will be stored on the browsers local storage. After signing in, the application will use the JWT as an authorizer to access an API Gateway endpoint. What are the steps needed to implement the scenario above? (Select THREE.)

- **A.** Create an Amazon Cognito Identity Pool.
- **B.** Create an Amazon Cognito User Pool.
- **C.** On the API Gateway Console, create an authorizer using the Cognito User Pool ID.
- **D.** Set the name of the header that will be used from the request to the Cognito User Pool as a token source for authorization.

---

## Question 12
A team of developers needs permission to launch EC2 instances with an instance role that will allow them to update items in a DynamoDB table. Each developer has access to IAM users that belong in the same IAM group. Which of the following steps must be done to implement the solution?

- **A.** Create an IAM role with an IAM policy that will allow access to the DynamoDB table. Add the EC2 service to the trust policy of the role. Create a custom policy with iam:GetRolePolicy and iam:PutRolePolicy permissions. Attach the policy to the IAM group.
- **B.** Create an IAM role with an IAM policy that will allow access to the DynamoDB table. Add the EC2 service to the trust policy of the role. Create a custom policy with the iam:PassRole permission. Attach the policy to the IAM group.
- **C.** Create an IAM role with an IAM policy that will allow access to the EC2 instances. Add the DynamoDB service to the trust policy of the role. Create a custom policy with the iam:GetRole permission. Attach the policy to the IAM group.
- **D.** Create an IAM role with an IAM policy that will allow access to the EC2 instances. Add the DynamoDB service to the trust policy of the role. Create a custom policy with the iam:PassRole permission. Attach the policy to the IAM group.

**Correct Answer:** B. Create an IAM role with an IAM policy that will allow access to the DynamoDB table. Add the EC2 service to the trust policy of the role. Create a custom policy with the iam:PassRole permission. Attach the policy to the IAM group.

---

## Question 13
An EC2 instance has an IAM role that explicitly denies all S3 API Write operations. Moreover, the instance has access key credentials configured to gain full access to S3 operations. Which statement is correct for this scenario?

- **A.** The instance cannot upload objects to S3 buckets.
- **B.** The instance can list all S3 buckets but will not be able to delete them.
- **C.** The instance can perform all S3 operations on any S3 bucket.
- **D.** The instance can perform all S3 operations except for write operations on any S3 bucket.

**Correct Answer:** A. The instance cannot upload objects to S3 buckets.

---


---
## Resources:
- [AWS STS AssumeRoleWithWebIdentity API Documentation](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRoleWithWebIdentity.html)
- [How to Enable Cross-Account Access to the AWS Management Console](https://aws.amazon.com/blogs/security/how-to-enable-cross-account-access-to-the-aws-management-console/)
