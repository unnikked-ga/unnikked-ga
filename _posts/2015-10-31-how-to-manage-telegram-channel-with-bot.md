---
layout:     post
title:      "How to manage your Telegram channel with your unique bot"
subtitle:   "Bot please spam for me!"
date:       2015-10-31 11:55:00
author:     "Nicola Malizia"
tags: ["ifttt", "telegram"]

header-img: "data/cover/telegram-background.jpg"

twitter-card: true
twitter-image: "https://unnikked.ga/data/social/telegram-channel-schedule-bot.jpg"

open-graph: true
open-graph-image: "https://unnikked.ga/data/social/telegram-channel-schedule-bot.jpg"
---

Recently Telegram added [bot support](https://core.telegram.org/bots/api#recent-changes) for the new [channel](https://telegram.org/blog/channels) feature. Since I manage [my own channel](http://unnikked.ml/1OOZQa3) I tought I could try to find some automating mechanism to automate the posting process - ideally scheduling it. Here it is what I got.

## Requisites

- A channel where you are administrator
- A bot - you can check out my [introduction post](/getting-started-with-telegram-bots) to learn how to create a new bot
- An IFTTT account
- A Google account (for Google Calendar)

Yes, this process will not require you to write a single line of code!

## The concept

The idea behind this is to use you Google Calendar to schedule the post you want to share with your followers, thanks to a special recipe you can check any new post tagged fot example `##telegram-channel` and make a `HTTP` request to the bot platform server via the `Maker` IFTTT channel.

<p align="center"><a href="https://ifttt.com/view_embed_recipe/338138-schedule-post-for-a-telegram-channel-using-your-own-bot" target = "_blank" class="embed_recipe embed_recipe-l_56" id= "embed_recipe-338138"><img src= 'https://ifttt.com/recipe_embed_img/338138' alt="IFTTT Recipe: Schedule post for a Telegram channel using your own bot! connects google-calendar to maker" width="370px" style="max-width:100%"/></a><script async type="text/javascript" src= "//ifttt.com/assets/embed_recipe.js"></script></p>

### How to add the bot as administrator to my channel?

The process it's simple, you first need to go to your info channel and click on `Administrators`

![Telegram Channel Info](https://unnikked.ga/data/telegram-channel-info.jpg)

You will see the administrator list

![Telegram Channel Admin List](https://unnikked.ga/data/telegram-channel-admin-list.jpg)

By clicking on `Add administrator` you will be prompt with a search section, search your bot by using the format `@yourbotname`

![Telegram Channel Add Bot](https://unnikked.ga/data/telegram-channel-add-bot.jpg)

> This process is made on the official Android client, the process may vary if you use an alternative client

### How do I use the IFTTT recipe ?

Click on `Add` here below.

<p align="center"><a href="https://ifttt.com/view_embed_recipe/338138-schedule-post-for-a-telegram-channel-using-your-own-bot" target = "_blank" class="embed_recipe embed_recipe-l_56" id= "embed_recipe-338138"><img src= 'https://ifttt.com/recipe_embed_img/338138' alt="IFTTT Recipe: Schedule post for a Telegram channel using your own bot! connects google-calendar to maker" width="370px" style="max-width:100%"/></a><script async type="text/javascript" src= "//ifttt.com/assets/embed_recipe.js"></script></p>

You will see a dialog box like below (you may be asked to activate some channel first).

![Telegram Channel IFTTT Recipe](https://unnikked.ga/data/telegram-channel-ifttt-recipe.png)

Where the customizable field means:

- `Keyword or phrase` - the keyword you want to use to trigger your recipe
- `URL` - substitute at `[bot_key]` your private api token got by talking with [@BotFather](https://telegram.me/botfather)
- `Body` - this is the JSON formatted payload request, edit it carefully.
    - `chat_id` - insert here your channel name in the format `@channelname`, it is the public name that you gave to your channel

### How do I schedule now posts?

Now you simply need to log in to your Google Calendar and create a new event!

![Telegram Channel Google Calendar](https://unnikked.ga/data/telegram-channel-google-calendar.png)

In the `description` box you will put your content.

## Conclusion

Now you have a fully fledged content scheduler for your telegram channel without writing a single line of code, how cool is that?

If you manage more that one channel this method is scalable, you only need to create several recipes and channel the `Keyword of phrase` field to your custom tag, that's what triggers the specific recipe on IFTTT!

Have you any suggestion ? Please let me know in the comment bot and if you have not done it yet subscribe to my [Telegram channel](http://unnikked.ml/1OOZQa3)!
