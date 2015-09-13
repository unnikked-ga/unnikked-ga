---
layout:     post
title:      "A guestbook using Lumen and Vue.js"
subtitle:   ""
date:       2015-06-20 16:58:00
author:     "Nicola Malizia"
tags: ["javascript", "php"]

twitter-card: true
twitter-image: "http://i.imgur.com/sCDCwLJ.png"

open-graph: true
open-graph-image: "http://i.imgur.com/sCDCwLJ.png"
---

Every day there is a new framework that enable us to write web apps straightforwardly and easily.  

I admit, I'm not a proficient web app developer but I like to challenge myself to test and learn new technologies. 

What I will discuss in this blog post is a simple implementation of a [single page application](https://en.wikipedia.org/wiki/Single-page_application) using as backend [Lumen](http://lumen.laravel.com/) and as frontend [Vue.js](http://vuejs.org/). 

Lumen is a micro-framework created by Taylor Otwell, the designer of Laravel, one of the most popular PHP framework of these days. 

Lumen is based on Laravel but it aims to achieve faster performace and it does not have many features as the big brother Laravel, yet it is powerfull enough to build microservices and [REST](http://rest.elkstein.org/) APIs.  

<blockquote>Vue.js is a library for building modern web interfaces. 
It provides data-reactive components with a simple and flexible API.</blockquote>

As I said, I'm not familiar with web tecnologies but with these two tools I was able to investigate into something that always interested me. 

The concepts that we will see in this blog post are: 

0. Defining of a simple Rest-Like API. 
0. Designing the backend using Lumen.
0. Designing of the frontend using Vue.js

## The final result

For the impatient reader here it is the final result that we will achieve.

<p align="center"><img class="img-responsive img-thumbnail" src="http://i.imgur.com/lXQxsQq.png"\></p>

## Definition of API

To keep things simple the API comprehend only basic CRUD (not even all of them) operations. 

- `GET /api/comment` to retrieve all comments posted
- `POST /api/comment` to create a new comment
- `DELETE /api/comment/{id}` to delete a specific comment `id` 

## Backend

I assume that you have already configured a fresh installation of Lumen on your local machine or an environment like [Homestead](http://laravel.com/docs/5.1/homestead).

The first thing we need to do is to enable some features by editing `bootstrap/app.php`. 

Let's uncomment this three lines: 

```php
Dotenv::load(__DIR__.'/../');
$app->withFacades();
$app->withEloquent();
```

So we can now load environment configuration through the `.env` files and we will be able to use Facades and the Eloquent ORM. 

Our app is simple but requires a single table, let's call it `comments`. First thing first let's create a migration using the command line utility `artisan`. 

```bash
php artisan make:migration create_comments_table --create comments
```

It will create a migration class under `database/migrations` folder. We need to add two columns to the generated boilerplate code: a `string` for the `author` and the `text`. 

```php
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateCommentsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('comments', function (Blueprint $table) {
            $table->increments('id');
            $table->string('text');
            $table->string('author');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('comments');
    }
}
```

To apply this migration let's run: 

```bash
php artisan migrate
```

Now that we have the `comments` table we need a model to represent a `Comment`, under the `app/` create a `Comment` class. 

```php
<?php namespace App;

use Illuminate\Database\Eloquent\Model;

class Comment extends Model 
{
	protected $fillable = [
		'author', 
		'text'
	];
}
```

The `protected $fillable` filed is needed in order to enable the ORM to do mass assignments during a creation of a record, you may want to read the official documentation to understand this [behaviour](http://laravel.com/docs/5.1/eloquent#mass-assignment). 

Now under the `app/Http/Controllers` create the `CommentController` class, this class will handle our simple business logic. 

```php
<?php namespace App\Http\Controllers;

use Laravel\Lumen\Routing\Controller as BaseController;
use Illuminate\Http\Request;

use App\Comment;

class CommentController extends BaseController 
{

    /**
     * Send back all comments as JSON
     *
     * @return Response
     */
    public function index()
    {
        return response()->json(Comment::get());
    }
    
    /**
     * Store a newly created resource in storage.
     *
     * @return Response
     */
    public function store(Request $request)
    {
        return response()->json(Comment::create($request->all()));
    }
    
    /**
     * Remove the specified resource from storage.
     *
     * @param  int  $id
     * @return Response
     */
    public function destroy($id)
    {
        return response()->json(Comment::destroy($id));
    }   
}
```

A note to the implementation:  to keep things simple and to show the basic concept that is under this project we will not pay attention to the input received by the client (the framework sanitizes it automatically) and we will assume that every request will succeed (`200` Status Code Response) . 

As you can see the implementation is very simple, thanks to the convenient Eloquent ORM. 

The last step of designing the backed is to wire up the api routes defined before, [routes](http://lumen.laravel.com/docs/routing) must be specified into the `app/Http/routes.php` file. 

```php
<?php

$app->get('/', function() use ($app) {
    return view('index');
});


$app->get('api/comment', 'CommentController@index');
$app->post('api/comment', 'CommentController@store');
$app->delete('api/comment/{id}', 'CommentController@destroy');
```

We will implement the frontend into the `resources/index.php` file.

## Frontend

We will use as the frontent Vue.js plus jQuery to make ajax calls to our backed. 

I choose Vue.js because I thinks it's simpler to use respect to other frameworks and since I'm a beginner with Javascript I found it very intuitive. 

As you may have already imagined we will use bootstrap to style the page, this is the main content of the web page. 

```html
 <body class="container" id="guestbook"> 
        <div class="col-md-8 col-md-offset-2">
            <div class="jumbotron">
                <h2>Lumen and Vue.js Single Page Application</h2>
                <h4>Simple Guestbook</h4>
            </div>
            
            <form v-on="submit: onCreate">
                <div class="form-group">
                    <input type="text" class="form-control input-sm" name="author" v-model="author" placeholder="Name">
                </div>
            
                <div class="form-group">
                    <input type="text" class="form-control input-sm" name="text" v-model="text" placeholder="Put here your text">
                </div>
            
                <div class="form-group text-right">   
                    <button type="submit" class="btn btn-primary btn-lg">Submit</button>
                </div>
            </form>
                       
            <div class="comment" v-repeat="comment: comments">
                <h3>Comment #{{ comment.id }} <small>by {{ comment.author }}</h3>
                <p>{{ comment.text }}</p>
                <p><span class="btn btn-primary text-muted" v-on="click: onDelete(comment)">Delete</span></p>
            </div>
        </div>
```

Few comments on the code listed. 

The `id` attribute attached to the `<body>` tag is needed by Vue.js as explained [here](http://vuejs.org/guide/#View). 

The `v-on="submit: onCreate"` attribute is used by Vue.js in order to call the function to perform whenever a form is submitted. 

The `v-model` attribute is used to perform the two way binding feature of Vue.js, the feature that I like most. 

Whenever you type something into the form field automatically the inner model data structure (we will se it later) is updated. Since it is a two way binding whenever the inner model data structure is modified the DOM of the webpage is automatically updated. 

The `v-repeat="comment: comments"` attribute is used to cycle a collection. 

If you want to understand better this framework I strongly suggest to check this [series](https://laracasts.com/series/learning-vuejs) on laracast. 

The code used to perform the frontend logic is simple and intuitive in my opinion. 

```javascript
new Vue({ 
                el: '#guestbook',

                data: {
                    comments: [],
                    text: '',
                    author: ''
                },

                ready: function() {
                    this.getMessages();
                },

                methods: {
                    getMessages: function() {
                        $.ajax({
                            context: this,
                            url: "/api/comment",
                            success: function (result) {
                                this.$set("comments", result) 
                            }
                        })
                    },

                    onCreate: function(e) {
                        e.preventDefault()
                        $.ajax({
                            context: this,
                            type: "POST",
                            data: {
                                author: this.author,
                                text: this.text
                            },
                            url: "/api/comment",
                            success: function(result) {
                                this.comments.push(result);
                                this.author = ''
                                this.text = ''
                            }
                        })                        
                    },

                    onDelete: function (comment) {
                        $.ajax({
                            context: comment,
                            type: "DELETE",
                            url: "/api/comment/" + comment.id,
                        })
                        this.comments.$remove(comment);
                    }
                }
            })
```

The `el` property needs to tell to Vue.js at which node of the DOM it has to "attach". 

The `data` property is the inner data structure that Vue.js keep synchronized with the DOM, each key corresponds to a `v-model` binding. 

The `ready` property is used to define actions whenever a page is loaded, in this case we will load all the comments from the backend. 

The `methods` property is used to define functions used by this "component": 

-  `getMessages` is used to retrieve all the messages from the backend.
- `onCreate` is used to create a new comment making an AJAX call
- `onDelete` is used as well to perform a comment deletion by using an AJAX call

Please note that it is sufficient to update the `data` property in order to reflect the changes to the webpage. 

## Conclusion
This is certainly a simple and basic (stupid) example, but it allowed me to understand a lot of stuff needed to design a dynamic website. I hope that it was useful also for you. 

You may want to see a live version of the app [here](http://guestbook.unnikked.ga) and check out the code on [github](https://github.com/unnikked-ga/guestbook).

I suggest to reproduce this blog post manually so you can touch by hand the steps. 

Feel free to drop a comment below and let me thing about the technologies used, I know that this could have been implemented in several ways.  