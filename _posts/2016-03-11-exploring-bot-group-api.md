---
layout:     post
title:      "Exploring Telegram Bot API for Groups"
subtitle:   "Maybe we want more actions"
date:       2016-03-11 20:30:00
author:     "Nicola Malizia"
tags: ["telegram", "javascript"]

header-img: "data/cover/telegram-background.jpg"

twitter-card: true
twitter-image: "https://lh3.googleusercontent.com/-N4xzHAWIAgw/VuMgtCR3VtI/AAAAAAAAARg/YAgqGMHBQ9kKxgv39KvIBR8HEi08B7gNw/s0/api-group.jpg"

open-graph: true
open-graph-image: "https://lh3.googleusercontent.com/-N4xzHAWIAgw/VuMgtCR3VtI/AAAAAAAAARg/YAgqGMHBQ9kKxgv39KvIBR8HEi08B7gNw/s0/api-group.jpg"

---

When it comes to Telegram groups, the idea to add a bot is very natural. Bots can help to maintain groups active by providing custom features. 

As an example:

- [@PoolBot](https://telegram.me/PollBot) to create pools for groups
- [@TriviaBot](https://telegram.me/TriviaBot) to challenge a group to reply questions
- [@GithubBot](https://telegram.me/githubbot) to get notified of pushes for your coders group. 


The possibilities are endless. Unfortunately what Telegram lack for groups is the ability to let a bot moderate a group for you (so you can kick off spammers for example). 

In this article we are going to see how we can use specific information that we can retrieve from the [Message](https://core.telegram.org/bots/api#message) type in order to customize the felling of a group. 

## What the Message type provide us ?

If we look carefully to the documentation we can see: 

|Field|Type|Description|
|:-|:-|:-|
|...|...|...|
|`new_chat_participant`|	[User](https://core.telegram.org/bots/api#user)	|*Optional.* A new member was added to the| group, information about them (this member may be the bot itself)|
|`left_chat_participant`|	[User](https://core.telegram.org/bots/api#user)	|*Optional.* A member was removed from the group, information about them (this member may be the bot itself)
|`new_chat_title`	|String	|*Optional.* A chat title was changed to this value
|`new_chat_photo`	|Array of [PhotoSize](https://core.telegram.org/bots/api#photosize)	|*Optional.* A chat photo was change to this value
|`delete_chat_photo`	|True	|*Optional.* Service message: the chat photo was deleted
|`group_chat_created`	|True|	*Optional.* Service message: the group has been created
|`supergroup_chat_created`|	True	|*Optional.* Service message: the supergroup has been created
|`channel_chat_created`	|True	|*Optional.* Service message: the channel has been created
|`migrate_to_chat_id`	|Integer	|*Optional.* The group has been migrated to a supergroup with the specified identifier, not exceeding 1e13 by absolute value
|`migrate_from_chat_id`	|Integer|	*Optional.* The supergroup has been migrated from a group with the specified identifier, not exceeding 1e13 by absolute value

As you can see those fields are only informative. If you look to the [available methods](https://core.telegram.org/bots/api#available-methods) you will not see any method that let's you perform actions on groups (besides sending messages etc).


But this should not discourage us as we can still use this information to customize our group experience. 

> In this blog post we assume that the bot can't read every message written on the group. That is bot privacy is set to *ENABLED*.

### The `new_chat_participant` Field

The use of this field should be trivial, the first example I can came cross is to greet a user when he joins a group. In my opinion this can be used to inform the user about the topics, about the rules or something similar. 

From our side we can use this information to keep track the number of participants and see how the group evolves over time. (Of course if you track user joins you must also track user unjoins). 

![new_chat_participant](https://lh3.googleusercontent.com/-CNnP0Lwuysc/VuMPgamhx_I/AAAAAAAAAQQ/XMKSNdNNze0rGUuK-sb4CuheJdiMnB9iA/s0/photo_2016-03-11_19-32-49.jpg "new_chat_participant")

> *Have you an idea ? Leave a comment below so we can expand further this section.* 

### The `left_chat_participant` Field

This filed tell us when a user leaves the group. As I explained before with this information you can track how many user leave the group over time so you can see how the group evolves. 

![left_chat_participant](https://lh3.googleusercontent.com/-6BNupwaoqd0/VuMP26cYC2I/AAAAAAAAAQY/3EekKugtIEYOvPx6BdbyInD4FiMo0c2lg/s0/left_chat_participant.jpg "left_chat_participant.jpg")

> *Have you an idea ? Leave a comment below so we can expand further this section.* 

### The `new_chat_title`, `new_chat_photo`, `delete_chat_photo` Fields

You may wondering ? What I can do with those fields ? Well, if we use our imagination we could use those information to let the bot react to them, or to trigger a particular server action or log the changes over time. 

Another thing I can think of is to use the group title to tell to the bot should behave; for example let's say we set the group title to *"Let's have some fun"*, the bot will see the word _fun_ and will start to make jokes and send funny pictures to the group. 

![group_title_changed](https://lh3.googleusercontent.com/-Ro1v_gSDZgI/VuMQQOOFkeI/AAAAAAAAAQk/GsZnlO4Vp7s_-PmDzHBe3-_w7342iCjcA/s0/group_title_change.jpg "group_title_change.jpg")

> *Have you an idea ? Leave a comment below so we can expand further this section.* 

### The `group_chat_created`, `supergroup_chat_created`, `channel_chat_created` Fields

Those events are generated when you create a group, super group and a channel respectively. 

You may use those fields to initiate a tracking procedure on the bot side to track joins and unjoins for example. 

![group_chat_created](https://lh3.googleusercontent.com/-Gh54d8ENfVU/VuMQ7n1wNHI/AAAAAAAAAQ4/fIUfBQE1r-AAUdcpAFSKFzNTIC3o0IsXA/s0/group_chat_created.jpg "group_chat_created.jpg")

> *Have you an idea ? Leave a comment below so we can expand further this section.* 

### The `migrate_to_chat_id`, `migrate_from_chat_id` Fields

Those fields, in my opinion, are handy to the developer side than the user side of the group. 

Let's say we create an external script that do some actions and automatically post something on a group, if you upgrade your group to a supergroup you can easily get the new group id with those fields. 

> *Have you an idea ? Leave a comment below so we can expand further this section.* 

## An Example: The Greeting Bot

Enough talking Nicola, let's us see some examples! If you are here, well thank you for reading me. 

It's the time to see some code! I will use [hook.io](https://hook.io/) as host platform to make our life easier. 

Some prerequisites in case you can't start from zero: 
1. [Getting Started With Telegram Bots](https://unnikked.ga/getting-started-with-telegram-bots) - my first blog post on Telegram Bot, it will explain you how to register a bot and explain basic things. 
2. [How to Easily Build a Telegram Bot With Hook.io](https://unnikked.ga/build-telegram-bot-hook-io) - this blog post will teach you how to set up the registered bot with hook.io

> The requirements must be followed in the order shown

### Shut up And Give us The Code!

Since we made clear what we want to do here it is the final code. 

```javascript
module['exports'] = function greetingBot (hook) {
	var request = require('request');
  	var user = hook.params.message.new_chat_participant;
  	if(user) {
      request
          .post('https://api.telegram.org/bot' + hook.env.bot_token + '/sendMessage')
          .form({
              "chat_id": hook.params.message.chat.id,
              "text": "Welcome " + user.first_name
          });
    }
};
```

## Bonus

My dear reader, this was a long blog post and I want to give you my test debug script that I used to check what each fields do. 

The code is hosted on [Github Gist](https://gist.github.com/unnikked/55183bcf99cc58916e8a). 

## Conclusion

Finally we ended. To me that was a productive testing session as I've explored those fields. 

I believe that Telegram should give the opportunity to bots to moderate groups, this will help administrators to moderate them effectively. 

Nonetheless, bot are simply powerful to add features to group and make them more interactive, fun and enjoyable. 

If you have ideas, please share them below so we can discuss and improve the knowledge about bots. 

If you liked this blog post please make sure to share it with your friends and social media. 

Furthermore if you want to join my group and be greeted by my bot and the community, [join now](https://telegram.me/joinchat/AEis8D1UWPDa9sdeyzJQ6w)! 
