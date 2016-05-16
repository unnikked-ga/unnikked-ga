---
layout:     post
title:      "A Telegram Channel And Group Scheduler Out Of Google Calendar, IFTTT And Hook.io"
subtitle:   "Schedule with easy multimedia files to your channel"
date:       2016-05-16 00:00:00
author:     "Nicola Malizia"
tags: ["telegram"]

header-img: "data/cover/telegram-background.jpg"

twitter-card: true
twitter-image: "https://unnikked.ga/data/telegram-channel-scheduler/social.jpg"

open-graph: true
open-graph-image: "https://unnikked.ga/data/telegram-channel-scheduler/social.jpg"
---

A while ago I wrote a blog post about _[How to manage your Telegram channel with your unique bot](how-to-manage-telegram-channel-with-bot)_. After publishing the other day _[Handling multimedia files via Telegram Bots API](multimedia-files-telegram-bots)_ and viewing some recent comments on the first post I started thinking if it could be possible to schedule also multimedia files. It turns out we can!

We all know that visual contents appeals more, that's why I build this method for you (and mainly for me) to help you schedule multimedia contents to your Telegram Channel. It works also for groups!

**Disclaimer**: This method is a hack! Not that you have to hack something to use it, it's not an usual application where you login and you are good to go, this method uses three existing services:

- **Google Calendar**: it gives us a visual representation of the scheduled contents and it's also our UI.
- **IFTTT**: it servers as a middleman to retrieve events from Google Calendar and routing them to a custom script.
- **Hook.io**: here relies the script that does the magic!

The cool thing about it is that out of existing service you can build more! It took me literally 5 minutes to code the script!

#### Pros vs Cons

<i class="fa fa-plus" aria-hidden="true"></i> Made out of existing components<br>
<i class="fa fa-plus" aria-hidden="true"></i> The UI gives us a quick glance of what contents we are posting

<i class="fa fa-minus" aria-hidden="true"></i> UX can be clunky for people that does not know the JSON syntax<br>
<i class="fa fa-minus" aria-hidden="true"></i> It might be difficult to set up.<br>
<i class="fa fa-minus" aria-hidden="true"></i> Multimedia files needs to be on a public server.

That's why I'll try to help you set up this as easy as possible.

## Prerequisites

- A Google account in order to link Google Calendar to IFTTT
- IFTTT account that uses this recipe (I'll show how to configure later)
- A Hook.io account

I hope you don't need instructions on how to register accounts on Google, IFTTT and Hook.io. I'll skip this steps focusing on the important ones.

# Step 1 - Create a new Hook service

The first step is to create a [new service on hook](https://hook.io/new) that handles the request received from IFTTT triggered by Google Calendar events.

![Create a new Service](/data/telegram-channel-scheduler/hook-io-create-new-service.png)

Where:

- `Name` is the name of the hook, name it as you like most.
- `Route` must be empty

> **Double attention here** If you look the image you will see a white box with some strange urls, make sure you copy the link corresponding to the entry **Home**. _**You must copy it after you gave a name to the hook**_

Hook source is the source code of our microservice.

![Hook source](/data/telegram-channel-scheduler/hook-io-hook-source.png)

Where:

- `Gist url` must be this url `https://gist.github.com/unnikked/6fccd3216ed32eba0f8548e85e8c59cb`

Hit the **Create new service** button.

It's important that you set the service into _production_ mode so it can perform faster. To do it you have to go to the service control panel and set it to production.

![Hook production](/data/telegram-channel-scheduler/hook-io-set-production.png)

It's also important to set the environment variable `bot_scheduler_token` to the bot token that you get from talking with [@botfather](/getting-started-with-telegram-bots/)

Just go to the [environment section](https://hook.io/env) and set the token.

![Hook env](/data/telegram-channel-scheduler/hook-io-set-evn.png)

# Step 2 - Adding the custom IFTTT recipe

Lucky for you I've already made a recipe that it is ready to go.

<p align="center"><a href="https://ifttt.com/view_embed_recipe/417565-schedule-posts-to-telegram-channels-and-groups" class="embed_recipe embed_recipe-l_46" id= "embed_recipe-417565"><img src="https://ifttt.com/recipe_embed_img/417565" alt="IFTTT Recipe: Schedule posts to Telegram Channels and Groups connects google-calendar to maker" width="370px" style="max-width:100%"/></a><script async type="text/javascript" src= "//ifttt.com/assets/embed_recipe.js"></script>
</p>

Click on the **Add** button and follow the rest of this blog post.

![Add Recipe](/data/telegram-channel-scheduler/ifttt-add-recipe.png)

You'll see this bit of page, paste here the URL generated from Step 1 at `Double attention here`.

Hit the big blue **Add** button.

If you believe or not you are good to go! Don't worry I got you covered and you will know find everything you need to know on how to use it, so please read carefully as there are some gotchas.

# How to use

If you read my [previous blog post](/how-to-manage-telegram-channel-with-bot/) you already know that the content has to be written in the description box of the event. Nothing changed except.

- The **title** of the event must contains the word `#scheduler`.
- The **description** of the event must be a JSON string.

The reason behind this decision is due to the limitation that IFTTT and the Telegram Bot API platform have:

0. You cannot get from IFTTT the attachment from the event.
0. You cannot provide URIs for multimedia files when you call a Telegram Bot API endpoint.

For the reason listed before the script on hook.io serves to fill those _"problems"_.

The JSON string must be of the form:

```javascript
{
  "type": "type",
  "data": {
    // params here
  }
}
```

You will add entries as follow

![Google Calendar Entry](/data/telegram-channel-scheduler/google-calendar-entry.png)

Where `type` can be one of the following: `message`, `audio`, `document`, `sticker`, `video`, `location`, `venue`, `contact`.
And `data` depends by the specific `type`, follow the documentation below.

From now on I will provide you some templates to use directly when you need to schedule some contents, [saving you time](https://zapier.com/blog/how-to-make-document-template/).

## message
Use this type to send text messages.

|Parameters|Type|Required|Description
|-|-|-|-|
|`chat_id`|Integer or String| Yes	|Unique identifier for the target chat or username of the target channel (in the format `@channelusername`)
|`text`|String|Yes| Text of the message to be sent
|`parse_mode`|String|Optional|Send Markdown or HTML, if you want Telegram apps to show bold, italic, fixed-width text or inline URLs in your bot's message.
|`disable_web_page_preview`|Boolean|Optional|Disables link previews for links in this message
|`disable_notification`|Boolean|Optional|Sends the message silently. iOS users will not receive a notification, Android users will receive a notification with no sound.

---

#### Formatting options
> _I copy pasted this section from [here](https://core.telegram.org/bots/api#formatting-options) just for convenience_

The Bot API supports basic formatting for messages. You can use bold and italic text, as well as inline links and pre-formatted code in your bots' messages. Telegram clients will render them accordingly. You can use either markdown-style or HTML-style formatting.

Note that Telegram clients will display an alert to the user before opening an inline link (‘Open this link?’ together with the full URL).

**Markdown style**
To use this mode, pass _Markdown_ in the _parse_mode_ field when using `message` type. Use the following syntax in your message:

```
*bold text*
_italic text_
[text](URL)
`inline fixed-width code`
```pre-formatted fixed-width code block```
```

**HTML style**
To use this mode, pass HTML in the parse_mode field when using `message` type. The following tags are currently supported:

```
<b>bold</b>, <strong>bold</strong>
<i>italic</i>, <em>italic</em>
<a href="URL">inline URL</a>
<code>inline fixed-width code</code>
<pre>pre-formatted fixed-width code block</pre>
```

Please note:

- Only the tags mentioned above are currently supported.
- Tags must not be nested.
- All `<`, `>` and `&` symbols that are not a part of a tag or an HTML entity must be replaced with the corresponding HTML entities (`<` with `&lt;`,` >` with `&gt;` and `&` with `&amp;`).
- All numerical HTML entities are supported.
- The API currently supports only the following named HTML entities: `&lt;`, `&gt;`, `&amp;` and `&quot;`.

---

### Send only text

```javascript
{
  "type": "message",
  "data": {
    "chat_id": "@channelusername",
    "text": "your text here"
  }
}
```

### Send only text disabling web pages previews

```javascript
{
  "type": "message",
  "data": {
    "chat_id": "@channelusername",
    "text": "your text here",
    "disable_web_page_preview": true
  }
}
```

### Send only text disabling web pages previews and notifications

```javascript
{
  "type": "message",
  "data": {
    "chat_id": "@channelusername",
    "text": "your text here",
    "disable_web_page_preview": true,
    "disable_notification": true
  }
}
```

### Send Markdown formatted text

```javascript
{
  "type": "message",
  "data": {
    "chat_id": "@channelusername",
    "text": "your text here",
    "parse_mode": "Markdown"
  }
}
```

### Send Markdown formatted text disabling web pages previews

```javascript
{
  "type": "message",
  "data": {
    "chat_id": "@channelusername",
    "text": "your text here",
    "parse_mode": "Markdown",
    "disable_web_page_preview": true
  }
}
```

### Send Markdown formatted text disabling web pages previews and notifications

```javascript
{
  "type": "message",
  "data": {
    "chat_id": "@channelusername",
    "text": "your text here",
    "parse_mode": "Markdown",
    "disable_web_page_preview": true,
    "disable_notification": true
  }
}
```

### Send HTML formatted text

```javascript
{
  "type": "message",
  "data": {
    "chat_id": "@channelusername",
    "text": "your text here",
    "parse_mode": "HTML"
  }
}
```

### Send HTML formatted text disabling web pages previews

```javascript
{
  "type": "message",
  "data": {
    "chat_id": "@channelusername",
    "text": "your text here",
    "parse_mode": "HTML",
    "disable_web_page_preview": true
  }
}
```

### Send HTML formatted text disabling web pages previews and notifications

```javascript
{
  "type": "message",
  "data": {
    "chat_id": "@channelusername",
    "text": "your text here",
    "parse_mode": "HTML",
    "disable_web_page_preview": true,
    "disable_notification": true
  }
}
```

## photo

Use this type to send photos.

|Parameters|Type|Required|Description|
|`chat_id`|Integer or String|Yes|Unique identifier for the target chat or username of the target channel (in the format `@channelusername`)
|`photo`|URL|Yes|Photo to send. Must be a url available publicly
|`caption`|String|Optional|Photo caption, 0-200 characters
|`disable_notification`|Boolean|Optional|Sends the message silently. iOS users will not receive a notification, Android users will receive a notification with no sound.

Please note that the photo must be an url of a picture that is uploaded on the Internet (i.e you can download with your browser). Have you a custom picture and you want to use it ? Upload it on an image hosting website such as [imgur.com](https://imgur.com) and make sure you use the **Direct Link**!

![Imgur Hosting](/data/telegram-channel-scheduler/imgur.png)

### Send a photo

```javascript
{
  "type": "photo",
  "data": {
    "chat_id": "@channelusername",
    "photo": "your url here"
  }
}
```

### Send a photo disabling notification

```javascript
{
  "type": "photo",
  "data": {
    "chat_id": "@channelusername",
    "photo": "your url here",
    "disable_notification": true
  }
}
```

### Send a photo with a caption

```javascript
{
  "type": "photo",
  "data": {
    "chat_id": "@channelusername",
    "photo": "your url here",
    "caption": "your caption here"
  }
}
```

### Send a photo with a caption disabling notification

```javascript
{
  "type": "photo",
  "data": {
    "chat_id": "@channelusername",
    "photo": "your url here",
    "caption": "your caption here",
    "disable_notification": true
  }
}
```

### Example

This

```javascript
{
  "type": "photo",
  "data": {
    "chat_id": "4762864",
    "photo": "http://i.imgur.com/B2v73sN.png",
    "caption": "My favourite background"
  }
}
```

Becomes

![Example Photo](/data/telegram-channel-scheduler/example-photo.jpg)

## audio

Use this type to send audio files, if you want Telegram clients to display them in the music player. Your audio must be in the .mp3 format. On success, the sent Message is returned. **Bots can currently send audio files of up to 50 MB in size**, this limit may be changed in the future.

For sending voice messages, use the `voice` type instead.

|Parameters|Type|Required|Description
|`chat_id`|Integer or String|Yes|Unique identifier for the target chat or username of the target channel (in the format `@channelusername`)
|`audio`|URL|Yes|Audio file to send. Must be a url available publicly
|`duration`|Integer|Optional|Duration of the audio in seconds
|`performer`|String|Optional|Performer
|`title`|String|Optional|Track name
|`disable_notification`|Boolean|Optional|Sends the message silently. iOS users will not receive a notification, Android users will receive a notification with no sound.

Please note that the audio must be an url of an audio file that is uploaded on the Internet (i.e you can download with your browser).

### Send audio with duration, performer and title

```javascript
{
  "type": "audio",
  "data": {
    "chat_id": "@channelusername",
    "audio": "your url here",
    "duration": 1548,
    "performer": "performer name",
    "title": "title of the audio"
  }
}
```

### Send audio with duration, performer and title disabling notification

```javascript
{
  "type": "audio",
  "data": {
    "chat_id": "@channelusername",
    "audio": "your url here",
    "duration": 1548,
    "performer": "performer name",
    "title": "title of the audio",
    "disable_notification": true
  }
}
```

### Example

This

```javascript
{
  "type": "audio",
  "data": {
    "chat_id": "4762864",
    "audio": "https://dl.airtable.com/mrv3DjXxRPapF35y24Nm_3211_alink_guitar_creativ_noncom.mp3",
    "performer": "Sample audio"
  }
}
```

Becomes

![Example Photo](/data/telegram-channel-scheduler/example-photo.jpg)

## document

Use this type to send general files. **Bots can currently send files of any type of up to 50 MB in size, this limit may be changed in the future.**

|Parameters|Type|Required|Description
|`chat_id`|Integer or String|Yes|Unique identifier for the target chat or username of the target channel (in the format `@channelusername`)
|`document`|URL|Yes|File to send. Must be a url available publicly
|`caption`|String|Optional|Photo caption (may also be used when resending photos by file_id), 0-200 characters
|`disable_notification`|Boolean|Optional|Sends the message silently. iOS users will not receive a notification, Android users will receive a notification with no sound.

Please note that the document must be an url of a file that is uploaded on the Internet (i.e you can download with your browser).

### Send a document

```javascript
{
  "type": "document",
  "data": {
    "chat_id": "@channelusername",
    "document": "your url here"
  }
}
```

### Send a document disabling notification

```javascript
{
  "type": "document",
  "data": {
    "chat_id": "@channelusername",
    "document": "your url here",
    "disable_notification": true
  }
}
```

### Send a document with a caption

```javascript
{
  "type": "document",
  "data": {
    "chat_id": "@channelusername",
    "document": "your url here",
    "caption": "your caption here"
  }
}
```

### Send a document with a caption disabling notification

```javascript
{
  "type": "document",
  "data": {
    "chat_id": "@channelusername",
    "document": "your url here",
    "caption": "your caption here",
    "disable_notification": true
  }
}
```

### Example

This

```javascript
{
  "type": "document",
  "data": {
    "chat_id": "4762864",
    "document": "https://dl.airtable.com/H3IDEjsPQrebjBM479Mw_iburg.pdf",
    "caption": "Sample document"
  }
}
```

Becomes

![Example document](/data/telegram-channel-scheduler/example-document.jpg)

## sticker

Use this type to send .webp stickers.

|Parameters|Type|Required|Description
|`chat_id`|Integer or String|Yes|Unique identifier for the target chat or username of the target channel (in the format `@channelusername`)
|`sticker`|URL|Yes|sticker to send. Must be a url available publicly
|`disable_notification`|Boolean|Optional|Sends the message silently. iOS users will not receive a notification, Android users will receive a notification with no sound.

Please note that the sticker must be an url of a .webp file that is uploaded on the Internet (i.e you can download with your browser).

### Send a sticker

```javascript
{
  "type": "sticker",
  "data": {
    "chat_id": "@channelusername",
    "sticker": "your url here"
  }
}
```

### Send a sticker disabling notification

```javascript
{
  "type": "sticker",
  "data": {
    "chat_id": "@channelusername",
    "sticker": "your url here",
    "disable_notification": true
  }
}
```

### Example

This

```javascript
{
  "type": "sticker",
  "data": {
    "chat_id": "4762864",
    "sticker": "https://dl.airtable.com/aeRVgwZVQAatdFMFTgQt_sticker.webp",
  }
}
```

Becomes

![Example sticker](/data/telegram-channel-scheduler/example-sticker.jpg)


## video

Use this type to send video files, Telegram clients support mp4 videos (other formats may be sent as `document`). **Bots can currently send video files of up to 50 MB in size, this limit may be changed in the future.**

|Parameters|Type|Required|Description
|`chat_id`|Integer or String|Yes|Unique identifier for the target chat or username of the target channel (in the format `@channelusername`)
|`video`|URL|Yes|video to send. Must be a url available publicly
|`duration`|Integer|Optional|Duration of sent video in seconds
|`width`|Integer|Optional|Video width
|`height`|Integer|Optional|Video height
|`caption`|String|Optional|Photo caption (may also be used when resending photos by file_id), 0-200 characters
|`disable_notification`|Boolean|Optional|Sends the message silently. iOS users will not receive a notification, Android users will receive a notification with no sound.

Please note that the video must be an url of a .mp4 file that is uploaded on the Internet (i.e you can download with your browser).

### Send a video

```javascript
{
  "type": "video",
  "data": {
    "chat_id": "@channelusername",
    "video": "your url here"
  }
}
```

### Send a video disabling notification

```javascript
{
  "type": "video",
  "data": {
    "chat_id": "@channelusername",
    "video": "your url here",
    "disable_notification": true
  }
}
```

### Send a video with a caption

```javascript
{
  "type": "video",
  "data": {
    "chat_id": "@channelusername",
    "video": "your url here",
    "caption": "your caption here"
  }
}
```

### Send a video with a caption disabling notification

```javascript
{
  "type": "video",
  "data": {
    "chat_id": "@channelusername",
    "video": "your url here",
    "caption": "your caption here",
    "disable_notification": true
  }
}
```

### Example

This

```javascript
{
  "type": "video",
  "data": {
    "chat_id": "4762864",
    "video": "https://dl.airtable.com/zGIQDmahSDShGSOJ74ea_39932d67949391d488a735149949ac1a.mp4",
    "caption": "Sample Video"
  }
}
```

Becomes

![Example video](/data/telegram-channel-scheduler/example-video.jpg)

## voice

Use this method to send audio files, if you want Telegram clients to display the file as a playable voice message. For this to work, your audio must be in an .ogg file encoded with OPUS (other formats may be sent as `audio` or `document`). **Bots can currently send voice messages of up to 50 MB in size, this limit may be changed in the future.**

|Parameters|Type|Required|Description
|`chat_id`|Integer or String|Yes|Unique identifier for the target chat or username of the target channel (in the format `@channelusername`)
|`voice`|URL|Yes|voice to send. Must be a url available publicly
|`duration`|Integer|Optional|Duration of sent video in seconds
|`disable_notification`|Boolean|Optional|Sends the message silently. iOS users will not receive a notification, Android users will receive a notification with no sound.

Please note that the video must be an url of a .ogg file that is uploaded on the Internet (i.e you can download with your browser).

### Send a voice

```javascript
{
  "type": "voice",
  "data": {
    "chat_id": "@channelusername",
    "voice": "your url here"
  }
}
```

### Send a voice disabling notification

```javascript
{
  "type": "voice",
  "data": {
    "chat_id": "@channelusername",
    "voice": "your url here",
    "disable_notification": true
  }
}
```

### Example

This

```javascript
{
  "type": "voice",
  "data": {
    "chat_id": "4762864",
    "voice": "https://dl.airtable.com/zvDVAvDmSqvrUy9lCloS_audio_2016-05-15_15-35-05.ogg",
    "caption": "Sample sticker"
  }
}
```

Becomes

![Example voice](/data/telegram-channel-scheduler/example-voice.jpg)

## location

Use this type to send point on the map.

|Parameters|Type|Required|Description
|`chat_id`|Integer or String|Yes|Unique identifier for the target chat or username of the target channel (in the format `@channelusername`)
|`latitude`|Float number|Yes|Latitude of location
|`longitude`|Float number|Yes|Longitude of location
|`disable_notification`|Boolean|Optional|Sends the message silently. iOS users will not receive a notification, Android users will receive a notification with no sound.

### Send a location

```javascript
{
  "type": "location",
  "data": {
    "chat_id": "@channelusername",
    "latitude": 123.456,
    "longitude": 789.123
  }
}
```

### Send a location disabling notification

```javascript
{
  "type": "location",
  "data": {
    "chat_id": "@channelusername",
    "latitude": 123.456,
    "longitude": 789.123,
    "disable_notification": true
  }
}
```

### Example

This

```javascript
{
  "type": "location",
  "data": {
    "chat_id": "4762864",
    "latitude": 41.9,
    "longitude": 12.5
  }
}
```

Becomes

![Example Location](/data/telegram-channel-scheduler/example-location.jpg)

> **Note** This location is not where i live ;)

## venue
Use this method to send information about a venue.

|Parameters|Type|Required|Description
|`chat_id`|Integer or String|Yes|Unique identifier for the target chat or username of the target channel (in the format `@channelusername`)
|`latitude`|Float number|Yes|Latitude of the venue
|`longitude`|Float number|Yes|Longitude of the venue
|`title`|String|Yes|Name of the venue
|`address`|String|Yes|Address of the venue
|`foursquare_id`|String|Optional|Foursquare identifier of the venue
|`disable_notification`|Boolean|Optional|Sends the message silently. iOS users will not receive a notification, Android users will receive a notification with no sound.

### Send a venue

```javascript
{
  "type": "venue",
  "data": {
    "chat_id": "@channelusername",
    "latitude": 123.456,
    "longitude": 789.123,
    "title": "your title here",
    "address": "your address here"
  }
}
```

### Send a venue disabling notification

```javascript
{
  "type": "venue",
  "data": {
    "chat_id": "@channelusername",
    "latitude": 123.456,
    "longitude": 789.123,
    "title": "your title here",
    "address": "your address here",
    "disable_notification": true
  }
}
```

### Send a venue with a Foursquare identifier

```javascript
{
  "type": "venue",
  "data": {
    "chat_id": "@channelusername",
    "latitude": 123.456,
    "longitude": 789.123,
    "title": "your title here",
    "address": "your address here",
    "foursquare_id": "Foursquare identifier"
  }
}
```

### Send a venue with a Foursquare identifier disabling notification

```javascript
{
  "type": "venue",
  "data": {
    "chat_id": "@channelusername",
    "latitude": 123.456,
    "longitude": 789.123,
    "title": "your title here",
    "address": "your address here",
    "foursquare_id": "Foursquare identifier",
    "disable_notification": true
  }
}
```

### Example

This

```javascript
{
  "type": "venue",
  "data": {
    "chat_id": "4762864",
    "latitude": 41.9,
    "longitude": 12.5,
    "title": "Capital of Italy",
    "address": "Centre of the city"
  }
}
```

Becomes

![Example venue](/data/telegram-channel-scheduler/example-venue.jpg)

## contact

Use this type to send phone contacts.

|Parameters|Type|Required|Description
|`chat_id`|Integer or String|Yes|Unique identifier for the target chat or username of the target channel (in the format `@channelusername`)
|`phone_number`|String|Yes|Contact's phone number
|`first_name`|String|Yes|Contact's first name
|`last_name`|String|Optional|Contact's last name
|`disable_notification`|Boolean|Optional|Sends the message silently. iOS users will not receive a notification, Android users will receive a notification with no sound.

### Send a contact

```javascript
{
  "type": "contact",
  "data": {
    "chat_id": "@channelusername",
    "phone_number": "555555555555",
    "first_name": "your first name here",
  }
}
```

### Send a contact disabling notification

```javascript
{
  "type": "contact",
  "data": {
    "chat_id": "@channelusername",
    "phone_number": "555555555555",
    "first_name": "your first name here",
    "disable_notification": true
  }
}
```

### Send a contact including a last name

```javascript
{
  "type": "contact",
  "data": {
    "chat_id": "@channelusername",
    "phone_number": "555555555555",
    "first_name": "your first name here",
    "last_name": "your last name here"
  }
}
```

### Send a contact including a last name disabling notification

```javascript
{
  "type": "venue",
  "data": {
    "chat_id": "@channelusername",
    "phone_number": "555555555555",
    "first_name": "your first name here",
    "last_name": "your last name here",
    "disable_notification": true
  }
}
```

### Example

This

```javascript
{
  "type": "contact",
  "data": {
    "chat_id": "4762864",
    "phone_number": "555-5555555",
    "first_name": "unnikked"
  }
}
```

Becomes

![Example contact](/data/telegram-channel-scheduler/example-contact.jpg)

## Conclusions

If I've not lost you, you are now capable of scheduling all type of contents on your Telegram channel and if you understood well the main concept behind you may be aware that this mechanism can be adapted to other scenarios.

Have you any comment or question about that ? Please let me know in the comment box so I can update this article for you and improving our common knowledge. I cannot wait to see what you came up with.

If you liked this blog post please make sure to share it with your friends and social media.

Furthermore if you want to join my group and be greeted by my bot and the community, [join now](https://telegram.me/joinchat/AEis8D1UWPDa9sdeyzJQ6w)!
