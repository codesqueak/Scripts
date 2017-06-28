# Virtual Box Guest Additions for Centos 7

## Update Centos

Run the following commands.

```bash
cat <<EOF > vbox.sh
yum -y update 
yum -y install epel-release
yum -y groupinstall "Development Tools"
yum -y install kernel-devel
yum -y install epel-release
yum -y install dkms
yum -y update
EOF
chmod +x vbox.sh
sudo ./vbox.sh

```
**Reboot**

Load the guest additions CD (*Devices -> Insert Guest Additions CD Image ...*)

Open the CD in a terminal and run the following command
```bash
sudo ./VBoxLinuxAdditions.run
```
**Reboot**

Guest additions should now be installed and working



