# k8s-cluster-setup
kubernetes master worker node setup on redhat linux


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
6. Cloned the machine to launch 2 worker nodes
7. edit /etc/hosts -->
```
	192.168.1.6 kube-master
        192.168.1.3 wn1
        192.168.1.4 wn2
```
8. Ports used by master 
        |Protocol|  Direction Port Range | Purpose Used By|
	|--------|-----------------------|----------------|
        TCP       Inbound   6443        Kubernetes API server All
        TCP       Inbound   2379-2380   etcd server client API  kube-apiserver, etcd
        TCP       Inbound   10250       Kubelet API Self, Control plane
        TCP       Inbound   10259       kube-scheduler  Self
        TCP       Inbound   10257       kube-controller-manager Self
9. Allowing these ports on master
```
        sudo firewall-cmd --add-port=6443/tcp --permanent
        sudo firewall-cmd --add-port=2379-2380/tcp --permanent
        sudo firewall-cmd --add-port=10250/tcp --permanent
        sudo firewall-cmd --add-port=10259/tcp --permanent
        sudo firewall-cmd --add-port=10257/tcp --permanent
        sudo firewall-cmd --reload
        sudo firewall-cmd --list-all
```
10. Kubelet and node sevice uses these ports
```
        Protocol  Direction Port Range  Purpose Used By
        TCP       Inbound   10250       Kubelet API Self, Control plane
        TCP       Inbound   30000-32767 NodePort Services†  All
```
11. Allowing these ports on wn1 and wn2
```
	sudo firewall-cmd --add-port=10250/tcp --permanent
        sudo firewall-cmd --add-port=30000-32767/tcp --permanent

        sudo firewall-cmd --reload
        sudo firewall-cmd --list-all
```
12. Kubernetes required the kernel modules "overlay" and "br_netfilter" to be enabled on all servers. ON MASTER
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
