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




