---
layout:     post
title:      "How to create your custom Telegram bot using the long polling technique"
subtitle:   "Bot platform, have you any update for me ?"
date:       2015-07-05 16:10:00
author:     "Nicola Malizia"
tags: ["java", "telegram"]

header-img: "data/cover/telegram-background.jpg"

twitter-card: true
twitter-image: "http://i.imgur.com/aSGsKbG.png"

open-graph: true
open-graph-image: "http://i.imgur.com/aSGsKbG.png"
---

In this blog post we'll see the basics of designing a [Telegram bot](https://telegram.org/blog/bot-revolution) using the long polling technique. 

<blockquote><p>Long polling is itself not a true push; long polling is a variation of the traditional polling technique, but it allows emulating a push mechanism under circumstances where a real push is not possible, such as sites with security policies that require rejection of incoming HTTP/S Requests.</p>

<p>With long polling, the client requests information from the server exactly as in normal polling, except it issues its HTTP/S requests (polls) at a much slower frequency. If the server does not have any information available for the client when the poll is received, instead of sending an empty response, the server holds the request open and waits for response information to become available. Once it does, the server immediately sends an HTTP/S response to the client, completing the open HTTP/S Request. In this way the usual response latency (the time between when the information first becomes available and the next client request) otherwise associated with polling clients is eliminated.</p>

From <a href="https://en.wikipedia.org/wiki/Push_technology">wikipedia</a>.
</blockquote>

Despite there exists already plenty of [frameworks](https://www.reddit.com/r/TelegramBots/comments/3bsec7/unofficial_collection_of_api_wrappers/) for the most common languages, I want to show you just a simple example on how to use the `getUpdates` API method, since it was requested on my [first article](getting-started-with-telegram-bots) about telegram bots.

In this example we are going to create a simple bot that responds to three simple commands. 

- `/start` - provides some basic info. 
- `/echo` - replies back everything we write as the command parameters. 
- `/toupper` - converts the given text into uppercase. 

Just to remind you, if you want to see a bot using the webhook method check my blog post about a [Lisp bot evaluator](how-i-coded-a-telegram-bot-that-evaluates-lisp-code). 

In this example I will use the [Unirest for Java](http://unirest.io/java.html) library to make HTTP calls easily. 


## The TelegramBot class
This is the skeleton of our TelegramBot class. 

```java
public final class TelegramBot {
	private final String endpoint = "https://api.telegram.org/";
	private final String token;

	public TelegramBot(String token) {
		this.token = token;
	}

	public HttpResponse<JsonNode> sendMessage(Integer chatId, String text) throws UnirestException {
		return Unirest.post(endpoint + token + "/sendMessage")
				.field("chat_id", chatId)
				.field("text", text)
				.asJson();
	}

	public HttpResponse<JsonNode> getUpdates(Integer offset) throws UnirestException {
		return Unirest.post(endpoint + token + "/getUpdates")
				.field("offset", offset)
				.asJson();
	}

	public void run() throws UnirestException {
		// ...
	}
}
```

As it is an example we just focus on retrieving updates and sending text messages, so there are respectively two methods: `sendMessage` and `getUpdates`.

When using the `getUpdates` API method you must care of the `offset` parameter, let's see what the documentation tells about it. 

>Identifier of the first update to be returned. Must be greater by one than the highest among the identifiers of previously received updates. By default, updates starting with the earliest unconfirmed update are returned. An update is considered confirmed as soon as getUpdates is called with an offset higher than its <code>update_id</code>.

So in order to confirm requests we must provide an `offset` greater than the last `update_id` received. 

### Long polling in action

It might sound complicated but let's see how the code should look like. 

```java
public void run() throws UnirestException {
	int last_update_id = 0; // last processed command

	HttpResponse<JsonNode> response;
	while (true) {
		response = getUpdates(last_update_id++);

		if (response.getStatus() == 200) {
			JSONArray responses = response.getBody().getObject().getJSONArray("result");
			if (responses.isNull(0)) continue;
			else last_update_id = responses
					.getJSONObject(responses.length() - 1)
					.getInt("update_id") + 1;

			for (int i = 0; i < responses.length(); i++) {
				// process commands
			}
		}
	}
}
```

At the beginning the `last_update_id` variable is set to `0`, this produces the effect to retrieve the last updates available if any (when you start the bot). 

Then we check if there are any updates (i.e. the JSONArray is not empty) and we update the `last_update_id` variable with the last update available into the array. 

Ultimately we loop through this array of updates and process each element. 

Just note that everything is inside a endless loop. 

### Process commands

It should be clear how each command needs to be processed, let's see my example. 

```java
for (int i = 0; i < responses.length(); i++) {
	JSONObject message = responses
		.getJSONObject(i)
		.getJSONObject("message");

	int chat_id = message
		.getJSONObject("chat")
		.getInt("id");

	String username = message
		.getJSONObject("chat")
		.getString("username");

	String text = message
		.getString("text");

	if (text.contains("/start")) {
		String reply = "Hi, this is an example bot\n" +
			"Your chat_id is " + chat_id + "\n" +
			"Your username is " + username;
		sendMessage(chat_id, reply);
	} else if (text.contains("/echo")) {
		sendMessage(chat_id, "Received " + text);
	} else if (text.contains("/toupper")) {
		String param = text.substring("/toupper".length(), text.length());
		sendMessage(chat_id, param.toUpperCase());
	}
}
```

I don't want to be specific here because you can organize the method of processing commands in several ways, just plan your design and implement it. 

## Some considerations

By quoting directly the API documentation. 

<blockquote>
<ol>
<li>This method will not work if an outgoing webhook is set up.</li>
<li>In order to avoid getting duplicate updates, recalculate offset after each server response.</li>
</ol>
</blockquote>

We covered the point `2.` during this blog post.

## Conclusion

I hope that this simple blog post can help you understand how organize your code in order to implement you custom Telegram bot. 

Feel free to comment here and provide your solution, I'd love to hear you. 