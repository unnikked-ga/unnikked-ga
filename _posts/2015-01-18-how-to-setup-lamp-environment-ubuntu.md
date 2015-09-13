---
layout:     post
title:      "How to setup a LAMP environment on Ubuntu"
subtitle:   "Easily deploy your websites"
date:       2015-01-18 13:05:55
author:     "Nicola Malizia"
tags: ["self-hosted", "vps"]

twitter-card: true
twitter-image: "data/how-to-setup-a-lamp-environment.png"

open-graph: true
open-graph-image: "data/how-to-setup-a-lamp-environment.png"
---

If you want to test a self-hosted application (developed in PHP/MySQL) or you want to run your own website on your VPS or simply test it locally on your virtual machine you need to configure first a **LAMP** environment. 

LAMP stands for: 

- **L**inux
- **A**pache
- **M**ySQL
- **P**hp

In future blog posts I'm going to review some <a href="environment-testing-self-hosted-projects" target="_blank">self-hosted</a> applications that I use most and for that reason I will provide a step by step tutorial on how to set up a basic LAMP environment on a Ubunutu like distro.

Before to proceed update your repository list:

```bash
sudo apt-get update
```

## Apache

Apache is the first software you need to install on your machine. 

**Apache HTTP Server**, or commonly *Apache* it's the name of one of the most used modular web server open souce that it is capable to run on Unix/Linux like and Microsoft operating systems. 

To install apache: 

```bash
sudo apt-get install apache2
```

After finishing the installation the web server is automatically started, you can check if everything is ok by visiting your server using it's IP or the domain pointed. 

<img class="img-responsive" src="data/apache-it-works.png" alt="Apache It Works">

<blockquote>
On Ubuntu you can handle services using the `service` command

```bash
sudo service name_of_the_service command
```

Where `name_of_the_service` is the name of the service to manage, for example `apache2`. 

`command` is the command that the service have to execute, for example `start`, `stop`, `restart`.
</blockquote>

## MySQL

MySQL it's a *Relational Database Management System* (**RDBMS**) composed by a command line user interface and a server. 

It is usefull to store and retrieve the data produced by the application. 

Using the following command you install MySQL and some needed libraries 

```bash
sudo apt-get install mysql-server php5-mysql libapache2-mod-auth-mysql
```

During the installation you will be asked for choosing a password for the `root` user of MySQL

<img class="img-responsive" src="data/root-mysql-password.png" alt="MySQL root password">

We can now install a test database using:

```bash
sudo mysql_install_db
```

We can declare installed correctly MySQL, but it is not ready for a production environment so let's execute:

```bash
sudo /usr/bin/mysql_secure_installation
```

in order _secure_ our MySQL installation. 

After provided the `root` MySQL password we will be asked for (here it is what I reply in my case):

```

- You already have a root password set, so you can safely answer ‘n’. N
- By default, a MySQL installation has an anonymous user, allowing anyone to log into MySQL without having to have a user account created for them. This is intended only for testing, and to make the installation go a bit smoother. You should remove them before moving into a production environment. Y
- Normally, root should only be allowed to connect from ‘localhost’. This ensures that someone cannot guess at the root password from the network. Y
- By default, MySQL comes with a database named ‘test’ that anyone can access. This is also intended only for testing, and should be removed before moving into a production environment. Y
- Reloading the privilege tables will ensure that all changes made so far will take effect immediately. Y

```

We can now access into MySQL command line user interface by typing:

```bash
sudo mysql -u root -p
```

## PHP

PHP is a server-side scripting language designed for web development but also used as a general-purpose programming language.

To install it: 

```bash
sudo apt-get install php5 libapache2-mod-php5
```

## PhpMyAdmin

For the new user you may want to manage easily your MySQL databases, here it comes handy `phpMyAdmin`. It is a software written in PHP that will let you easily manage your databases from your browser, altought I recommend you to spend time to learn the command line interface. 

To install it symply type: 

```bash
sudo apt-get install phpmyadmin
```

After you installed it you can access it from your server domain name or server ip /phpmyadmin

```
server_ip/phpmyadmin
```

# Conclusions

We have installed a basic LAMP environment, you may want now to install further PHP modules for your applications or apache mods, it's up to your application. 

You may also want to install an FTP server in order to access easily to your files but I strongly recommend you to use at least SFTP.