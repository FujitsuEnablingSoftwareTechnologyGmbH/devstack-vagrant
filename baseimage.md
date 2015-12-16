# Creating the Base Image

To make DevStack provisioning faster, it is better to create a base image.

## Prerequisites
See [here](README.md).

## Provisioning
```bash
vagrant up base --provider libvirt
```

## Packaging
Once provisioning is finished, power off the VM by `vagrant halt base`. Check volume path by `virsh vol-list --pool default`. In this example, the volume path is `/var/lib/libvirt/images/devstack_base.img`.

```
$ virsh vol-list --pool default
 Name                 Path
------------------------------------------------------------------------------
 centos-7-minimal_vagrant_box_image_0.img /var/lib/libvirt/images/centos-7-minimal_vagrant_box_image_0.img
 centos7devstack-0.0.2_vagrant_box_image_0.img /var/lib/libvirt/images/centos7devstack-0.0.2_vagrant_box_image_0.img
 centos7devstack_vagrant_box_image_0.img /var/lib/libvirt/images/centos7devstack_vagrant_box_image_0.img
 devstack_base.img /var/lib/libvirt/images/devstack_base.img
 teamimage_vagrant_box_image_0.img /var/lib/libvirt/images/teamimage_vagrant_box_image_0.img
 test-nested-kvm_test.img /var/lib/libvirt/images/test-nested-kvm_test.img
```

Download the packaging script [`create_box.sh`](https://github.com/pradels/vagrant-libvirt/blob/master/tools/create_box.sh) and run `sudo ./create_box.sh /var/lib/libvirt/images/devstack_base.img`.
Upload `devstack_base.box` file to your Vagrant repository.
