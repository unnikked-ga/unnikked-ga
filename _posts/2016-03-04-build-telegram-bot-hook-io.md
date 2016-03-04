---
layout:     post
title:      "How to easily build a Telegram Bot with Hook.io"
subtitle:   "An open source platform to host http microservices"
date:       2016-02-04 13:35:00
author:     "Nicola Malizia"
tags: ["telegram", "javascript", "hookio"]

header-img: "data/cover/telegram-background.jpg"

twitter-card: true
twitter-image: "https://lh3.googleusercontent.com/-2mwS3AIcK8s/Vtl5ywHTncI/AAAAAAAAAP4/BPubZdQoX_g/s0/build-telegram-bot-hook-io.jpg"

open-graph: true
open-graph-image: "https://lh3.googleusercontent.com/-2mwS3AIcK8s/Vtl5ywHTncI/AAAAAAAAAP4/BPubZdQoX_g/s0/build-telegram-bot-hook-io.jpg"

---

Recently I've been searching a service that let me host the code for a bot without setting up too much programs. 

This is because I want to provide you simple examples that you can kick off in no time, I know that feeling when you can do something quickly. 

So searching after searching I found [hook.io](https://hook.io). This service provides an infrastructure that let's you deploy HTTP microservices effortlessly and in a serverless fashion like AWS Lambda.  

Basically you can script HTTP requests without deploying an entire web stack (as LAMP). I want to call it HaaS (WebHook as a Service). 

Hook.io supports lot's of programming languages, even Bash (huge fan of it for command line scripts, although is in a experimental state). 

But what interested me is that you can use NPM packages in your scripts, despite you cannot access to the entire NPM registry. 

Since there are millions of ways to develop a bot, I'll decide to show only how things should be done according to me without focusing emphatically to the implementation, hook.io will be our platform of reference to run instantly the examples. 

In this tutorial I'll show you just a little setting up tutorial and develop a simple and fancy echo bot. 

## Requisites 
<ul>
	<li>Bot API Token, you can grab it by talking with <a href="https://telegram.me/botfather">@BotFather</a> as I described in my first tutorial. </li>
	<li>Hook.io account.</li>
</ul>

## Create a new hook

To create a new hook simply visit the [Create Hook](https://hook.io/new) section. 

![Create Hook](https://lh3.googleusercontent.com/-4NYo2qAPu-E/Vtlx64AM_-I/AAAAAAAAAOs/lCxiv6Q2O0I/s0/Schermata+del+2016-03-04+12%253A30%253A28.png "create hook")

We will only need the **Name** filed, choose here the name of the hook. For example `echo-bot`

With the free account hooks are default set to public. 

![hook source](https://lh3.googleusercontent.com/-NG5Cc8A9ohQ/VtlyXE9cTkI/AAAAAAAAAO0/B11oBRFqrPU/s0/hook+source.png "hook source.png")

On the **Hook Source** panel you can choose your programming language and you can provide here the source code of the webhook (the source code of the bot in our case). 

For this example we provide the source directly in the code editor. Paste here the following code: 

```javascript
module['exports'] = function echoBot (hook) {
	var request = require('request');
  	request
      	.post('https://api.telegram.org/bot' + hook.env.echo_bot_key + '/sendMessage')
    	.form({
      		"chat_id": hook.params.message.chat.id,
      		"text": hook.params.message.text
    	});
};
```

You can grab more information about the `hook` object on the [official documentation](https://hook.io/docs#hook), in our case we access the `params` property where the Telegram Bot Platform will POST updates. The request is already deselialized so we can access them directly. 

The `hook.env` object holds environment information needed for the specific hook. This is extremely useful so you don't have to hard code the bot token directly into the source code. 

You can hit `Create new Hook` button now. 

## Setting up environment variable and bot webhook

To set up the API Token simply visit the [environment section](https://hook.io/env) on hook.io and set up a key as follow. 

![setting environment](https://lh3.googleusercontent.com/-r_SqkVZgE_4/Vtl0n3Tuq5I/AAAAAAAAAPM/R5lxZ7Xxvew/s0/setting+environment.png "setting environment.png")

Make sure you access the right key in your code. In this example `echo_bot_key` is reflected into the source code `hook.env.echo_boy_key`. This is crucial or the hook will not work. 

> Remember to put here the token you generated via @botfather

To setup the webhook for the bot you simply open your browser and construct an URL like the following: 

`https://api.telegram.org/bot<TOKEN>/setWebhook?url=https://hook.io/<hook-user>/<hook-name>`

Where: 

<ul>
	<li><strong>TOKEN</strong> is the token you generated via @botfather</li>
	<li><strong>hook-user</strong> is your hook username</li>
	<li><strong>hook-name</strong> is the hook name you choose while creating a new hook</li>
</ul>

After you hit enter you will see a response like the following: 

```
{"ok":true,"result":true,"description":"Webhook was set"}
```

## Test your bot

Simply open a conversation to your bot and it will reply you back like a parrot. 

![echo bot](https://lh3.googleusercontent.com/-9zEpFnBfXpk/Vtl3P6MYExI/AAAAAAAAAPk/FqwX5tI6UDw/s0/photo_2016-03-04_12-53-02.jpg "echo bot.jpg")

# Conclusions

We have seen a simple platform to host our example code that we will discuss on future blog posts. 

The interesting part of hook.io is that is open source, you can see the [source code on GitHub](https://github.com/bigcompany/hook.io). 

If you enjoyed this blog post please leave a comment below and let me know what do you want to see more, I'm looking forward to start a blog series on developing telegram bots. 

Please consider also to share this blog post and follow me on [Twitter](https://twitter.com/unnikked) and Telegram [channel](https://telegram.me/unnikkedga). If you want to discuss this blog post you can join also my Telegram [supergroup](https://telegram.me/joinchat/AEis8D1UWPDa9sdeyzJQ6w). 
