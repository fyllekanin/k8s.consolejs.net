# K3S Setup

## Pre-requirements
- Update and upgrade the machine
- Do not run as root

## Installing K3S

```
curl -sfL https://get.k3s.io | sh -s -

```

Set the config permissions correct for kubectl
```
sudo chmod 644 /etc/rancher/k3s/k3s.yaml
```

Remove traefik
```
kubectl -n kube-system delete helmcharts.helm.cattle.io traefik
```

Update the service
```
sudo service k3s stop
Add:
    --disable=traefik \
    --write-kubeconfig-mode=644 \
Reload the daemon "systemctl daemon-reload"
sudo service k3s start
```
## Installing Nginx Ingress

Go to https://github.com/nginxinc/kubernetes-ingress/releases and find the latest release

Clone down the repository for the release
```
git clone https://github.com/nginxinc/kubernetes-ingress.git --branch <version>
```
  
And enter the kubernetes-ingress/deployments

### Create namespace and service account
```
kubectl apply -f common/ns-and-sa.yaml
```
### Create cluster role and cluster role binding for the SA
```
kubectl apply -f rbac/rbac.yaml
```
### Create common secret
```
kubectl apply -f common/default-server-secret.yaml
```
### Create config map
```
kubectl apply -f common/nginx-config.yaml
```
### Create IngressClass
```
kubectl apply -f common/ingress-class.yaml
```
### Setup custom resources
```
kubectl apply -f common/crds/k8s.nginx.org_virtualservers.yaml
kubectl apply -f common/crds/k8s.nginx.org_virtualserverroutes.yaml
kubectl apply -f common/crds/k8s.nginx.org_transportservers.yaml
kubectl apply -f common/crds/k8s.nginx.org_policies.yaml
```
### Add load balacing posibility
```
kubectl apply -f common/crds/k8s.nginx.org_globalconfigurations.yaml
```
### Setup the daemonset
```
kubectl apply -f daemon-set/nginx-ingress.yaml
```
