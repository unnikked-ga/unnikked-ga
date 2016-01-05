---
layout:     post
title:      "Understanding Telegram inline bots"
subtitle:   "Let's create a bot that search via DuckDuckGo"
date:       2016-01-05 22:55:05
author:     "Nicola Malizia"
tags: ["telegram", "php"]

header-img: "data/cover/telegram-background.jpg"

twitter-card: true
twitter-image: "https://lh3.googleusercontent.com/-sia8Z4hiCQc/Vow7zOoRqPI/AAAAAAAAALQ/KCC04_gao0o/s0/understanding-telegram-inline-bots.jpg"

open-graph: true
open-graph-image: "https://lh3.googleusercontent.com/-sia8Z4hiCQc/Vow7zOoRqPI/AAAAAAAAALQ/KCC04_gao0o/s0/understanding-telegram-inline-bots.jpg"

---

On 4 January Telegram introduced a new feature for [The Bot Platform](/getting-started-with-telegram-bots) - [Inline Bots](/blog/inline-bots).  

With this new feature you can interact with bots inline in your chat, besides having them as members! This is a nice feature since you can do a lot of stuff now without cluttering the chat. 

As examples check out: 
- [@gif](https://telegram.me/gif) to search for gifs on Giphy,
- [@vid](https://telegram.me/vid) to search videos on Youtube.
- [@pic](https://telegram.me/pic) to search images on Yandex. 
- [@bing](https://telegram.me/bing) to search images on Bing.
- [@wiki](https://telegram.me/wiki) to search for terms on Wikipedia.
- [@imdb](https://telegram.me/imdb) to search movies infos on IMDB
- [@bold](https://telegram.me/bold) to convert texts to bold, italic and formatted

![example](https://lh3.googleusercontent.com/-_W26_IqV-8c/VownlckBdVI/AAAAAAAAAKU/DHruQcn5128/s0/photo_2016-01-05_21-28-35.jpg)

##What's new

In case you have not done yet I strongly recommend to read my first article for [understanding](/getting-started-with-telegram-bots) how bots work.

First of all you need to enable the `inline mode` on your bot using the **@BotFather** as [I've explained in my first article](/getting-started-with-telegram-bots).

![set inline mode](https://lh3.googleusercontent.com/-PMt1vwx1kpQ/VowpdPkH-0I/AAAAAAAAAKk/jTC9ZchClZM/s0/photo_2016-01-05_21-36-49.jpg) 

> This step must be done via @BothFather

From the code point of view there are some changes. For the `update` object there is a new field called `inline_query`, to process inline queries you must process this field. 

The `InlineQuery` objects comprehend of the following fields. 

<table>
<thead>
<tr>
  <th>Field</th>
  <th>Type</th>
  <th>Description</th>
</tr>
</thead>
<tbody><tr>
  <td>id</td>
  <td>String</td>
  <td>Unique identifier for this query</td>
</tr>
<tr>
  <td>from</td>
  <td>User</td>
  <td>Sender</td>
</tr>
<tr>
  <td>query</td>
  <td>String</td>
  <td>Text of the query</td>
</tr>
<tr>
  <td>offset</td>
  <td>String</td>
  <td>Offset of the results to be returned, can be controlled by the bot</td>
</tr>
</tbody></table>


The `InlineQueryResult`  represents one result of an inline query. Telegram clients currently support results of 5 types. 

The `ChosenInlineResult` represents a result of an inline query that was chosen by the user and sent to their chat partner.

The only method added is `answerInlineQuery`, Use this method to send answers to an inline query. On success, `True` is returned.
No more than **50** results per query are allowed.

<table>
<thead>
<tr>
  <th>Parameters</th>
  <th align="left">Type</th>
  <th>Required</th>
  <th align="left">Description</th>
</tr>
</thead>
<tbody><tr>
  <td>inline_query_id</td>
  <td align="left">String</td>
  <td>Yes</td>
  <td align="left">Unique identifier for the answered query</td>
</tr>
<tr>
  <td>results</td>
  <td align="left">Array of InlineQueryResult</td>
  <td>Yes</td>
  <td align="left">A JSON-serialized array of results for the inline query</td>
</tr>
<tr>
  <td>cache_time</td>
  <td align="left">Integer</td>
  <td>Optional</td>
  <td align="left">The maximum amount of time in seconds that the result of the inline query may be cached on the server. Defaults to 300.</td>
</tr>
<tr>
  <td>is_personal</td>
  <td align="left">Boolean</td>
  <td>Optional</td>
  <td align="left">Pass True, if results may be cached on the server side only for the user that sent the query. By default, results may be returned to any user who sends the same query</td>
</tr>
<tr>
  <td>next_offset</td>
  <td align="left">String</td>
  <td>Optional</td>
  <td align="left">Pass the offset that a client should send in the next query with the same text to receive more results. Pass an empty string if there are no more results or if you don‘t support pagination. Offset length can’t exceed 64 bytes.</td>
</tr>
</tbody></table>

For and in-depth explanation you may want to check out the official [API documentation](https://core.telegram.org/bots/api#inline-mode). 

## Inline DuckDuckGo searches

So yeah, what about some code? To play with this new feature I've created a bot that let's you make searches inline on DuckDuckGo, my preferite search engine (yes I don't use Google search). 

For simplicity and speed of development I decided to base the bot on the official example bot that you can find [on the FAQ page](https://core.telegram.org/bots/samples/hellobot).  

What the bot do: 
- it only process inline queries.
- it scrapes the result of the query on DuckDuckGo using the [basic html format](https://duckduckgo.com/html). 
- returns the result of scraped query to the user and let the Telegram servers cache it (so the bot will not hit the search rate limit, hopefully). 

> You may find that the code is not robust and it's not complete, just note that this is only for educational purpose, so yea I did not care about edge cased. 

As you may notices, only searches are supported; no instant answers, no bangs. 

## Show me the code!

I've made all the edits to the end of the example file. 

```
if (isset($update["message"])) {
  processMessage($update["message"]);
} else if (isset($update["inline_query"])) {
    $inlineQuery = $update["inline_query"];
    $queryId = $inlineQuery["id"];
    $queryText = $inlineQuery["query"];
    
    if (isset($queryText) && $queryText !== "") {
      apiRequestJson("answerInlineQuery", [
        "inline_query_id" => $queryId,
        "results" => queryDuckDuckGo($queryText),
        "cache_time" => 86400,
      ]);
    } else {
      apiRequestJson("answerInlineQuery", [
        "inline_query_id" => $queryId,
        "results" => [
          [
            "type" => "article",
            "id" => "0",
            "title" => "Unnikked Blog",
            "message_text" => "I'm the author of this bot, please visit my blog for more https://unnikked.ga",
          ],  
        ]
      ]);
    }
}
```

Note the `else if` statement, here is where I check for the `inline_query` field, grab the query and process it. 

When replying the `inline_query_id` must be unique, since I don't know what it could mean for the server I noticed that passing again the `id` of the query works fine, unfortunately I'm not sure why this filed is needed. 

The `results` field is made by the `queryDuckDuckGo($queryText)` function that looks like this:

```
function queryDuckDuckGo($query) {
  
  $content = file_get_contents('https://duckduckgo.com/html/?q=' . urlencode($query));
  
  if(!$content) return [[
    "type" => "article",
    "id" => "0",
    "title" => "Service unavailable",
    "message_text" => "Service unavailable",
  ]];
  
  $crawler = new Crawler($content);
  
  $results = $crawler->filter(".links_main .large");
  foreach ($results as $result) {
    $titles[] = trim($result->textContent);
  }
  
  $results = $crawler->filter(".links_main .snippet");
  foreach ($results as $result) {
    $snippets[] = trim($result->textContent);
  }
  
  $results = $crawler->filter(".links_main .url");
  foreach ($results as $result) {
    $urls[] = trim($result->textContent);
  }
  
  foreach (range(0, count($titles) - 1) as $i) {
    $collection[] = [
      "type" => "article",
      "id" => "$i",
      "title" => "$titles[$i]",
      "message_text" => "$titles[$i]\n$snippets[$i]\n$urls[$i]",
    ];
  }
  
  return $collection;
}
```

That's all there is to it! 

![enter image description here](https://lh3.googleusercontent.com/-eIZw8Q-TQQQ/Vowve_iOZpI/AAAAAAAAAK8/l20ignCNC1w/s0/photo_2016-01-05_22-02-29.jpg "inlineduckbot")

By the way the bot name is [@inlineduckbot](http://telegram.me/inlineduckbot). 