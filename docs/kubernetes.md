# How to Install Kubernetes on Centos 7

## Initial Setup

Basic enviroment changes, make sure we have sudo and give the machine a defined host name (kubemaster in this case)

```bash
usermod -aG wheel <username>
hostnamectl set-hostname kubemaster
```

## Storage for Docker

Storage under Docker is a big subject!  This is a basic method to get you working. **DO NOT DO THIS IN PRODUCTION**

To function, Docker requires at minimum 2G of free space, which in this instance is drawn from a defined **Volume Group**

To check the present status use
```bash
sudo vgdisplay
```
Look for the **VG Name** which you will require later and **FREE PE / size** which should be a minimum of 2G (Recommend at least 10G for the example at the end of this document). 
If storage is available, move on to the next step, else continue here to add more.

The simplest way to add storage is to identify unused space using
```bash
sudo lsblk
```
On a VM, a fast way to obtain extra space is to add another disk. For this example, I have added a new device labelled **/dev/sdb**. To use
this new space we have to add it to the physical volume manager and then extend the Volme Group (Identified as **VG Name** earlier).
```bash
sudo pvcreate /dev/sdb
sudo vgextend <VG Name> /dev/sdb

```
## Configure Firewall

To run sucessfully, access to various ports is required. To configure, run this:
```bash
sudo firewall-cmd --zone=public --add-port=6443/tcp --permanent
sudo firewall-cmd --zone=public --add-port=9898/tcp --permanent
sudo firewall-cmd --zone=public --add-port=10250/tcp --permanent
sudo firewall-cmd --reload

```

## Install Docker and Kubernetes

Open a terminal, sudo -i, and run this:
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

## Possible Security Issue

As the moment, cluster creation may fail due to SELinux restrictions.  A workaround is to set permissive mode.  **DO NOT DO THIS IN PRODUCTION**. Investigate further.

Open /etc/selinux/config and set SELINUX=permissive

See [this](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security-Enhanced_Linux/sect-Security-Enhanced_Linux-Enabling_and_Disabling_SELinux-Disabling_SELinux.html)

You will need to reboot to make this take effect

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
If you now run the following command, you should see all the Kubernetes related services running:
```bash
kubectl get pods --all-namespaces
```
Giving something like:
```
NAMESPACE     NAME                                 READY     STATUS    RESTARTS   AGE
kube-system   kube-apiserver-kubemaster            1/1       Running   0          13s
kube-system   kube-controller-manager-kubemaster   1/1       Running   0          21s
kube-system   kube-dns-692378583-rvxtl             0/3       Pending   0          1m
kube-system   kube-flannel-ds-88w87                2/2       Running   0          17s
kube-system   kube-proxy-cbwln                     1/1       Running   0          1m
kube-system   kube-scheduler-kubemaster            1/1       Running   0          20s

```

## Run a Test Application

Now you have a working cluster, try running this test application:

```bash
kubectl apply -f "https://github.com/microservices-demo/microservices-demo/blob/master/deploy/kubernetes/complete-demo.yaml?raw=true"
```

Once all containers are shown as **Running**, use
```bash
kubectl -n sock-shop get svc front-end
```
To identify the application address (CLUSTER-IP)


Open a browser on the host machine and check the application is working correctly.


## Tear Down

If you need to remove everything installed by **kubeadm init**, run
```bash
kubectl drain kubemaster --delete-local-data --force --ignore-daemonsets
kubectl delete node kubemaster
sudo kubeadm reset
```







