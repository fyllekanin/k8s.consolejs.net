
## Create a new namespace

```
{
  "apiVersion": "v1",
  "kind": "Namespace",
  "metadata": {
    "name": "<namespace>",
    "labels": {
      "name": "<namespace>"
    }
  }
}
```

## Create user credentials
Keep track of:
username
group

```
openssl genrsa -out <username>.key 2048
openssl req -new -key <username>.key -out <username>.csr -subj "/CN=<username>/O=<group>"
openssl x509 -req -in <username>.csr -CA /var/lib/rancher/k3s/server/tls/client-ca.crt -CAkey /var/lib/rancher/k3s/server/tls/client-ca.key -CAcreateserial -out <username>.crt -days 500
```

Store the **<username>.crt** and **<username>.key** in a good place

## Add the user

```
kubectl config set-credentials <username> --client-certificate=/path/to/<username>.crt --client-key=/path/to/<username>.key
kubectl config set-context <username>-context --namespace=<namespace> --user=<username>
```

## Create the role

```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: <namespace>
  name: deployment-manager
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "replicasets", "pods", "ingress"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

## Bind the role to the user

```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: deployment-manager-binding
  namespace: <namespace>
subjects:
- kind: User
  name: <username>
  apiGroup: ""
roleRef:
  kind: Role
  name: deployment-manager
  apiGroup: ""
```

## Create kubeconfig

```
kubectl config view --flatten > kubeconfig.yaml
```

Update kubeconfig.yaml
- Remove default user
- Remove default context
- Update server to "k8s.consolejs.net"
- Update cluster name from "default" to "<cluster-name>"
- Update cluster in context to "<cluster-name>

```
kubectl config unset users.<username>
kubectl config unset contexts.<username>-context
```