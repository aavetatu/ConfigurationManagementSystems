# Homework 6

## x)

https://terokarvinen.com/2017/04/11/vagrant-revisited-install-boot-new-virtual-machine-in-31-seconds/

	- $ vagrant init debian/bullseye64
	- $ vagrant up
	- $ vagrant ssh

https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/

	- $ mkdir twohost/; cd twohost/
	- $ nano Vagrantfile
	- $ vagrant ssh t001
	- vagrant@t001$ ping -c 1 192.168.88.102
	- vagrant@t001$ ping -c 1 8.8.8.8 # Google nameserver
	- vagrant@t001$ exit

https://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/

	- slave$ sudoedit /etc/salt/minion
	- slave$ sudo systemctl restart salt-minion.service
	- master$ sudo salt-key -A
	- master$ sudo salt '*' cmd.run 'whoami'

## a) Install virtual machine with vagrant

I had installed vagrant on my windows machine in advance from https://developer.hashicorp.com/vagrant/downloads

I installed a new virtual machine with vagrant

	C:\Windows\system32>vagrant init debian/bullseye64
	C:\Windows\system32>vagrant up
	C:\Windows\system32>vagrant ssh
	vagrant@bullseye:~$ whoami
	vagrant
	vagrant@bullseye:~$ exit
	logout
	Connection to 127.0.0.1 closed.
	
## b) Install two virtual machines in the same network with vagrant

I made a new init

	C:\Temp>vagrant init
	
I opened C:\Temp in file explorer and edited Vagrantfile with notepad++

	C:\Temp>type Vagrantfile
	# -*- mode: ruby -*-
	# vi: set ft=ruby :
	# Copyright 2019-2021 Tero Karvinen http://TeroKarvinen.com

	$tscript = <<TSCRIPT
	set -o verbose
	apt-get update
	apt-get -y install tree
	echo "Done - set up test environment - https://terokarvinen.com/search/?q=vagrant"
	TSCRIPT

	Vagrant.configure("2") do |config|
			config.vm.synced_folder ".", "/vagrant", disabled: true
			config.vm.synced_folder "shared/", "/home/vagrant/shared", create: true
			config.vm.provision "shell", inline: $tscript
			config.vm.box = "debian/bullseye64"

			config.vm.define "t001" do |t001|
					t001.vm.hostname = "isanta"
					t001.vm.network "private_network", ip: "192.168.88.101"
			end

			config.vm.define "t002", primary: true do |t002|
					t002.vm.hostname = "renki1"
					t002.vm.network "private_network", ip: "192.168.88.102"
			end

	end
	
I created two new virtual machines

	C:\Temp>vagrant up
	
I opened the first one with ssh 

	C:\Temp>vagrant ssh t001
	vagrant@isanta:~$ whoami
	vagrant
	vagrant@isanta:~$ hostname -I
	10.0.2.15 192.168.88.101
	vagrant@isanta:~$ exit
	logout
	Connection to 127.0.0.1 closed.

I opened the second one with ssh and tested to ping the first machine

	C:\Temp>vagrant ssh t002
	vagrant@renki1:~$ whoami
	vagrant
	vagrant@renki1:~$ ping 192.168.88.101
	PING 192.168.88.101 (192.168.88.101) 56(84) bytes of data.
	64 bytes from 192.168.88.101: icmp_seq=1 ttl=64 time=0.334 ms

## c) Implement the Salt master-slave architecture over the network

In this exercise the master promt below is "isanta$" and the minion promt is "renki1$"

First I needed to install salt-master to one of the virtual machines

	C:\Temp>vagrant ssh t001
	isanta:~$ sudo apt-get update
	isanta:~$ sudo apt-get -y install salt-master
	isanta:~$ hostname -I
	10.0.2.15 192.168.88.101
	isanta:~$ exit
	
After that I needed to isntall salt-minion to the other machine

	C:\Temp>vagrant ssh t002
	renki1:~$ sudo apt-get update
	renki1:~$ sudo apt-get -y install salt-minion
	
Next I specified masters ip-address for minion to use

	renki1:~$ sudoedit /etc/salt/minion
	renki1:~$ cat /etc/salt/minion
	master: 192.168.88.101
	id: renki1
	
And I restarted the service

	renki1:~$ sudo systemctl restart salt-minion.service
	renki1:~$ exit
	
I connected again to the master to accept the slave key

	C:\Temp>vagrant ssh t001
	isanta:~$ sudo salt-key -A
	The following keys are going to be accepted:
	Unaccepted Keys:
	renki1
	Proceed? [n/Y] Y
	Key for minion renki1 accepted.

And tested if the connection works

	isanta:~$ sudo salt '*' cmd.run 'whoami'
	renki1:
		root
	isanta:~$ sudo salt '*' cmd.run 'hostname -I'
	renki1:
		10.0.2.15 192.168.88.102
		
Sources:

https://terokarvinen.com/2022/palvelinten-hallinta-2022p2/

https://shellgeek.com/use-cat-equivalent-type-command-in-windows/

https://www.sitepoint.com/getting-started-vagrant-windows/
