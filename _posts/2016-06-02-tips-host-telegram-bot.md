---
layout:     post
title:      "Tips to host a Telegram Bot"
subtitle:   "Hosting a bot ? It's easier than you think"
date:       2016-06-02 00:00:00
author:     "Nicola Malizia"
tags: ["telegram"]

header-img: "data/cover/telegram-background.jpg"

twitter-card: true
twitter-image: "https://unnikked.ga/data/tip-host-telegram-bot/social.jpg"

open-graph: true
open-graph-image: "https://unnikked.ga/data/tip-host-telegram-bot/social.jpg"
---

I write a lot about Telegram bot lately on my blog, even if I provided some instructions on how to host a bot I see confusion on how to host Telegram bot properly.

In this blog post I'll try to outline some tips on how to host a bot online or on your personal server. Please note that I will not provide custom instructions on how to host X on Y service, I will provide you some informations that will let you understand better the deployment step.

## To webhook or to not webhook

Dude, so much confusion for such a simple concept. It's is better to use a webhook or no ? If I use a webhook how can I get a SSL certificate ? I don't want to pay a SSL certificate just for testing things out! Plz Help me!

The answer is ? It depends.

First thing first let's define once for all what means long polling and what means webhook.

## Long polling

I've already explained it on a [previous blog post](https://unnikked.ga/how-to-create-your-custom-telegram-bot-using-the-long-polling-technique/) but let's [ELI5](https://www.reddit.com/r/explainlikeimfive).

When you use the [getUpdate](https://core.telegram.org/bots/api#getupdates) method your bot is constantly asking to the telegram server if there are any messages for it to process. It's like the bot begs every seconds the API server to feed it with updates.

Putting it simple is like you constantly (like every second) ask to your friend if he is able to hanging out with you until he replies. Fortunately servers do not get upset.

You need SSL, fancy domains setups and gibberish of sort for that ? No, with this method you can host your bot literally everywhere, on any machine that can connect to Internet and haves tools that let you interact with HTTP and strings (yes, because JSON it's just a string).

## Webhook

Oh here the rant comes. Understanding the concept behind webhooks it's even simpler than long polling. If you beg the server when you use the long polling technique with the webhook method you don't do anything, you wait for the server that contacts you.

To put the thing in simple terms: we all have a telephone line (whether it's a home line or a mobile line, and if you are reading this you have it by definition), so it's like you are doing your stuff and someone calls, you pick the phone, reply, close and continue to do your stuff.

_Do i need an SSL certificate ?_ Yes!

_How can I get one for free ?_ The best solution that I can offer you that let's you save time and money is to use [Cloudflare Universal SSL](https://blog.cloudflare.com/introducing-universal-ssl/), you just need use Cloudflare's nameservers to manage a domain with them and get your secure and free SSL endpoint.

_Ok, thanks but I don't have a domain name how do I get one for free ?_ For this I can provide you the register that gave me unnikked.ga, [freenom](http://www.freenom.com/).

# Conclusions

You may have realized that I do not provided you step by step instructions and suggested hosting services this is because those tips can be applied in any case, whether you are hosting it on your raspberry pi or on your VPS or on your Smartphone (yes you can do ;)) or on your IOT fridge.

Are you still confused or you don't know how to proceed on you specific use case ? Drop a comment below and I will be glad to help you.

If you liked this blog post please make sure to share it with your friends and social media.

Furthermore if you want to join my group and be greeted by my bot and the community, [join now](https://telegram.me/joinchat/AEis8D1UWPDa9sdeyzJQ6w)!
