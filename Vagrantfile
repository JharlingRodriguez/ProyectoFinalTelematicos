# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

	if Vagrant.has_plugin? "vagrant-vbguest"
		config.vbguest.no_install = true
		config.vbguest.auto_update = false
		config.vbguest.no_remote = true
	end

 		
	config.vm.define :servidor1 do |servidor1|
 		servidor1.vm.box = "bento/ubuntu-22.04"
 		servidor1.vm.network :private_network, ip: "192.168.50.10"
 		servidor1.vm.hostname = "servidor1"
		servidor1.vm.boot_timeout = 600
 	end
	
	config.vm.define :servidor2 do |servidor2|
 		servidor2.vm.box = "bento/ubuntu-22.04"
 		servidor2.vm.network :private_network, ip: "192.168.50.20"
 		servidor2.vm.hostname = "servidor2"
		servidor2.vm.boot_timeout = 600
 	end
 	
	config.vm.define :servidor3 do |servidor3|
 		servidor3.vm.box = "bento/ubuntu-22.04"
 		servidor3.vm.network :private_network, ip: "192.168.50.30"
 		servidor3.vm.hostname = "servidor3"
		servidor3.vm.boot_timeout = 600
 	end

	config.vm.define :balanceador do |balanceador|
 		balanceador.vm.box = "bento/ubuntu-22.04"
 		balanceador.vm.network :private_network, ip: "192.168.50.40"
 		balanceador.vm.hostname = "balanceador"
		balanceador.vm.boot_timeout = 600
 	end
end