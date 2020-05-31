# Install Kubernetes cluster on Ubuntu 20.04 server

To manualy deploy Kubernetes cluster on physical or virtual machines follow this procesure:

## Pre-requisites
Two or more physical or virtual machines with network card and Ubuntu 20.04 installed. One machine for Kubernetes Master and at least one for Kubernetes node.
- Kubernates master must have at least 2 CPUs (at least 2 CPU cores, not 2 physical CPUs) and 2GB RAM.
- Nodes can be as small as 1 CPU and 1GB of RAM.

## Installing Kubernetes Master
First update the machine:
```
sudo apt-get update && sudo apt-get upgrade -y
```

### Install Docker
```
sudo apt-get install docker.io -y
```
Start Docker service and make it permanent
```
sudo systemctl start docker
sudo systemctl enable docker
```

### Install Kubernetes
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt-get install kubeadm kubelet kubectl -y
```

### Set Hostname and hosts file
Set hostname
```
sudo hostnamectl set-hostname kubemaster
```
Edit **/etc/hosts** file as root or edit it in editor
```
cat >>/etc/hosts<<EOF
192.198.10.100 kubemaster.example.com kubemaster
192.198.10.101 kubenode1.example.com kubenode1
192.198.10.102 kubenode2.example.com kubenode2
EOF
```

### Turn swap off
Open **/etc/fstab** in editor
```
sudo nano /etc/fstab
```
comment out line with swapfile by addin # at the begining, save the file and execute
```
sudo swapoff -a
```

### Initialise pod network
Issue the command **without root privileges**:
```
kubeadm init --pod-network-cidr=192.168.10.100/24
```

### Copy Kubernetes config
Create folder **.kube** at yours home directory, copy config and change owner.
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Deploy pod network
```
sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

## Installing Kubernetes Node
First update the machine:
```
sudo apt-get update && sudo apt-get upgrade -y
```

### Install Docker
```
sudo apt-get install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```

### Install Kubernetes
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt-get install kubeadm kubelet kubectl -y
```

### Set Hostname and hosts file
Set hostname
```
sudo hostnamectl set-hostname kubemaster
```
Edit **/etc/hosts** file as root or edit it in editor
```
cat >>/etc/hosts<<EOF
192.198.10.100 kubemaster.example.com kubemaster
192.198.10.101 kubenode1.example.com kubenode1
192.198.10.102 kubenode2.example.com kubenode2
EOF
```

### Turn swap off
Open **/etc/fstab** in editor
```
sudo nano /etc/fstab
```
comment out line with swapfile by addin # at the begining, save the file and execute
```
sudo swapoff -a
```

### Join Kubernetes cluster as Node

To generate cluster join command, on Kubernetes Master execute
```
kubeadm token create --print-join-command
```

This command should look like this:
- kubeadm join 192.168.10.6:6443 --token 9gvd32.nnoqp4iwowo34gs6     --discovery-token-ca-cert-hash sha256:847f04527874789054a1167e93fe7d905666478e6db8e2f7634a9cf6b6a80088

Copy command generated by **yours** Kubernetes Master, and execute it on Kubernetes Node but with **sudo** privilegues.

Repeat this proces for all nodes.
