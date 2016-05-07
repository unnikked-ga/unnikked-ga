---
layout:     post
title:      "Handling multimedia files via Telegram Bots API"
subtitle:   "Because sending text is not enough funny"
date:       2016-05-07 15:00:00
author:     "Nicola Malizia"
tags: ["telegram"]

header-img: "data/cover/telegram-background.jpg"

twitter-card: true
twitter-image: "https://lh3.googleusercontent.com/-PNnRNoGFvV4/Vy3wEeDns0I/AAAAAAAAASM/-bJh5LZebn8aRAX9n_PemNZNbW8-dlu6gCLcB/s0/handling-multimedia-files-telegram-bot.jpg"

open-graph: true
open-graph-image: "https://lh3.googleusercontent.com/-PNnRNoGFvV4/Vy3wEeDns0I/AAAAAAAAASM/-bJh5LZebn8aRAX9n_PemNZNbW8-dlu6gCLcB/s0/handling-multimedia-files-telegram-bot.jpg"

---


When it comes to user interaction some images do always a good impression. Indeed images express better, in a short amount of time, information. 

But we are not here to discuss how effectively are images or multimedia files, we'll discover which type of content the Telegram Bot API supports and how we should use it in order to send a cool [low poly](https://reddit.com/r/low_poly) picture of Earth to our users. 

We will first explore the API that the platform provides to us, a simple example in Javascript that you can run on [hook.io](hook.io) and some final tips and considerations. 

## Multimedia related methods

If we look over the [official documentation](https://core.telegram.org/bots/api) we'll see that are supported pictures, audios, videos and as well as general files. 

Let's see together the `sendPhoto` endpoint for example. 

**sendPhoto**
Use this method to send photos. On success, the sent Message is returned.

|Parameters| Type Required|Description|
|-|-|
|chat_id|Integer or String|	Yes	Unique identifier for the target chat or username of the target channel (in the format `@channelusername`)
|photo	|InputFile or String	|Yes	Photo to send. You can either pass a `file_id` as String to resend a photo that is already on the Telegram servers, or upload a new photo using `multipart/form-data`.
|caption	|String	|Optional	Photo caption (may also be used when resending photos by file_id), 0-200 characters

You may have already noticed that I left some parameters from the official documentation, this is intentional as we only focus on the main concept.

- `chat_id` is the usual field that let's us reply to the incoming message. 

- `photo` field is the big deal here, what this field contains ? A string ? What is this InputFile ? 

### Using a file_id String
> You can either pass a file_id as String to resend a photo that is already on the Telegram servers

This is straight forward to understand. This means that you can use a string to resend a file (can be a photo, video etc) that was previously uploaded to telegram server, by a user or a bot itself. 

> **Resending files without reuploading**
> There are **two ways** of sending a file (photo, sticker, audio etc.). If it‘s a new file, you can upload it using `multipart/form-data`. If the file is already on our servers, you don’t need to reupload it: each file object has a file_id field, you can simply pass this file_id as a parameter instead.

> - It is not possible to change the file type when resending by file_id. I.e. a [video](https://core.telegram.org/bots/api#video) can't be [sent as a photo](https://core.telegram.org/bots/api#sendphoto), a photo can't be sent as a document, etc.
> - It is not possible to resend thumbnails.
>- Resending a photo by file_id will send all of its [sizes](https://core.telegram.org/bots/api#photosize).

We can grab `file_id` from the [Audio](https://core.telegram.org/bots/api#audio), [Document](https://core.telegram.org/bots/api#document), [PhotoSize](https://core.telegram.org/bots/api#photosize), [Sticker](https://core.telegram.org/bots/api#sticker), [Video](https://core.telegram.org/bots/api#video) and [Voice](https://core.telegram.org/bots/api#voice) types or from a database where we stored it previously.

### Using the InputFile type
> **InputFile**
> This object represents the contents of a file to be uploaded. Must be posted using `multipart/form-data` in the usual way that files are uploaded via the browser. 

But what is this `multipart/form-data` thing? It's a method of encoding provided by the HTTP protocol, it roughly means that we can send binary data over this protocol. 

It's crucial that you set your HTTP library to use this encoding type when sending files. 

Can be this `photo` field a URL? **No it can't**, the reason is simple: a URL represents only the logical location of the resource and it does not contains any data about the resource. 

## Hook.io Example code

So what's the deal with the code ? Honestly the example I'm providing you is not the best as the library do all the magic things behind, but it's a good start to show you the flow. Here it is an example that download a picture from the web and sends it to the user. 

```javascript
module['exports'] = function imgBot (hook) {
	var request = require('request');
  	var TOKEN = hook.env.bot_key;
  	var ENDPOINT = 'https://api.telegram.org/bot' + TOKEN;
  
  	var message = hook.params.message;
  	var from = message.chat.id;
  	
  	if (message.text == "/img") {
    		var photoURL = "http://i.imgur.com/5rOhtdL.png";
    		var formData = {
    			chat_id: from,
    			photo: request(photoURL)
    		};
    		request.post({
    			url: ENDPOINT + '/sendPhoto',
    			formData: formData
    		});
  	}
}
```

The example is not the best because I cannot store files on hook.io so I had to download a file from internet first to send. 

```javascript
var formData = {
	chat_id: from,
    photo: /* put a file handle here */
};
```

In general when sending a `multipart/form-data`  request you only need to provide the path (or a stream depending of the library) of the file you want to send. 

The end result is

![End result](https://lh3.googleusercontent.com/-uZ7tJ0NaaIA/Vy27HnD3QxI/AAAAAAAAAR4/5A2EGKWpY4sCQGd9jM89oqo9nQJwPDEHACLcB/s0/Schermata+del+2016-05-07+11%253A52%253A49.png "Schermata del 2016-05-07 11:52:49.png")

To recap the follow to send a file (we understand that it can be a picture, a video etc) is the following:

- the file must exists somewhere, on a file system or on another machine
- the request must be a POST request with the encoding type set to `multipart/form-data`

### What about downloading files ? 

Users can send to your bot also multimedia files, by using the method `getFile` you can retrieve this file and process it as you need. 

>**getFile**
>Use this method to get basic info about a file and prepare it for downloading. For the moment, **bots can download files of up to 20MB** in size. On success, a [File](https://core.telegram.org/bots/api#file) object is returned. The file can then be downloaded via the link `https://api.telegram.org/file/bot<token>/<file_path>`, where `<file_path>` is taken from the response. It is guaranteed that the link will be valid for at least 1 hour. When the link expires, a new one can be requested by calling `getFile` again.

## Conclusions

Multimedia file handling for bots provides new ways of user interaction, as an example you can build a bot that generates graphs from some data and send it as a picture to your user. 

You can build a music library or a video generator out of user sent photos. You can build a text to speech bot by sending audio files. 

As always applications are endless. 

If you have ideas, please share them below so we can discuss and improve the knowledge about bots.

If you liked this blog post please make sure to share it with your friends and social media.

Furthermore if you want to join my group and be greeted by my bot and the community, [join now](https://telegram.me/joinchat/AEis8D1UWPDa9sdeyzJQ6w)!




