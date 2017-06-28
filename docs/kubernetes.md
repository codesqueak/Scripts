# How to Install Kubernetes on Centos 7

## Initial Setup

Basic enviroment changes, make sure we have sudo and give the machine a defined host name (kubemaster in this case)

```bash
usermod -aG wheel <username>
hostnamectl set-hostname kubemaster
```

## Storage for Docker

Storage under Dcoker is a big subject.  This is a basic method to get you working

To function, Docker requires at minimum 2G of free space, which in this instance is drawn from a defined **Volume Group**

To check the present status use
```bash
sudo vgdisplay
```
Look for the **VG Name** which you will require later and **FREE PE / size** which should be a minimum of 2G.
If storage is available, move on to the next step, else continue here.

The simplest way to add storage is to identify unused space using
```bash
sudo lsblk
```
On a VM, to obtain extra space is to add another disk. For this example, I have added a new device labelled **/dev/sdb**. To use
this new space we have to add it to the physical volume manager and then extend the Volme Group (Identified as **VG Name** earlier).
```bash
sudo pvcreate /dev/sdb
sudo vgextend <VG Name> /dev/sdb

```
## Install Docker and Kubernetes

Open a terminal, sudo, and run this:
```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
setenforce 0
yum install -y docker kubelet kubeadm kubernetes-cni
systemctl enable docker && systemctl start docker
systemctl enable kubelet && systemctl start kubelet
```
Once complete (it may take a few minutes), run this next command to verify storage is working correctly.
```bash
systemctl status docker-storage-setup.service
```

## Create a Cluster

Use [kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/) to do an initial cluster install. This may take several minutes to complete.
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
Once the install is complete, make a note of the **kubeadm join** command as you will need this later to add further nodes.

Also execute the commands listed to get useage of **kubectl**
```bash
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf
```
The final stage is to add a network to the cluster.  The one is chose is flannel.
```bash
kubectl create -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel-rbac.yml
kubectl create -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
Since we only have the master at this stage, we need to the Kubernetes it can schedule pods here.  To do this, run:
```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```
If you now run the following command, you should see all the Kubernetes relatedservices running:
```bash
kubectl get pods --all-namespaces
```
Giving something like:
```
NAMESPACE     NAME                                 READY     STATUS    RESTARTS   AGE
kube-system   etcd-kubemaster                      1/1       Running   0          4m
kube-system   kube-apiserver-kubemaster            1/1       Running   0          4m
kube-system   kube-controller-manager-kubemaster   1/1       Running   0          4m
kube-system   kube-dns-692378583-tg7p3             2/3       Running   3          9m
kube-system   kube-flannel-ds-8qs0t                2/2       Running   0          4m
kube-system   kube-proxy-hw6lg                     1/1       Running   0          9m
kube-system   kube-scheduler-kubemaster            1/1       Running   0          4m

```




