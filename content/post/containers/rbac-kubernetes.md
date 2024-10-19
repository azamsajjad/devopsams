+++
title = 'RBAC in Kubernetes'
date = 2024-10-19T10:00:02Z
draft = false
description = "Role-Based Access Control (RBAC) in Kubernetes is a method for managing access to cluster resources."
featured = false
tags = [
    "kubernetes"
]
categories = ["containers"]
thumbnail = "images/kubernetes.png"
+++
## Role-Based Access Control in Kubernetes
Role-Based Access Control (RBAC) in Kubernetes is a method for managing access to cluster resources. It allows you to assign permissions to users, groups, or service accounts, ensuring that they have access only to the resources necessary to perform their tasks. This minimizes security risks by preventing unauthorized actions.
<!-- more -->


RBAC helps enforce **least privilege access**, where permissions are restricted to the minimal level required for users or services to complete their tasks, thus reducing the risk of accidental or malicious activities within the cluster.

### Key Components of Kubernetes RBAC

1. **Role/ClusterRole**:
    - A **Role** defines permissions (verbs such as get, list, create, etc.) for resources within a specific namespace.
    - A **ClusterRole** is similar but applies to the entire cluster or across multiple namespaces.
   
2. **RoleBinding/ClusterRoleBinding**:
    - A **RoleBinding** assigns a Role to users, groups, or service accounts within a specific namespace.
    - A **ClusterRoleBinding** assigns a ClusterRole to users, groups, or service accounts across the entire cluster.
   
3. **Users, Groups, and Service Accounts**:
    - These are the subjects to whom roles are assigned, dictating their access levels and permissions within the cluster.

### How RBAC Works in Kubernetes

RBAC associates users with roles through **RoleBindings** or **ClusterRoleBindings**. Roles and ClusterRoles define what actions (verbs) can be performed on specific resources.

For example:
- A **Role** may allow a user to view Pods within a particular namespace.
- A **ClusterRole** could provide an administrator with full access to all resources across the cluster.

**RBAC Structure**:
- **Role** or **ClusterRole**: Specifies **what** actions can be performed on **which** resources.
- **RoleBinding** or **ClusterRoleBinding**: Specifies **who** has access to the resources defined by the roles.

### Creating a Role in Kubernetes

A **Role** defines access permissions within a specific namespace. Below is an example of a YAML manifest for a **Role** that provides read-only access to Pods within a namespace:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

In this example:
- The **pod-reader** role grants permissions to perform the actions (verbs) `get`, `list`, and `watch` on Pods within the `dev` namespace.

### Creating a RoleBinding

A **RoleBinding** links a Role to a user, group, or service account in a specific namespace. Here's an example YAML manifest that binds the **pod-reader** role to a user:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: dev
subjects:
  - kind: User
    name: "jane" # Replace with the actual username
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

This **RoleBinding**:
- Assigns the **pod-reader** role to the user **jane** within the `dev` namespace.

### Creating a ClusterRole

A **ClusterRole** is used when access is required across the entire cluster or multiple namespaces. Below is an example of a **ClusterRole** that provides admin access to all resources:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin
rules:
  - apiGroups: [""]
    resources: ["*"]
    verbs: ["*"] # Allows all actions (create, delete, update, etc.)
```

In this example:
- The **cluster-admin** ClusterRole grants full permissions (`*` for all actions) on all resources within all API groups.

### Creating a ClusterRoleBinding

A **ClusterRoleBinding** is used to assign a **ClusterRole** to a user or group, allowing them access across the entire cluster. Here's an example:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-access
subjects:
  - kind: User
    name: "admin" # Replace with the actual username
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

This **ClusterRoleBinding**:
- Assigns the **cluster-admin** ClusterRole to the user **admin**, giving them administrative access throughout the cluster.

### Assigning Permissions to a Service Account

Service accounts are often used to assign roles to applications running within the cluster. Below is an example of how to bind a **Role** to a service account:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-to-service-account
  namespace: dev
subjects:
  - kind: ServiceAccount
    name: my-service-account
    namespace: dev
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

In this example:
- The **my-service-account** service account is bound to the **pod-reader** role, allowing it to read Pods within the `dev` namespace.

### Listing Roles and RoleBindings

To view the existing roles and bindings in your Kubernetes cluster, you can use the following commands:

- List all **Roles** in a specific namespace:
  ```bash
  kubectl get roles -n <namespace>
  ```

- List all **RoleBindings** in a specific namespace:
  ```bash
  kubectl get rolebindings -n <namespace>
  ```

- List all **ClusterRoles**:
  ```bash
  kubectl get clusterroles
  ```

- List all **ClusterRoleBindings**:
  ```bash
  kubectl get clusterrolebindings
  ```

### Best Practices for RBAC in Kubernetes

1. **Principle of Least Privilege**: Always assign the least amount of access necessary. Avoid granting broad permissions unless absolutely necessary.
2. **Namespace Isolation**: Assign roles within specific namespaces to limit access scope. Use cluster-wide roles only when needed.
3. **Audit Role and RoleBinding Usage**: Regularly review the roles and bindings in use to ensure they comply with current security policies.
4. **Use Service Accounts for Applications**: Assign permissions to applications using service accounts instead of user credentials for better security management.

### Conclusion

Kubernetes RBAC provides a robust system for managing access control in clusters. By defining **Roles** and **ClusterRoles** and associating them with users, groups, or service accounts, you can ensure that each entity in the cluster has only the necessary permissions. Following best practices, such as the principle of least privilege and namespace isolation, will significantly improve your cluster's security.