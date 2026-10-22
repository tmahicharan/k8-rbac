# Kubernetes RBAC and EKS Access

## Overview

This repository contains Kubernetes YAML manifests demonstrating Role-Based Access Control (RBAC), user and group permissions, ServiceAccounts, and AWS access integration in Amazon EKS.

It covers namespace-scoped permissions, cluster-scoped permissions, RoleBindings, ClusterRoleBindings, EKS identity mapping, and an AWS CLI Pod.

## Repository Structure

```text
k8-rbac/
├── 01-role.yaml
├── 02-rolebinding.yaml
├── 03-aws-auth.yaml
├── 04-admin-role.yaml
├── 05-admin-rolebinding.yaml
├── 06-group-rolebinding.yaml
├── 07-serviceacc.yaml
└── 08-pod.yaml
```

## Concepts Covered

### Role

`01-role.yaml` defines the `roboshop-trainee` Role in the `roboshop` namespace. It grants `get`, `list`, and `watch` permissions on Pods in that namespace.

### RoleBinding

`02-rolebinding.yaml` binds the `roboshop-trainee` Role to the user named `mahi`.

A RoleBinding grants the permissions defined by its referenced Role within the binding's namespace.

### EKS Identity Mapping

`03-aws-auth.yaml` contains an `aws-auth` ConfigMap for mapping AWS IAM identities to Kubernetes usernames and groups.

This file is cluster-specific. Review its existing mappings and your cluster's authentication configuration before using it. Do not replace a live ConfigMap without understanding the impact.

### Admin Roles and Bindings

`04-admin-role.yaml` defines two resources:

- A namespace Role named `roboshop-admin`, with wildcard resource and verb permissions inside `roboshop`.
- A ClusterRole named `roboshop-admin`, with read-only access to PersistentVolumes and StorageClasses.

`05-admin-rolebinding.yaml` contains a RoleBinding for the namespace Role and a ClusterRoleBinding for the ClusterRole. These bindings have different scopes and should be reviewed independently.

The namespace Role grants broad permissions. Avoid assigning it to users unless those permissions are explicitly required.

### Group RoleBinding

`06-group-rolebinding.yaml` demonstrates binding a Role to a group. Verify the subject's `kind`, group name, and referenced Role before applying it.

### ServiceAccount and AWS CLI Pod

`07-serviceacc.yaml` defines a ServiceAccount named `secrect-reader` in the `roboshop` namespace and includes an EKS IAM role annotation.

`08-pod.yaml` defines an AWS CLI Pod that uses this ServiceAccount.

The ServiceAccount annotation suggests an IAM Roles for Service Accounts (IRSA) configuration. Confirm that the IAM role trust policy, cluster OIDC provider, and AWS permissions are correctly configured. A ServiceAccount annotation alone does not guarantee access.

The manifests contain cluster-generated metadata. Review environment-specific fields before reusing them in another cluster.

## Prerequisites

- A Kubernetes cluster with `kubectl` configured.
- Permission to inspect and manage RBAC resources.
- An existing `roboshop` namespace.
- For EKS-specific resources, appropriate AWS CLI access and cluster permissions.
- Correct IAM trust and permission policies if using IRSA.

## Validation

Inspect the current resources:

```bash
kubectl get roles,rolebindings -n roboshop
kubectl get serviceaccounts,pods -n roboshop
kubectl get clusterroles,clusterrolebindings
```

Review a Role and its binding:

```bash
kubectl describe role roboshop-trainee -n roboshop
kubectl describe rolebinding rolebinding-mahi -n roboshop
```

Check the permissions of the currently configured identity:

```bash
kubectl auth can-i get pods -n roboshop
kubectl auth can-i list pods -n roboshop
```

These checks reflect the identity used by the current `kubectl` context, not automatically the users referenced in the YAML files.

## Deployment Guidance

Review the manifests and confirm the intended permissions before applying them.

For a non-production test cluster, the general workflow is:

```bash
kubectl apply --dry-run=client -f 01-role.yaml
kubectl apply --dry-run=client -f 02-rolebinding.yaml
```

Client-side dry-run checks basic manifest processing; it does not prove that the resources will be accepted by the cluster or that the permissions are correct.

Do not apply all files blindly. In particular, review `03-aws-auth.yaml`, the wildcard permissions in `04-admin-role.yaml`, both bindings in `05-admin-rolebinding.yaml`, and the IAM role annotation in `07-serviceacc.yaml` before making changes.

## Security Best Practices

- Follow least-privilege access.
- Avoid wildcard permissions unless necessary.
- Keep IAM permissions and Kubernetes RBAC permissions narrowly scoped.
- Protect cluster authentication mappings from accidental replacement.
- Do not commit credentials, tokens, or sensitive environment data.
- Test access changes in a development cluster before applying them to production.

## Disclaimer

This repository is intended for learning and demonstration. Review all permissions and environment-specific settings before using these manifests in a real cluster.
