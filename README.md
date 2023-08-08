# terraform-kubernetes-rbac

A Terraform module for managing [Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/).

## Usage

Recreate the Kubernetes RBAC examples from the [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) documentation.

```hcl

#------------Local Variables------------
locals {
  labels = {
    "app.kubernetes.io/managed-by" = "terraform"
    "terraform.io/module"          = "terraform-kubernetes-rbac"
  }
}

#------------Module For Roles, Role-bindings, Cluster-Roles AND Cluster-Role-Bindings------------
module "rbac" {
  source = "./modules/kubernetes-rbac"

  labels = local.labels

  #Cluster Roles are Cluster Wide and can be used with Role-bindings and Cluster-ole-bindings
  #Mulitple Cluster Roles can be Specified in the same cluster_roles map
  #"namespace" omitted since ClusterRoles are not namespaced
  cluster_roles = {
    # 1 - Cluster-Role with Role-Binding
    "secret-reader" = { # Specify the Cluster Role Name
      cluster_role_rules = [
        {
          api_groups = [""]
          resources  = ["secrets"]
          verbs      = ["get", "watch", "list"]
        },
      ]

      role_binding_name      = "read-secret" # Specify name for the Role Binding
      role_binding_namespace = "default"     #grants permissions within the "default" namespace.
      role_binding_subjects = [
        {
          kind = "Group"
          name = "group-user"
        }
      ]
    },

    # 2-  Cluster-Role with Cluster-Role-Binding
    "secret-reader-global" = { # Specify the Cluster Role Name
      cluster_role_rules = [
        {
          api_groups = [""]
          resources  = ["secrets"]
          verbs      = ["get", "watch", "list"]
        },
      ]

      cluster_role_binding_name = "read-secrets-global" # Specify name for the Role Binding
      cluster_role_binding_subjects = [
        {
          kind      = "ServiceAccount"
          name      = "manager"
          namespace = "development"
        },
        {
          kind = "Group"
          name = "sada" # Name is case sensitive
        }
      ]
    }
  }


  #Roles are Specific to Namespace and can work with only Role-bindings
  #Mulitple Roles can be Specified in the same Roles map
  roles = {
    "pod-reader" = {             # Specify the role name
      role_namespace = "default" # Specify NameSpace Name Where Role needs to be created
      role_rules = [
        {
          api_groups = [""]
          resources  = ["pods"]
          verbs      = ["get", "watch", "list"]
        },
      ]

      role_binding_name = "read-pods" # Specify the role binding name
      role_binding_subjects = [
        {
          kind = "Group"
          name = "test" # Name is case sensitive
        }
      ]
    },
  }
}

```

## Requirements

| Name                                                                        | Version   |
| --------------------------------------------------------------------------- | --------- |
| <a name="requirement_terraform"></a> [terraform](#requirement_terraform)    | >= 0.13.1 |
| <a name="requirement_kubernetes"></a> [kubernetes](#requirement_kubernetes) | >= 2.7.0  |

## Providers

No providers.

## Modules

| Name                                                                       | Source         | Version |
| -------------------------------------------------------------------------- | -------------- | ------- |
| <a name="module_cluster_roles"></a> [cluster_roles](#module_cluster_roles) | ./modules/rbac | n/a     |
| <a name="module_roles"></a> [roles](#module_roles)                         | ./modules/rbac | n/a     |

## Resources

No resources.

## Inputs

| Name                                                                     | Description                                                                                      | Type          | Default | Required |
| ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------ | ------------- | ------- | :------: |
| <a name="input_annotations"></a> [annotations](#input_annotations)       | The global annotations. Applied to all resources.                                                | `map(string)` | `{}`    |    no    |
| <a name="input_cluster_roles"></a> [cluster_roles](#input_cluster_roles) | Map of cluster role and cluster/role binding definitions to create.                              | `any`         | `{}`    |    no    |
| <a name="input_create"></a> [create](#input_create)                      | Controls whether the Authorization and RBAC resources should be created (affects all resources). | `bool`        | `true`  |    no    |
| <a name="input_labels"></a> [labels](#input_labels)                      | The global labels. Applied to all resources.                                                     | `map(string)` | `{}`    |    no    |
| <a name="input_roles"></a> [roles](#input_roles)                         | Map of role and role binding definitions to create.                                              | `any`         | `{}`    |    no    |

## Outputs

| Name                                                                       | Description        |
| -------------------------------------------------------------------------- | ------------------ |
| <a name="output_cluster_roles"></a> [cluster_roles](#output_cluster_roles) | The cluster roles. |
| <a name="output_roles"></a> [roles](#output_roles)                         | The roles.         |
