### How to Install a Kubernetes Cluster on CentOS 7

```
Multiple servers running Centos 7 (1 Master Node, 2 Worker Nodes). It is recommended that your Master Node have at least 2 CPUs, though this is not a strict requirement.
Internet connectivity on all your nodes. We will be fetching Kubernetes and docker packages from the repository. Equally, you will need to make sure that the yum package manager is installed by default and can fetch packages remotely.
You will also need access to an account with sudo or root privileges. In this tutorial, I will be using my root account.
```

* Installation of Kubernetes Cluster on Master-Node

- Step 1: Prepare Hostname, Firewall and SELinux

```
hostnamectl set-hostname master-node
cat <<EOF>> /etc/hosts
10.128.0.27 master-node
10.128.0.29 node-1 worker-node-1
10.128.0.30 node-2 worker-node-2
EOF

setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
reboot

modprobe br_netfilter

echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

```

* Install Kubeadm and Docker

```
yum install kubeadm docker -y 

systemctl enable kubelet
systemctl start kubelet
systemctl enable docker
systemctl start docker
swapoff -a
```

*  Update Iptables Settings
```
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

* Initializing Kubernetes master is a fully automated process that is managed by the “kubeadm init“ command which you will run.

```
kubeadm init 
kubeadm init --apiserver-advertise-address=192.168.33.101
sudo kubeadm init --apiserver-advertise-address=192.168.30.128 --pod-network-cidr=10.244.0.0/16

kubeadm join 192.168.30.128:6443 --token gyjp2p.gxq4gj9neveitl4y --discovery-token-ca-cert-hash sha256:a7acb1f8e6769d1adaf5d09041d67871b753536135350ed325a8cccbc926eba6
```



