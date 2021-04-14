# virt-manager-QEMU-KVM-installation-guide
QEMU, KVM, virt-manager installation guide for archlinux

## Install KVM packages

First step is installing all packages needed to run KVM:

```bash
sudo pacman -S qemu virt-manager virt-viewer dnsmasq vde2 bridge-utils openbsd-netcat
```

Also install ebtables  and iptables packages:

```bash
#sudo pacman -S ebtables iptables
```
```bash
sudo pacman -S iptables-nft 
```


## Install libguestfs

`libguestfs` is a set of tools used to access and modify virtual machine (VM) disk images. You can use this for:

- viewing and editing files inside guests
- scripting changes to VMs
- monitoring disk used/free statistics
- creating guests
- P2V
- V2V
- performing backup e.t.c

```bash
sudo pacman -S libguestfs
sudo systemctl enable libvirtd.service
sudo systemctl start libvirtd.service
```

also you might want to check if dnsmasq service is running

```bash
sudo systemctl enable dnsmasq.service
sudo systemctl start dnsmasq.service
```
## Enable normal user account to use KVM
Since we want to use our standard Linux user account to manage KVM, let’s configure KVM to allow this.

Open the file `/etc/libvirt/libvirtd.conf` for editing.

```bash
sudo vim /etc/libvirt/libvirtd.conf
```

Set the UNIX domain socket group ownership to libvirt

```
unix_sock_group = "libvirt"
```

Set the UNIX socket permissions for the R/W socket

```
unix_sock_rw_perms = "0770"
```

Add your user account to libvirt group.

```bash
sudo usermod -a -G libvirt $(whoami)
newgrp libvirt
```

Restart libvirt daemon.

```bash
sudo systemctl restart libvirtd.service
```

## Enable Nested Virtualization (Optional)
Nested Virtualization feature enables you to run Virtual Machines inside a VM. Enable Nested virtualization for kvm_intel by enabling kernel module as shown.

```bash
sudo modprobe -r kvm_intel
sudo modprobe kvm_intel nested=1
```

```bash
cat /sys/module/kvm_intel/parameters/nested
```

To make this configuration persistent,run:

```bash
echo "options kvm-intel nested=1" | sudo tee /etc/modprobe.d/kvm-intel.conf
```

Confirm that Nested Virtualization is set to Yes:

```bash
$ systool -m kvm_intel -v | grep nested
    nested              = "Y"
    nested_early_check  = "N"
$ cat /sys/module/kvm_intel/parameters/nested 
Y
```

# Debug

Error starting domain: Requested operation is not valid: network 'default' is not active

```bash
sudo virsh net-list --all

Name      State      Autostart   Persistent
----------------------------------------------
 default   inactive   no          yes
```

```bash
sudo virsh net-start default
```
