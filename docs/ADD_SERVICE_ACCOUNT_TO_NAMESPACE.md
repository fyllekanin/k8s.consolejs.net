# Add a service account to a namespace 

## Pre-requirements
- Setup K3S

## Lets get going

### Variables to update in all snippets
{account-name}
{role-name}
{binding-name}
{token-secret-name}
{namespace}
  
### Create the service account

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: {account-name}
  namespace: {namespace}
```

### Create the role the account will have later

Here you can also update which apiGroups, resources and vers the service account will have access to

```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: {namespace}
  name: {role-name}
rules:
- apiGroups: ["", "extensions", "apps", "networking.k8s.io"]
  resources: ["deployments", "replicasets", "pods", "ingresses", "services"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

### Create a binding between the role and the account
```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: {binding-name}
  namespace: {namespace}
subjects:
- kind: ServiceAccount
  name: {account-name}
  namespace: {namespace}
roleRef:
  kind: Role
  name: {role-name}
  apiGroup: rbac.authorization.k8s.io
```

### Lastly, lets create a secret which will be populated with the access token for the account
```
apiVersion: v1
kind: Secret
metadata:
  name: {token-secret-name}
  namespace: {namespace}
  annotations:
    kubernetes.io/service-account.name: {account-name}
type: kubernetes.io/service-account-token
```