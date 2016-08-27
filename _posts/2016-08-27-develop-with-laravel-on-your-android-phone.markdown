---
layout: post
title: "Develop With Laravel On Your Android Phone"
date: "2016-08-27 15:08:27 +0200"
subtitle:   "Seriously? Why? Because I can."
author:     "Nicola Malizia"
tags: ["android", "php"]

twitter-card: true
twitter-image: "https://unnikked.ga/data/develop-with-laravel-on-your-android-phone/social-cover.png"

open-graph: true
open-graph-image: "https://unnikked.ga/data/develop-with-laravel-on-your-android-phone/social-cover.png"
---

Have you an old smartphone that is laying around and you don't know what to do with ? I know that feel!

On a moment of laziness I discovered [Termux](https://termux.com) and an idea sparkled in my head.

> Termux is a terminal emulator and Linux environment bringing powerful terminal access to Android.

I want to use my old smartphone as a web server for Laravel!

![Shut up and install laravel](/data/develop-with-laravel-on-your-android-phone/shut-up-install-laravel.jpg)

## Installing packages

First of all install Termux from the [Google Play Store](https://play.google.com/store/apps/details?id=com.termux) or [F-Droid](https://f-droid.org/repository/browse/?fdfilter=termux&fdid=com.termux). Open the app and you will see a classic shell promt. Let's install some basic packages.

To install PHP

```
apt install php
```

> At the moment of writing is only available php5.6.25

To install sqlite (so we can test if migrations works)

```
apt install sqlite
```

![apt install sqlite](/data/develop-with-laravel-on-your-android-phone/apt-install-sqlite.jpg)

## Let's compose an application

To install composer, follow the instructions provided by the [official website](https://getcomposer.org/download/).

```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('SHA384', 'composer-setup.php') === 'e115a8dc7871f15d853148a7fbac7da27d6c0030b848d9b3dc09e2a0388afed865e6a3d6b3c0fad45c48e2b5fc1196ae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```

We can try if it works by running `php composer.phar`

![Composer](/data/develop-with-laravel-on-your-android-phone/composer.jpg)

## Installing Laravel

I don't like to write the obvious and waste your time so here it is the command.

```
php composer.phar create-project --prefer-dist laravel/laravel blog
```
It will take some time.

![Install Laravel](/data/develop-with-laravel-on-your-android-phone/install-laravel.jpg)

## Checking everything is working

So let's face the thruth and see if everything works. First `cd blog` and then unleash a

```
php artisan serve
```

![It works](/data/develop-with-laravel-on-your-android-phone/it-works.jpg)

<p align="center"><img src="/data/develop-with-laravel-on-your-android-phone/it-works-obama.jpg"></p>

## What about migrations ?

Yea Yea I know, you are curious about if you can migrate your migrations ? Yes you can! But only if you use sqlite.

First of all edit `.env` file as follow.

```
vim .env
```

![Database Connection](/data/develop-with-laravel-on-your-android-phone/db-connection.jpg)


> **Don't you have `vim`?** Don't worry. Run `apt install vim` and you are ready to go.
> **Can't you use `vim`?** Use the editor of your choice.

Now we can `touch` our database.

```
touch database/database.sqlite
```

Now that we have our loved database we can scaffold the auth system.

```
php artisan make:auth
```

And now we can migrate!

```
php artisan migrate
```

![Migrate](/data/develop-with-laravel-on-your-android-phone/migrate.jpg)

<p align="center"><img src="/data/develop-with-laravel-on-your-android-phone/migrations.jpg"></p>


Once we `php artisan serve` again we can see the registration form in all it's beauty.

![Register](/data/develop-with-laravel-on-your-android-phone/register.jpg)

## Wait old boy I wanna access it from my desktop

My dear friend, nothing is more simpler that that.

First of all get your phone local ip through the old and fashion

```
ifconfig
```
![ifconfig](/data/develop-with-laravel-on-your-android-phone/ifconfig.jpg)

Now remember the local ip of you phone and `serve` it on this address.

```
php artisan serve --host=192.168.0.103
```

> Please note that `192.168.0.103` vary depending on your network connection. No copy paste allowed here :)

![It works](/data/develop-with-laravel-on-your-android-phone/it-works-on-desktop.png)

## Hehe I have an ever trickier question. I want to expose it on the internet, I bet you can't do it!

Oh my fried, of course you can do it. Do you know that exist a service called [ngrok](https://ngrok.com/)?

> **Secure tunnels to localhost**
”I want to expose a local server behind a NAT or firewall to the internet.”

Once you have installed ngrok on you local machine (not your phone) you can run.

```
ngrok http 192.168.0.103:8000
```

![Ngrok](/data/develop-with-laravel-on-your-android-phone/ngrok.png)

Happy Laravel!

# Conclusions

Well if you read till here you already noted that I tried to use a funny language here. I'm aware that this is just playing and you will not get nothing productive out of it (do you?).

It is in my nature to explore new things and try to push what I have in my hands to the extreme. This is what I like from technology, the possibilites that it opens to you, even if it silly or not useful at all.

Seeing that a little box that holds in your pocket is capable of doing complex stuff like this it just lights me up.

I don't know if this article will be valuable to you but I hope you enjoyed my journey while reading it.

Have you suggestions? Wanna whare your story with termux? Drop a comment below or join [my group on Telegram](https://telegram.me/joinchat/AEis8D1UWPDa9sdeyzJQ6w).
