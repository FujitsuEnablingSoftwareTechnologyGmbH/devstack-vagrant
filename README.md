# *devstack* with *Vagrant* and `libvirt`


## What's This About?
The purpose of this project is to easily fire up a VM with a running *OpenStack* inside, ready to accept deployments of *Kubernetes* clusters. *OpenStack* is provisioned inside the VM via *devstack*. The VM itself is created with *Vagrant* and `libvirt`.


## Minimum Requirements
* Ram: 8GB
* Disc: 50GB
* CPUs: 4 Cores


## Preparation of *Ubuntu 14.04.3* Host

### *Virtual machine manager* and `libvirt`
```bash
sudo apt-get install virt-manager libvirt-bin qemu qemu-system-x86
```

### *Vagrant* 1.7+
Download from [website](http://www.vagrantup.com/downloads.html), then:
```bash
sudo dpkg -i vagrant_1.7.2_x86_64.deb
sudo apt-get install libvirt-dev
vagrant plugin install vagrant-libvirt
vagrant plugin install vagrant-host-shell
vagrant plugin install vagrant-reload
```

### *Ansible* 1.9+
```
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
```

Reboot system.


## Preparation of *CentOS 7* Host

Add *epel* repository:
```
sudo yum install epel-release
```

### *Virtual machine manager* and `libvirt`
```bash
sudo yum install virt-manager libvirt qemu qemu-system-x86
```

### Possibly Required
By default *CentOS* is using *SELinux*, so additional configuration is required to work with `libvirt`.

#### Setup SSH access for `libvirt`
1. Create the user group:
```bash
sudo groupadd libvirt
```

2. Update the `libvirt` configuration in `/etc/libvirt/libvirtd.conf`. Uncomment these two lines:
```bash
#unix_sock_group = "libvirt"
#unix_sock_rw_perms = "0770"
```

3. Restart the service:
```bash
sudo service libvirtd restart
```

4. Manage group membership - Add the desired users to this user group: 
```bash
sudo usermod -G libvirt -a *username*
```


#### Configure management access via PolicyKit
Create a new policy file `/etc/polkit-1/localauthority/50-local.d/50-de.est.libvirt-remote-access.pkla` with the following content:
```
[libvirt Management Access]
Identity=unix-group:libvirt
Action=org.libvirt.unix.manage
ResultAny=yes
ResultInactive=yes
ResultActive=yes
```

### Install Vagrant 1.7+
Download from [website](http://www.vagrantup.com/downloads.html), then:
```
sudo yum install vagrant_1.7.3_x86_64.rpm
sudo yum install gcc libvirt-devel
vagrant plugin install vagrant-libvirt
vagrant plugin install vagrant-host-shell
vagrant plugin install vagrant-reload
```

### Ansible 1.9+
```
sudo yum install ansible
```

Reboot the system


## Gotchas

*Vagrant* usually stores machine images underneath `/var`, so please make sure that you have at least 30GB of free space on the file system that is hosting `/var`. 


## Ready... Set... Go...

Start the provisioning of *devstack* in the VM with:
```bash
vagrant up --provider libvirt
```

### Environment Variables
- `DEVSTACK_MEM` how much RAM to give to the VM, in MB, defaults to `6144`MB
- `DEVSTACK_CPUS` how many CPUs to give to the VM, in MB, defaults to `4`

So for custom memory and CPUs, you could do:
```bash
DEVSTACK_MEM=8192 DEVSTACK_CPUS=6 vagrant up --provider libvirt
```

### Basic Usage
*Horizon* is running on port `80`, user `demo` or `admin`, password `secretsecret`. To check on the VM's IP address, you can do:
```bash
vagrant ssh
ifconfig eth0
```

After restarting the physical host, the VM unfortunately must be provisioned again:
```bash
vagrant destroy
vagrant up
```
