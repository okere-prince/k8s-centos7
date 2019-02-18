# HOW TO INSTALL KUBERNETES ON CENTOS 7

### Disable SELinux.
```
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g'
/etc/sysconfig/selinux
```

### Enable the br_netfilter module for cluster communication.
```
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
```

### Disable swap to prevent memory allocation issues.

```
swapoff -a
vim /etc/fstab.  ->  Comment out the swap line
```

### Install Docker CE.

#### Install the Docker prerequisites.
```
yum install -y yum-utils device-mapper-persistent-data lvm2
```
#### Add the Docker repo and install Docker.

```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
 yum install -y docker-ce
```
### Add the Kubernetes repo.
 ```
 cat <<EOF > /etc/yum.repos.d/kubernetes.repo
 [kubernetes]
 name=Kubernetes
 baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
 enabled=1
 gpgcheck=0
 repo_gpgcheck=0
 gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
         https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
 EOF
 ```
 
### Install Kubernetes.
```
yum install -y kubelet kubeadm kubectl
Reboot.
```

### Enable and start Docker and Kubernetes.
```
systemctl enable docker
systemctl enable kubelet
systemctl start docker
systemctl start kubelet
```
### Check the group Docker is running in.
```
docker info | grep -i cgroup
```

### Set Kubernetes to run in the same group.
```
sed -i 's/cgroup-driver=systemd/cgroup-driver=cgroupfs/g' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

### Reload systemd for the changes to take effect, and then restart Kubernetes.
```
systemctl daemon-reload
systemctl restart kubelet
```

**Note:** Complete the following section on the MASTER ONLY!

### Initialize the cluster using the IP range for Flannel.
```
kubeadm init --pod-network-cidr=10.244.0.0/16
```
**Note:** Copy the kubeadmin join command.

### Deploy Flannel.
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
Exit sudo and run the following:
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Check the cluster state.
```
Kubectl get pods --all-namespaces
```

**Note:** Complete the following steps on the NODES ONLY!

Run the join command that you copied earlier, then check your nodes from the master.
```
kubectl get nodes
```
**Note:** make sure to disable the firewalld in the master and nodes in order to let the nodes able to connect with the cluster.


### ENJOY
