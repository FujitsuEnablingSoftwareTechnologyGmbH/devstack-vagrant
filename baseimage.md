# Creating the Base Image

## Prerequisites
See [here](README.md).

## *CentOS 7* Base Box
Download and convert *CentOS 7* *VirtualBox* box to `libvirt` box. Required only once:
```bash
wget https://atlas.hashicorp.com/relativkreativ/boxes/centos-7-minimal/versions/1.0.3/providers/virtualbox.box
mv virtualbox.box centos-7-minimal.box
sudo apt-get install qemu-utils libvirt-dev
vagrant plugin install vagrant-mutate
vagrant mutate centos-7-minimal.box libvirt
```

## Provisioning
```bash
vagrant up base --provider libvirt # Vagrant will wait for guest IP address
```

Start *Virtual Machine Manager* or `virt-manager`. Open the virtual machine's console, and login using username `root` and password `vagrant`. Create file `/etc/sysconfig/network-scripts/ifcfg-eth0` with this [content](conf/ifcfg-eth0). Execute `ifup eth0` and *Vagrant* will start provisioning.

## Packaging
Once provisioning is finished, power off the VM by `vagrant halt base`. Check volume path by `virsh vol-list --pool default`. In this example, the volume path is `/var/lib/libvirt/images/devstack-libvirt_base.img`.

```
$ virsh vol-list --pool default
 Name                 Path
------------------------------------------------------------------------------
 centos-7-minimal_vagrant_box_image_0.img /var/lib/libvirt/images/centos-7-minimal_vagrant_box_image_0.img
 centos7devstack-0.0.2_vagrant_box_image_0.img /var/lib/libvirt/images/centos7devstack-0.0.2_vagrant_box_image_0.img
 centos7devstack_vagrant_box_image_0.img /var/lib/libvirt/images/centos7devstack_vagrant_box_image_0.img
 devstack-libvirt_base.img /var/lib/libvirt/images/devstack-libvirt_base.img
 teamimage_vagrant_box_image_0.img /var/lib/libvirt/images/teamimage_vagrant_box_image_0.img
 test-nested-kvm_test.img /var/lib/libvirt/images/test-nested-kvm_test.img
```

Download the packaging script [`create_box.sh`](https://github.com/pradels/vagrant-libvirt/blob/master/tools/create_box.sh) and run `sudo ./create_box.sh /var/lib/libvirt/images/devstack-libvirt_base.img`.
