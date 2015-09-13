---
layout:     post
title:      "How to setup an environment to test self-hosted applications"
subtitle:   ""
date:       2014-12-28 11:56:13
author:     "Nicola Malizia"
tags: ["vagrant"]


twitter-card: true
twitter-image: "data/self-hosted-web-applications.png"

open-graph: true
open-graph-image: "data/self-hosted-web-applications.png"
---

Self-Hosted applications are a class of application (web based or not) that you can run on your home server or VPS or on whatever service that is capable to run the specific project (like Wordpress), most of them are open source and thus free to be used. 

I have recently been attracted by this type of projects for several reasons: I can learn about how to deploy, hack, explore, modify those projects, I can maintain my data out of prying eyes and I must admit it is funny to spend free time studying them. 

At the moment the projects that I use the most daily are: [**selfoss**](selfoss.aditu.de) - the new multipurpose rss reader, live stream, mashup, aggregation web application , [**wallabag**](https://www.wallabag.org/) - a pocket like web application and [**unmark**](https://unmark.it/) - the to do app for bookmarks.

I will review those application on my blog, so stay tuned for further updates!

In this post I will show you how to setup a basic environment on your local machine in order to test an play with those apps.

<div class="panel panel-info">
	<div class="panel-heading">
	    <h3 class="panel-title">Some background knowledge</h3>
	 </div>
	  <div class="panel-body">
 A Virtual Machine is a (guest) computer running inside of your (host) computer.
<br/><br/> VirtualBox “virtualizes” the hardware by tricking virtual servers to think that  they are running on a real hardware.
<br/><br/>
A guest computer can be almost anything - Windows, Mac, Linux or another operating system.
	  </div>
</div>

First of all you need to install [VirtualBox]() and Vagrant. If you are a Linux user, that uses an Ubuntu like distro, you can install VirtualBox via the official repository. 

```bash
sudo apt-get install virtualbox virtualbox-guest-additions-iso
```

To install Vagrant I recommend to manually download and install the *deb* from the project website, at the time I'm writing this blog post the latest Vagrant version is `1.7.1`:

```bash
wget https://dl.bintray.com/mitchellh/vagrant/vagrant_1.7.1_x86_64.deb
sudo dpkg --install vagrant_1.7.1_x86_64.deb
```

To setup a vagrant environment you simply write in your shell:

```bash
vagrant init ubuntu/trusty64
```

For example if you want to set up a Ubuntu Trusty virtual machine. 

You can search [here](https://vagrantcloud.com/boxes/search) for various pre-made and publicly available Vagrant boxes. 

After executing this command a `VagrantFile` file configuration will be created (comments removed):

```ruby
Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"
end

```

We can now:

```bash
vagrant up
```

This command the first time might take some time, Vagrant will download the image of the specified box and it will configure the virtual machine for you. 

After needed step we can now `ssh` into the virtual machine by simply writing :

```bash
vagrant ssh
```

To shutdown the machine we need to write instead:

```bash
vagrant shutdown
```

##Some configuration (optional)
Here some configuration that I apply to VagrantFile in order to make my life easier while testing self-hosted applications.

###Network 
I usually prefer to "expose" the virtual machine to my LAN without spending time configuring a NAT (I'm used to this configuration) because if I need to show something to my friends I can simply temporarly expose the virtual machine over the Internet. 

You can skif this paragraph if you want to let Vagrant do the job for you. 

I also prefer to assign a specific IP address so I can bind it manually to a fake domain using the `/etc/hosts` file. 

Using VirtualBox I usually set a *bridged network interface*, to achieve this with Vagrant, instead of configuring the Virtual Machine and the OS by hand, you can include in your VagrantFile this statement:

```ruby
config.vm.network "public_network", ip: "192.168.0.201"
```

Obviously the IP address depends on your network configuration, if you have like me a home router that provides DHCP this should do the trick. 

The first time you `vagrant up` you will be asked which network interface to use (of you local machine): 

```
==> default: Available bridged network interfaces:
1) wlan0
2) eth0
==> default: When choosing an interface, it is usually the one that is
==> default: being used to connect to the internet.
    default: Which interface should the network bridge to? 1
```

In my case I chose  `1` because my laptop is connected via Wi-Fi to my home router. 

You can always check if everything is ok by typing `ifconfig` in your vagrant box and check if the provided IP is actually set. 

```bash
eth1      Link encap:Ethernet  HWaddr 08:00:27:06:91:5c  
          inet addr:192.168.0.201  Bcast:192.168.0.255  Mask:255.255.255.0
```

If more than one network interface is available on the host machine, Vagrant will ask you to choose which interface the virtual machine should bridge to. A default interface can be specified by adding a :bridge clause to the network definition.

```bash
config.vm.network "public_network", bridge: 'en1: Wi-Fi (AirPort)'
```

The string identifying the desired interface must exactly match the name of an available interface. If it can't be found, Vagrant will ask you to pick from a list of available network interfaces.

Eventually you can also express a specific IP if you don't want to be automatically assigned a free IP from your home router: 

```bash
config.vm.network "public_network", bridge: 'en1: Wi-Fi (AirPort), ip: "192.168.0.201"'
```

##File Sharing
I also prefer to set a shared folder between my local machine and the virtual machine in case I need to quick transfer a file between them (I usually use `sshfs` or `scp`. 

Put this statement in your `VagrantFile`:

```ruby
config.vm.synced_folder ".", "/vagrant"
```

In this way the directory in which resides your `VagrantFile` will be auto-mounted into the file system of your Vagrant box at `/vagrant`.

##Using a VPS
You may already have a VPS or you want to get one in order to host a self-hosted project 24/7 instead of having to `vagrant up` on your local machine and expose your computer on the Internet. Here it is some service that I use and used that provide a good VPS hosting: 

- <u>[DigitalOcean](https://www.digitalocean.com/?refcode=83db418e48e4)</u> (It runs my blog)
- <u>[Hostinger](http://www.hostinger.com/)</u> (I had a game server on it)
- <u>[HapHost](http://haphost.com/free-vps-hosting)</u> (provides free VPS hosting and runs my test projects) 
- <u>[ChunkHost](http://chunkhost.com/r/unnikkedga)</u> (It runs my test projects)
- <u>[Renderhosting](http://account.renderhosting.net/whmcs/aff.php?aff=064)</u> (It runs my self-hosted every day used apps for 1$/mo, promo ended unfortunately)

#Conclusions
Testing self-hosted projects is fun and let you learn a lot of thing, plus you can use everyday and improve productivity. To test one of them you need a proper environment and the right tools, whit this blog post you are ready to host your favourite self-hosted application!