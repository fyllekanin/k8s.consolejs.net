## Creating a kubeconfig based on a service account

### Script
```
# Cosmetics for the created config
clusterName='<name>'
# your server address goes here get it via `kubectl cluster-info`
server='https://<ip>:<port>'
# the Namespace and ServiceAccount name that is used for the config
namespace='<namespace for service account>'
serviceAccount='<acc name>'

# The following automation does not work from Kubernetes 1.24 and up.
# You might need to
# define a Secret, reference the ServiceAccount there and set the secretName by hand!
# See https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#manually-create-a-long-lived-api-token-for-a-serviceaccount for details
secretName=test-secret

######################
# actual script starts
set -o errexit


ca=$(kubectl --namespace="$namespace" get secret/"$secretName" -o=jsonpath='{.data.ca\.crt}')
token=$(kubectl --namespace="$namespace" get secret/"$secretName" -o=jsonpath='{.data.token}' | base64 --decode)

echo "
---
apiVersion: v1
kind: Config
clusters:
  - name: ${clusterName}
    cluster:
      certificate-authority-data: ${ca}
      server: ${server}
contexts:
  - name: ${serviceAccount}@${clusterName}
    context:
      cluster: ${clusterName}
      namespace: ${namespace}
      user: ${serviceAccount}
users:
  - name: ${serviceAccount}
    user:
      token: ${token}
current-context: ${serviceAccount}@${clusterName}
```
