# Kubernetes – Install Argo CD (NodePort, Vagrant lab)

## 1. Prérequis système
Provision a Kubernetes cluster (1 control-plane, 2 workers) using Vagrant.
``` bash
git clone https://github.com/kaninda/kubernetes-vagrant-lab.git
```
## 2. Install argoCD on controlplane 
``` bash
# https://github.com/argoproj/argo-cd/releases/tag/v3.0.21
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v3.0.21/manifests/install.yaml
```
The latest Argo CD version at the time of writing is 3.0.21.
## 3. Patch argocd service as nodeport
``` bash
kubectl patch svc argocd-server -n argocd   -p '{"spec":{"type":"NodePort"}}'
```
## 4. Access Argo CD
### 4.1. From the Kubernetes node
``` bash
https://<NODE_IP>:<NODEPORT>
```
### 4.2. From the host machine (recommended for Vagrant NAT)
``` bash
https://argocd.lab.local:<NODEPORT>
```
edit
``` bash
#C:\Windows\System32\drivers\etc\hosts
X.X.X.X argocd.lab.local
```
X.X.X.X ip adress of master node 

## 5. Extract password to get connect tp argocd
``` bash
k get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```
user=admin

## 6. Install argocd CLI
``` bash
#install argocd CLI
curl -sSL -o /usr/local/bin/argocd \
  https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
```
``` bash
chmod +x /usr/local/bin/argocd
```
## 7. Connect to argocd CLI
``` bash
login 192.168.1.40:30297 --insecure
```



