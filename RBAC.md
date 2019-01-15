## RBAC

#### Role: 
A Role is used to grant access to resources within a single namespace.
#### ClusterRole: 
A ClusterRole is used to grant access to resources within a cluster. Because they are cluster-scoped, they can also be used to grant access to:

```
1. cluster-scoped resources (like nodes)
2. non-resource endpoints (like “/healthz”)
3. namespaced resources (like pods) across all namespaces [ie: to run $ kubectl get pods --all-namespaces]
```
#### RoleBinding
A RoleBinding grants the permissions defined in a role (namespaced) to a user or set of users in the same namespace.
It can also grant permission defined in a clusterRole to a user or set of users in a specified namespace. 
This allows administrators to define a set of common roles for the entire cluster, then reuse them within multiple namespaces.
#### ClusterRoleBinding
ClusterRoleBinding is used to grant permission at the cluster level and in all namespaces.
#### Aggregated ClusterRoles
ClusterRoles can be created by combining other ClusterRoles using an aggregationRule.


