# k8s-cluster-setup
kubernetes master worker node setup on redhat linux
[Demo](https://drive.google.com/drive/folders/1rbvoELKErRhjr8c2eF71ERsMCNGYRnUD?usp=sharing)
[Blog](https://sunnykkc13.medium.com/kubernetes-setup-489ecb64a896)

Date -- November 4, 2022
Steps to install K8S on Rocky Linux
1. add k8s repo
```
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
```
2. disable swap --> edit `/etc/fstab` --> `sudo swapoff -a`
3. sudo `setenforce 0`
4. sudo `sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config`
5. sestatus
6. Kubernetes required the kernel modules "overlay" and "br_netfilter" to be enabled on all servers. ON MASTER
```
sudo modprobe overlay
sudo modprobe br_netfilter
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system
```
7. Install docker
```
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install docker-ce
sudo systemctl start docker && sudo systemctl restart docker
```
---
```
sudo nano /etc/containerd/config.toml
# remove cri from disabled_plugins
```
---
```
sudo systemctl restart containerd
#sudo systemctl status containerd
```
8. Install kubernetes components
```
sudo dnf install kubelet kubeadm kubectl --disableexcludes=kubernetes
```
9. Cloned the machine to launch 2 worker nodes
10. edit /etc/hosts -->
```
192.168.1.6 kube-master
192.168.1.3 wn1
192.168.1.4 wn2
```
11. Ports used by master 
|Protocol|  Direction Port Range | Purpose Used By|
|--------|-----------------------|----------------|
TCP       Inbound   6443        Kubernetes API server All
TCP       Inbound   2379-2380   etcd server client API  kube-apiserver, etcd
TCP       Inbound   10250       Kubelet API Self, Control plane
TCP       Inbound   10259       kube-scheduler  Self
TCP       Inbound   10257       kube-controller-manager Self
12. Allowing these ports on master
```
sudo firewall-cmd --add-port=6443/tcp --permanent
sudo firewall-cmd --add-port=2379-2380/tcp --permanent
sudo firewall-cmd --add-port=10250/tcp --permanent
sudo firewall-cmd --add-port=10259/tcp --permanent
sudo firewall-cmd --add-port=10257/tcp --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```
13. Kubelet and node sevice uses these ports
```
Protocol  Direction Port Range  Purpose Used By
TCP       Inbound   10250       Kubelet API Self, Control plane
TCP       Inbound   30000-32767 NodePort Services†  All
```
14. Allowing these ports on wn1 and wn2
```
sudo firewall-cmd --add-port=10250/tcp --permanent
sudo firewall-cmd --add-port=30000-32767/tcp --permanent

sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```
15. Pull the images of master components
```
sudo kubeadm config images pull
```
16. on masternode add kubeadm-config.yaml
```
#kubeadm-config.yaml
kind: ClusterConfiguration
apiVersion: kubeadm.k8s.io/v1beta3
kubernetesVersion: v1.25.3
---
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd
```
17. initialise the cluster using kubeadm
```
sudo kubeadm init --config kubeadm-config.yaml
```
18. If successfully initialised then run
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
19. To join the cluster, on worker node run
```
sudo kubeadm join 192.168.1.16:6443 --token <your_generated_token>         --discovery-token-ca-cert-hash <your_generated_certs>
```
