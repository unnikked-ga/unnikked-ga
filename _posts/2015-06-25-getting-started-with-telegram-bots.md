---
layout:     post
title:      "Getting started with Telegram bots"
subtitle:   "I will do everything you want"
date:       2015-06-25 22:10:00
author:     "Nicola Malizia"
tags: ["telegram"]

twitter-card: true
twitter-image: "http://i.imgur.com/jzuVTTi.png"

open-graph: true
open-graph-image: "http://i.imgur.com/jzuVTTi.png"
---

On 24 June Telegram released the new [Bot platform](https://telegram.org/blog/bot-revolution). You can now create or use existing bots to enhance your Telegram experience. 

The new Bot platform is shipped with a fancy HTTP API mechanism, so building a custom Bot is a breeze. 

I was not able to wait more and after having played with my friends with the official bots I started to figure out how to use the APIs provided. 

Since the interaction is based purely on HTTP requests it was easy to me to get start easily using only my command line and some curl commands. 

## BotFather

The bot BotFather as the name suggests is the "father" of all bots (say again bot please), talking to this fancy program will let you create new children alongside an API token. 

To create a new bot simply send a message to [@BotFather](https://telegram.me/botfather). 

```
You: /newbot

BotFather: Alright, a new bot. How are we going to call it? Please choose a name for your bot.

You: YourFancyBot

BotFather: Done! Congratulations on your new bot. You will find it at telegram.me/YourFancyBot. You can now add a description, about section and profile picture for your bot, see /help for a list of commands.

Use this token to access the HTTP API:
75761921:AAF-VKn46WCNEfr9MHmFr9gyNULkT2Go13k

For a description of the Bot API, see this page: https://core.telegram.org/bots/api

```

Where obviously `YourFancyBot` is the name you want to give to your bot. 

## Talking to your bot

Once you created a bot and obtained the token you can start to interact with your bot. 

<blockquote>Alongside curl commands I will use also the utility <a href="http://stedolan.github.io/jq/">jq</a> so the returned JSON is pretty printed.</blockquote>

Every request is formatted using this URL schema: 

```
https://api.telegram.org/bot<token>/METHOD_NAME
```

Where `<token>` is the piece of information received by @BotFather and `METHOD_NAME` is the method you want to perform on the bot.

You can view a list of method names on the official [api documentation](https://core.telegram.org/bots/api#available-methods).

### getMe

Let's say we want to retrieve basic information about the newly created bot, we need to use the `getMe` method. 

```
curl -s \
-X POST \
https://api.telegram.org/bot<token>/getMe \
| jq .
```

The response will be like the following

```json
{
  "result": {
    "username": "YourFancyBot",
    "first_name": "YourFancyBot",
    "id": 1337
  },
  "ok": true
}
```

## Getting chat input

There are two main ways to obtain input from your bot, i.e. when the user sends messages to it. 

I will not cover it thoroughly here because for our example we would need to setup some sort of web server and implement a bit complex mechanism to parse and execute commands sent to the bot. 

Without spending much words there are two methods to retrieve messages sent to the bot: 

- `getUpdates`
- `setWebhook`

With `getUpdates` you can setup a pooling mechanism to pool data from the bot and process it, we will use this method in our examples (only to obtain a `chat_id`). 

With `setWebhook` we can setup a "callback" url where the bot will push everything received into a conversation, personally I'd rather prefer this method. 

If you want to know how webhooks work check out the Wikipedia [entry](https://en.wikipedia.org/wiki/Webhook). 

###sendMessage

Let's say we want to send a message to yourself, but we do not want to set up a web server since we are only testing the APIs. 

In order to use the `sendMessage` method we need to use the proper `chat_id`. 

First things first let's send the `/start` command to our bot via a Telegram client. 

After sent this command let's perform a `getUpdates` commands. 

```
curl -s \
-X POST \
https://api.telegram.org/bot<token>/getUpdates \
| jq .
```

The response will be like the following

```json
{
  "result": [
    {
      "message": {
        "text": "/start",
        "date": 1435176541,
        "chat": {
          "username": "yourusername",
          "first_name": "yourfirstname",
          "id": 65535
        },
        "from": {
          "username": "yourusername",
          "first_name": "yourfirstname",
          "id": 65535
        },
        "message_id": 1
      },
      "update_id": 714636917
    }
   ],
  "ok": true
}
```

We are interested in the property `result.message[0].chat.id`, save this information elsewhere. 

<blockquote>Please note that this is only an example, you may want to set up some automatism to handle those informations</blockquote>

Now how we can send a message ? It's simple let's check out this snippet. 

```bash
curl -s \
-X POST \
https://api.telegram.org/bot<token>/sendMessage \
-d text="A message from your bot" \
-d chat_id=65535 \
| jq .
```

Where `chat_id` is the piece of information saved before.

<p align="center"><img class="img-responsive" src="http://i.imgur.com/HzQtZ3t.png"></p>

Pretty cool huh!

###sendPhoto

As a last example let's say we want to send a photo from your computer to your telegram conversation. 

This is simple as the last command, we need to use instead this time the `sendPhoto` method. 

```bash
curl -s \
-X POST \
https://api.telegram.org/bot<token>/sendPhoto \
-F chat_id=65535 \
-F photo="@path/to/your/image" \
| jq .
```

Where `path/to/your/image` is the path to an image file. The response will be like. 

```JSON
{
  "result": {
    "photo": [
      {
        "height": 28,
        "width": 90,
        "file_size": 827,
        "file_id": "AgADBAADracxGwEJhAR6IDIHWyPwwsBxYzAABJYTyFJznz6YgSgAAgI"
      },
      {
        "height": 98,
        "width": 320,
        "file_size": 13116,
        "file_id": "AgADBAADracxGwEJhAR6IDIHWyPwwsBxYzAABIy6XcxfOPhCgCgAAgI"
      },
      {
        "height": 205,
        "width": 666,
        "file_size": 42544,
        "file_id": "AgADBAADracxGwEJhAR6IDIHWyPwwsBxYzAABEwmPSHAVm5MfygAAgI"
      },
      {
        "height": 0,
        "width": 0,
        "file_size": 31908,
        "file_id": "AgAD_v___62nMRsBCYQEeiAyB1sj8MIAFAI"
      }
    ],
    "date": 1435261824,
    "chat": {
      "username": "yourusername",
      "first_name": "yourfirstname",
      "id": 65535
    },
    "from": {
      "username": "YourFancyBot",
      "first_name": "YourFancyBot",
      "id": 1337
    },
    "message_id": 41
  },
  "ok": true
}
```
<p align="center"><img class="img-responsive" src="http://i.imgur.com/LlW0asc.png"></p>

## Conclusion

The new Bot platform in my opinion is a cool feature added to  Telegram, it could be useful not only to have some good times with your friends (see [@ImageBot](https://telegram.me/ImageBot)) but also to improve the productivity of a team since you can potentially send commands to performs actions via your Telegram client or receive some sort of push notifications. 

I know that there are also alternatives, maybe better of this (the right tool for the right job) but for me, in this moment, this is a great feature of this instant messaging platform. 

If you liked this content please drop a comment below, I may want to publish new how tos about this platform like setting up a basic bot to process commands or integrating it with other systems. 
