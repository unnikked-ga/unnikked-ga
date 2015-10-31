---
layout:     post
title:      "How I coded a Telegram bot that evaluates Lisp code"
subtitle:   "Code processing right on your phone"
date:       2015-06-28 14:42:00
author:     "Nicola Malizia"
tags: ["php", "telegram"]

header-img: "data/cover/telegram-background.jpg"

twitter-card: true

open-graph: true
---

My last blog post on Telegram Bots it was well received, unexpectedly. By viewing some requests I decided to show you how I have setup my [@LispBot](https://telegram.me/LispBot), a little toy bot that evaluates a sort of Lisp code.

The bot is based on the PHP Liso interpreter [Ilias](https://github.com/igorw/ilias) made by [Igor](https://github.com/igorw), if you want to know how this internally works checkout his [blog post](https://igor.io/2012/12/06/sexpr.html) series. 

I'm sure that some reader have already asked to himself: Why did you use PHP ? 

Simply I wanted to put something online without leaving my personal computer open 24/7. Since the APIs of those bot are accessible via simple HTTP calls and since you can get a free host that uses PHP, I thought, for a toy project, PHP should do the trick. 

## Prerequisites
In order to setup a project like this you'll need:

0. A web host, there are plenty free web host services out there that supports PHP. 
0. A personal domain name, I registered one for free using [freenom](freenom.com).
0. A Cloudflare account to manage your domain, so you can get it's free Universal SSL, needed for set the webhook call back. 
0. Barebone PHP or a micro-framework or a fully fledged framework, this is up to you. 

I'm not going to explain step by step point **1.**, **2.** and **3.** we will focus instead on how I implemented the point **4.** providing some observations at the end. 

While I could have implemented everything just using plain PHP I used FatFreeFramework for the routing, after you will see the code you will understand that it was not needed to use it. 

## The workflow
Before to show you how to set the endpoint to the bot (obviously [create a bot](https://unnikked.ga/getting-started-with-telegram-bots) first let's see how schematically I designed the bot.

- Receiving the POST request on my endpoint. 
- Checking if it's a valid JSON. 
- Checking if there is any valid command. 
- Processing the code. 
- Reply to the client with the evaluated code. 

Unfortunately PHP lacks of concurrent programming, so each request it's done sequentially. 

## The Bot

Finally let's dive into the code of the bot, note that this is not the working version. 

```php
<?php

$f3->route('POST /process', function ($f3) {
	// decoding input
	$message = json_decode(file_get_contents('php://input'), true);
	// if it's not a valid JSON return
	if(is_null($message)) return;
	$endpoint = "https://api.telegram.org/bot<token>/";
	
	// make sure if text and chat_id is set into the payload
	if(!isset($message["message"]["text"]) || !isset($message["message"]["chat"]["id"])) return;
	
	// remove special characters, needed for the interpreter apparently
	$text = $message["message"]["text"];
	$text = str_replace(["\r", "\n", "\t"], ' ', $text);
	
	// saving chat_id for convenience
	$chat_id = $message["message"]["chat"]["id"];
	
	// if the string will not start with /eval return
	if(!preg_match("/\/eval/", $text)) return;
	
	// show to the client that the bot is typing 
	file_get_contents($endpoint."sendChatAction?chat_id=".$chat_id."&action=typing");
	
	// bootstrapping the interpreter
	$program = new Program(new Lexer(), new Reader(), new FormTreeBuilder(), new Walker());
	
	// if there is something wrong send the message to the client, this error is related to the code passed
	try {
		$env = Environment::standard();
		$value = $program->evaluate($env, $text);
	} catch (RuntimeException $e) {
		file_get_contents($endpoint."sendMessage?chat_id=".$chat_id."&text=".urlencode($e->getMessage()));
		return;
	}
	// send to the client the code evaluated
	file_get_contents($endpoint. "sendMessage?chat_id=".$chat_id."&text=".$value);
});
```

You may wondering why I simply return (the framework will send by default HTTP 200 status code) instead of returning a 400 status code message ? 

Well if you send a status code different from 200 Telegram servers will resend you the same command to process, so your bot will probably remain stuck processing and telling the same thing to the server. 

The status code 200 should be used, based on my observations, to tell to Telegram Bot API: _Ok, I received your command, I will process it_.

## Setting the webhook

You have now your bot up and running on your fancy server. How to tell to Telegram to send messages to it ? Simply use the `setWebhook` method. 

Open your browser and navigate here `https://api.telegram.org/bot<token>/setWebhook?url=https://your_bot_end_point`

Where `your_bot_end_point` is your domain name as example `https://mybotdomain.com/process`

You should see a message like that: 

```json
{"ok":true,"result":true,"description":"Webhook was set"}
```

You have to make sure that you have a signed ssl certificate, that's why I use [Cloudflare Universal SSL feature](https://www.cloudflare.com/ssl). 

## Some observations
- If you use the webhook method make sure to reply quickly with 200 status code by processing the command using background threads or processes, so you can process other messages as fast as you can. ([Producer Consumer problem](https://en.wikipedia.org/wiki/Producer%E2%80%93consumer_problem)). 
- Bot in chat groups receive only string that starts with `/`, you can still disable privacy to let grab whatever input.
- On personal conversation the bot will try to process every message sent to it. 

This bot kinda sucks, because it process every input sequentially. PHP lacks about concurrent programming so you cannot create threads to let process the command. 

## Conclusion

If you want to test this bot chat with [@LispBot](http://telegram.me/LispBot). As some of you suggested what to know how to process messages using the `getUpdates` method, I'm working on it and I'll try to write a blog post as soon as possible. 

If you have any idea or comment please let me know in the comment, I would love to read it. 
