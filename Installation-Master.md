# kubernetes-vagrant-lab (Install Master node)

## 1. Prérequis système

### 1.1. container runtime

```bash
sudo apt-get update
sudo apt-get install containerd -y
```

### 1.2. Align cgroup driver between kubelet and container runtime

```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/' | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

## 2. Installation kubeadm, kubelet, kubectl

```bash
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
# If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## 3. Prepare network kernel

### 3.1. Enable bridge traffic by loading module br_netfilter
```bash
echo 'br_netfilter' | sudo tee /etc/modules-load.d/k8s.conf
```
### 3.2. Adding some systctl for firewall and routing
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
```
```bash
sudo sysctl --system
```
```bash
sysctl net.ipv4.ip_forward
```
## 4. Install Master Nodes
```bash
sudo kubeadm init \
 --apiserver-advertise-address=XXXX \
 --pod-network-cidr=XXXX
```
-- apiserver-advertise-address defines the ip adress for the interface enp0s8.

--pod-network-cidr defines the IP range used for Pod-to-Pod communication,
and must be aligned with the CNI configuration (Calico in this case). you can use 10.244.0.0/16

### Note:

With Calico, if CALICO_IPV4POOL_CIDR is not explicitly set in the manifest,
Calico automatically creates a default IPPool using the value provided to kubeadm (--pod-network-cidr).
After installation you can check

```bash
kubectl get ippools
```
you can see: **default-ipv4-ippool**

## 5. Set authorization for the principal user to the master node
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 6. Node IP selection (kubelet)
In our environments (Vagrant/Virtualbox), nodes have both NAT and bridged interfaces.
- a NAT interface (enp0s3: e.g 10.0.2.x)
- a bridged interface (enp0s8: eg 192.168.x.x)

Kubelet may select the NAT IP by default, causing networking issues.
Always explicitly configure kubelet to use the bridged interface IP via --node-ip.
This is achieved by configuring kubelet with the --node-ip flag.

````bash
#/etc/default/kubelet
KUBELET_EXTRA_ARGS=--node-ip=x.x.x.x
````
x.x.x.x is the ip address for the interface bridge enp0s8
ex: 192.168.1.22

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

## 7. Download calico manifest
````bash
#download manifest
curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/calico.yaml
#install calico
kubectl apply -f calico.yaml  
````