+++
title = 'AWS Cognito Concepts'
date = 2024-10-12T10:58:37Z
draft = false
description = "Article explaining Basic Concept of Amazon Cognito."
featured = false
tags = [
    "aws",
    "iam"
]
categories = ["aws"]
thumbnail = "images/building.png"
+++

# Amazon Cognito Overview

## What is Amazon Cognito?
Amazon Cognito is a fully managed service that simplifies authentication for mobile and web applications, providing secure authorization for users. It enables users to access AWS resources after successfully authenticating with a web-based identity provider (IdP) such as Facebook, Amazon, or Google.

### Benefits of Using Amazon Cognito
- **Identity Federation:** Allows users to log in through various third-party IdPs, enhancing user experience and flexibility.
- **User Management:** Manages user registration, authentication, and account recovery.
- **Temporary Credentials:** Issues temporary AWS credentials for accessing AWS resources without hardcoding credentials in applications.
- **Multi-Device Support:** Synchronizes user data across multiple devices seamlessly.

### Types of Identity Providers
- **SAML:** Security Assertion Markup Language (SAML) for Single Sign-On (SSO).
- **OpenID Connect (OIDC):** OAuth-based authentication for mobile and web applications.

### Authentication Process
1. Users log in to a web identity provider (e.g., Facebook).
2. Upon successful sign-in, the IdP returns a JSON Web Token (JWT).
3. Cognito exchanges the JWT for temporary AWS security credentials (via Identity Pools).
4. The temporary credentials are mapped to an IAM role that defines what resources users can access.

### Cognito Push Synchronization
Cognito utilizes AWS Simple Notification Service (SNS) to push updates and synchronize user data across multiple devices, ensuring a consistent user experience.

---

### Advanced App Client Settings
1. **Identity Providers:**
   - Select identity providers available to this app client.
   - **Cognito User Pool:** Users can sign in using their email, phone number, or username.

2. **OAuth 2.0 Grant Types:**
   - Choose at least one OAuth grant type to configure token delivery.
   - **Authorization Code Grant:** Provides an authorization code as the response.
   - **Implicit Grant:** The client receives the access token (and optionally, ID token) directly.

3. **OpenID Connect Scopes:**
   - Specify the attributes this app client can retrieve for access tokens.
   - **Scopes:**
     - **OpenID:** Required for basic authentication.
     - **Phone:** Requires OpenID.
     - **Email:** Requires OpenID.
     - **aws.cognito.signin.user.admin:** Allows admin access to sign in users.
     - **Profile:** Requires OpenID.

### Customization
- Navigate to `Amazon Cognito > User Pools > MyUserPool > Edit Hosted UI customization` to add your logo and customize the login page.
- Access `Amazon Cognito > User Pools > MyUserPool > App client: myappclient` to preview your login page.

---