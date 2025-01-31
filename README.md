# Kubernetes installation steps
===================================
Multi-Node Kubernetes Cluster Setup using Kubeadm
1. Prerequisites
Before setting up the cluster, ensure the following:
Ubuntu 20.04+ installed on all nodes
Minimum 2 CPUs and 2GB RAM per node
Root or sudo privileges
Unique hostnames for all nodes
Internet access
2. Configure System Settings on All Nodes
Disable Swap
sudo su -

sudo swapoff -a

sed -i '/ swap / s/^/#/' /etc/fstab
Configure sysctl Parameters for Kubernetes
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system

sysctl net.ipv4.ip_forward

3. Install Container Runtime (Containerd)
Add Docker Repository
sudo apt-get update

sudo apt-get install -y ca-certificates curl

sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc


Add Docker Repository to APT Sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

Install Containerd
sudo apt-get install -y containerd.io

containerd config default > /etc/containerd/config.toml
Modify Containerd Configuration
Edit the configuration file:
sudo vi /etc/containerd/config.toml
Find and set:
SystemdCgroup = true
Restart Containerd
sudo systemctl restart containerd
sudo systemctl status containerd

4. Install Kubernetes Components
Add Kubernetes Repository
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
Install Kubeadm, Kubelet, and Kubectl
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl


ONLY MASTER:
5. Initialize Kubernetes Control Plane Node
 kubeadm init --apiserver-advertise-address 172.31.37.62 (privateIP) --pod-network-cidr 10.244.0.0/16

Set Up kubectl for the User
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
Deploy a Network Plugin (Flannel)
kubectl apply -f https://reweave.azurewebsites.net/k8s/v1.32/net.yaml
6. Add Worker Nodes to the Cluster
Run the following command on each Worker Node:
sudo kubeadm join 172.31.37.62:6443 --token wgla2v.gc0mhnbvg2b9j6hi \
    --discovery-token-ca-cert-hash sha256:f584f919e31fa69af0d71cb7f4775e0002073bc77165a7ab2f2ac346a7e71ae9
To generate a new join token if the previous one expires, run the following on the Control Plane Node:
sudo kubeadm token create --print-join-command
7. Verify Cluster Setup
kubectl get nodes
kubectl get pods -n kube-system


Links:
https://github.com/rajch/weave#using-weave-on-kubernetes

https://kubernetes.io/docs/concepts/cluster-administration/addons/

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/creat
e-cluster-kubeadm/

https://docs.docker.com/engine/install/ubuntu/


https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/







