def configure_vm(vm, **opts)

	vm.box = opts.fetch(:box, "estfujitsu/devstack-centos7")
	vm.box_version = opts.fetch(:box_version, "= 1.1")

	# To make minions connect to master, we use static IP network.
	# Since 192.168.121.0/24 and 192.168.122.0/24 are reserved by QEMU in advance, 
	# we use 192.168.123.0/24.
	vm.network 'private_network', ip: opts[:private_ip], libvirt__dhcp_enabled: false, 
		libvirt__network_name: 'devstack-123'

	vm.provider :libvirt do |domain|
		domain.memory = opts.fetch(:memory, 4096)
		domain.cpus = opts.fetch(:cpus, 2)
		domain.nested = true
	end

	# Disable default share, because we dont use it
	vm.synced_folder ".", "/vagrant", disabled: true
end


def execute_stack_sh(vm)
  # Execute stack.sh via shell provisioner because we want to see stdout and because
  # the command is not idempotent.
  vm.provision 'shell', inline: "sudo -i -u stack bash -c 'cd devstack; . /etc/bashrc; ./stack.sh'"
end


def execute_unstack_sh(vm)
  vm.provision 'shell', inline: "sudo -i -u stack bash -c 'cd devstack; ./unstack.sh'"
end


def apply_ansible(vm, playbook, **opts)
	vm.provision :ansible do |ansible|
		ansible.host_key_checking = false
		ansible.playbook = playbook
		ansible.verbose = "v"
	end
end


Vagrant.configure(2) do |config|

	# ---------------------------------------------------------------------------------------------
	#
	# Definition of the VM used for preparing the base image.
	#
	config.vm.define 'base', autostart: false do |base|
	
		configure_vm(base.vm, private_ip: "192.168.123.200", memory: 6*1024, cpus: 4, 
			box: 'centos/7', box_version: '1509.01')
		base.vm.provision 'shell', inline: 'yum update -y'
		base.vm.provision 'reload'
		apply_ansible(base.vm, "./ansible/devstack.pre.yaml")
		execute_stack_sh(base.vm)
		execute_unstack_sh(base.vm)

		# Insecure key was replaced with new one when base VM was provisioned.
		# Restore Vagrant insecure key.
		base.vm.provision 'file', source: './conf/vagrant.pub', 
			destination: '/home/vagrant/.ssh/authorized_keys'
		base.vm.provision 'shell', inline: 'chmod 600 /home/vagrant/.ssh/authorized_keys'

		# Clean up
		base.vm.provision 'shell', inline: <<-'EOS'
			yum clean all
			# VMs created from this base image may not have eth1
			rm -f /etc/sysconfig/network-scripts/ifcfg-eth1
			rm -rf /opt/stack/logs
		EOS
	end

	# ---------------------------------------------------------------------------------------------
	#
	# Definition of the VM for running devstack.
	#
	config.vm.define :devstack, primary: true do |devstack|
	
		devstack_mem = ENV['DEVSTACK_MEM']
		devstack_cpus = ENV['DEVSTACK_CPUS']
		if devstack_mem.nil? || devstack_mem == ""
			 devstack_mem = 6*1024
		end
		if devstack_cpus.nil? || devstack_cpus == ""
			 devstack_cpus = 4
		end

		configure_vm(devstack.vm, private_ip: "192.168.123.100", 
			memory: devstack_mem, cpus: devstack_cpus)

		apply_ansible(devstack.vm, "./ansible/devstack.pre.yaml")
		execute_stack_sh(devstack.vm)
		apply_ansible(devstack.vm, "./ansible/devstack.post.yaml")

		devstack.vm.provision :host_shell do |h|
			h.inline = <<-'EOS'
				ip route show | grep '^172\.24\.4\.0\/24' || (
					echo "Enable routing from your physical host to DevStack's public network 
					  (it may require sudo password)"
					sudo ip route add 172.24.4.0/24 via 192.168.123.100
				)
				echo 'Access Devstack on http://192.168.123.100'
			EOS
		end
	end

end