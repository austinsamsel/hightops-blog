---
title: Building Capote Meteor & Mocha TDD
date: 2015-10-03 21:34 UTC
tags: meteor, mocha, tdd
---

Here's a long post I did as I was first working on testing my Meteor apps using Mocha and Chai. Don't take anything here as your one source of truth. I think this is a good post if you're looking to get started and want to see what some basic tests look like.

## Building Capote: Meteor & Mocha BDD (Part 1)

You know what's annoying? Testing your app by physically logging in and going through your app to make sure everything works. Also, it takes a lot of time. Also, its scary. If I was doing that while working for a client, I'd really be stressed.

That's why there's BDD. Writing tests as you write your software. It helps your software suck less, gives you a little more peace of mind, and even though it means writing extra code, there's a really good chance it will save you a lot of time overall.

When you're learning a framework, you don't get much exposure to testing. Usually your learning resources are focused on covering the framework itself. They don't cover tests because there's so many different ways to do it. With Meteor there's a bunch of suites to test your app: cucumber, jasmine, mocha, etc.

If you're looking to get started with BDD or want to see some examples of using Mocha in BDD to build a Meteor app, this tutorial is for you.

If you just want to see the code for this post, you can checkout the [GitHub repository](https://github.com/austinsamsel/capote/tree/part-1).

## Resources

##### Meteor BDD bible:

[www.meteortesting.com](www.meteortesting.com)

##### Also, especially relevant:

Mocha documentation: [https://mochajs.org/](https://mochajs.org/)
Chai documentation: [http://chaijs.com/](http://chaijs.com/ )

Because while we're using Mocha we'll use Chai for assertions.

##### other Meteor mocha tutorials:

[http://www.webtempest.com/meteor-js-testing-mocha-tutorial](http://www.webtempest.com/meteor-js-testing-mocha-tutorial)
[https://github.com/meteor-velocity/velocity-examples](https://github.com/meteor-velocity/velocity-examples) (technically just examples)

## About our app

In this tutorial, we'll build an app that tracks a user's daily word count. I've gotten into the habit of waking up in the morning and writing at least 500 words a day. It's a great way to empty out my mind and free myself up for a little more creativity. We're going to name this app Capote, after Truman Capote. Whose that? Doesn't matter. But if you don't know, he was a writer from a long time ago and he probably wrote at least 500 words a day (I have no idea, but probably!). Also his last name sounds kind of cool for an app. So there it is, Capote.

Here are the features we'll build:

* A listing of posts.
* Create and edit posts.
* Daily goal set by user.
* Count words for each post.

## Set up

For this first post, we'll set up our test suite and write our first test. We'll want to create a sample list of posts so we have some data to work with. First things first, we'll generate a new app.

	$ meteor create capote

Then let's trash the default files *capote.css*, *capote.html* and *capote.js*

Let's also add in our testing package:

	$ meteor add mike:mocha

Let's set up our tests directory structure like this:

	tests/
	- mocha/
	- - client/
	- - server/

This type of pattern is required for velocity to read the tests and you'll need to use it while testing in Meteor with Cucumber, Jasmine, too. What is velocity? Velocity is Meteor's reactive test-runner and it supports a whole bunch of testing frameworks.

## Our first test

Let's start with the server. There's just a few parts to each post we need right now: a timestamp, a title, and the body of the post.

We'll write our first test. Any mocha tests are first going to be wrapped in the following:

```javascript
// tests/mocha/server/server.js
if (!(typeof MochaWeb === 'undefined')){
  MochaWeb.testOnly(function(){
    /* some test code*/
  });
}
```

There isn't much documentation I can find about why we ought to do this... It looks like it helps to only run tests in the dev or mirror environment, but I'm honestly not sure. I've been able to run tests without it. For now, I include it because the Velocity developers do it.

Moving along. We'll write our first test, which just makes sure we have at least one post in the database.

```javascript
describe("Server initialization", function(){
  it("should insert posts into the database on server startup", function(){
    chai.assert(Posts.find().count() > 0);
  });
});
```


If you run meteor now you'll see that velocity pops up. In your logs you'll also see a message saying, "You can see the mirror logs at: tail -f /path/to/your/logs". You can also pop open another command line window and paste in *tail -f /path/to/your/logs* and get some extra insight.

Both the logs and our UI tell us the same thing. "1 test failed" and "Posts is not defined." I love BDD. It tells us what comes next. We can go ahead and define Posts now.

```javascript
// model/posts.js
Posts = new Mongo.Collection('posts');
```

When the tests run again, we're told we have a failed "Server initialization." Let's add some posts on server start up.

```javascript
// server/app.js
if (Meteor.isServer) {
  Meteor.startup(function () {
    if (Posts.find().count() === 0) {
      Posts.insert({
        title: "Neutra messenger bag",
        content: "Tousled forage trust fund readymade Neutra messenger bag. Drinking vinegar chia Marfa, vegan messenger bag disrupt Wes Anderson try-hard. Small batch scenester raw denim synth cronut cornhole, iPhone try-hard single-origin."
      });
      Posts.insert({
        title: "fatback filet mignon",
        content: "Bacon ipsum dolor amet alcatra turkey shank cupim corned beef brisket chuck boudin tri-tip t-bone kevin fatback filet mignon. Short loin tongue short ribs."
      });
      Posts.insert({
        title: "know what I'm sayin'",
        content: "You see? It's curious. Ted did figure it out - time travel. And when we get back, we gonna tell everyone. How it's possible, how it's done, what the dangers are. But then why fifty years in the future when the spacecraft encounters a black hole does the computer call it an 'unknown entry event'?"
      });
    }
  });
}
```

I used [some](http://hipsum.co/) [pretty](https://baconipsum.com/) [sweet](http://slipsum.com/) ipsum generators to create some sample content. We're inserting three posts with a title and content only if there are no posts in the database upon start up.

If you check the logs or the velocity helper in the UI, you'll notice that our first test passed. Great!

## Picking up the pace

I'm going to cap this post here, but in the next posts as we build out this app, I'm going to pick up the pace with these tests and we'll get a whole bunch done, real fast.

You can grab the completed code for this post [here](https://github.com/austinsamsel/capote/tree/part-1).


# Building Capote: Meteor & Mocha BDD (Part 2)

In the last post, we set up our app, fixtures, and got our first test to pass. You can grab the code from the last chapter [here](https://github.com/austinsamsel/capote/tree/part-1). If you want to review the last post, you can [visit it here](http://hightopsnyc.com/blog/building-capote.html).

## Testing UI Objects and Sort Order

It's time to actually show our posts in the UI to our users. In the first test, I want to make sure there's at least one post displaying by checking if the css class that's associated with the post gets printed to the page. Then I'll know there's a post on the page, no matter the content of the post. Then I want to test the order of the posts so that the most recent posts show up first in the list.

```javascript
// tests/mocha/client/client.js

if (!(typeof MochaWeb === 'undefined')){
  MochaWeb.testOnly(function(){

    describe("Posts", function(){
      it("shows a post", function(){
        Meteor.flush();
        chai.assert.typeOf($(".title:eq(0)"), 'object');
      })
      it("should show my latest post, first.", function(){
        Meteor.flush();
        chai.assert.equal($(".title:eq(0)").html(), "Neutra messenger bag");
      });
    });

  });
}
```

After the tests, we'll get to writing the actual code. First we'll create a subscription so our template can find the posts in the database. We don't need to create a publication since we still have the autopublish package installed by default (and we'd be sure to remove it if this app was going into production). We also set up our templates to display the posts.

```javascript
// client/posts.js
if (Meteor.isClient) {
  Template.body.helpers({
    posts: function () {
      return Posts.find({}, {sort: {createdAt: -1}});
    }
  });
}

// client/index.html
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Capote - track your words per day</title>
</head>

<body>
  <div class="container">
    {{#each posts}}
      {{> post}}
    {{/each}}
  </div>
</body>

<template name="post">
  <div class="title">{{title}}</div>
  <div class="content">{{content}}</div>
</template>
```

## Testing Timestamps

Before we can think about measuring the daily word count streak, we need to include a timestamp with each post. The following is a test to check for that.

```javascript
// tests/mocha/client/client
...
  it("should show a clean timestamp", function(){
    chai.assert.equal($(".timestamp:eq(1)").html(), "01-03-2015");
  });
...
```

We'll update our fixtures by adding a date/time for when our posts are created. We'll also include the new field in our templates.

```javascript
// server/app.js

if (Meteor.isServer) {
  Meteor.startup(function () {
    if (Posts.find().count() === 0) {
      Posts.insert({
        title: "Neutra messenger bag",
        content: "Tousled forage trust fund readymade Neutra messenger bag. Drinking vinegar chia Marfa, vegan messenger bag disrupt Wes Anderson try-hard. Small batch scenester raw denim synth cronut cornhole, iPhone try-hard single-origin.",
        createdAt: new Date(2015,0,4) // new timestamp
      });
      Posts.insert({
        title: "fatback filet mignon",
        content: "Bacon ipsum dolor amet alcatra turkey shank cupim corned beef brisket chuck boudin tri-tip t-bone kevin fatback filet mignon. Short loin tongue short ribs.",
        createdAt: new Date(2015,0,3) // new timestamp
      });
      Posts.insert({
        title: "know what I'm sayin'",
        content: "You see? It's curious. Ted did figure it out - time travel. And when we get back, we gonna tell everyone. How it's possible, how it's done, what the dangers are. But then why fifty years in the future when the spacecraft encounters a black hole does the computer call it an 'unknown entry event'?",
        createdAt: new Date(2015,0,1) // new timestamp
      });
    }
  });
}

// client/index.html

<template name="post">
  <!-- our new timestamp below -->
  <div class="timestamp">{{createdAt}}</div>
  <div class="title">{{title}}</div>
  <div class="content">{{content}}</div>
</template>
```

To see whether this passes or fails, we'll manually reset our app by running *meteor reset && meteor* in the command line. Mocha's mirror derives itself from the application your developing, so the fixtures won't reset on their own. The idea is that you wouldn't want to reset the mirror database each time before running tests because it would take too long.

Our test still fails because, by default, Meteor prints out the full timestamp, like: *Sat Jan 03 2015 00:00:00 GMT-0500 (EST)*

We'll need to make use of [Moment JS](http://momentjs.com/) to parse our timestamps. Run the following in the command line to add the moment package to our app.

	$ meteor add moments:moment

Then we'll create a helper to parse the timestamp. It takes the default timestamp as an argument. Next we'll update our template with the cleanDate helper, effectively passing in that long default timestamp and returning our clean timestamp.

```javascript
// client/helper.js
Template.registerHelper('cleanDate', function(date) {
	return moment(date).format('MM-DD-YYYY');
});


// client/index.html
<div class="timestamp">{{cleanDate createdAt}}</div>
```

Now our tests all pass.

## Wrapping Up

If you have any questions or you want me to break down the steps even further, please let me know. If you want to check out the code for this section, you can grab it from [GitHub](https://github.com/austinsamsel/capote/tree/part-2). In the next post we'll write tests for creating and deleting posts.

# Building Capote: Meteor & Mocha BDD (Part 2)

In the last post, we set up our app, fixtures, and got our first test to pass. You can grab the code from the last chapter [here](https://github.com/austinsamsel/capote/tree/part-1). If you want to review the last post, you can [visit it here](http://hightopsnyc.com/blog/building-capote.html).

## Testing UI Objects and Sort Order

It's time to actually show our posts in the UI to our users. In the first test, I want to make sure there's at least one post displaying by checking if the css class that's associated with the post gets printed to the page. Then I'll know there's a post on the page, no matter the content of the post. Then I want to test the order of the posts so that the most recent posts show up first in the list.

```javascript
// tests/mocha/client/client.js

if (!(typeof MochaWeb === 'undefined')){
  MochaWeb.testOnly(function(){

    describe("Posts", function(){
      it("shows a post", function(){
        Meteor.flush();
        chai.assert.typeOf($(".title:eq(0)"), 'object');
      })
      it("should show my latest post, first.", function(){
        Meteor.flush();
        chai.assert.equal($(".title:eq(0)").html(), "Neutra messenger bag");
      });
    });

  });
}
```


After the tests, we'll get to writing the actual code. First we'll create a subscription so our template can find the posts in the database. We don't need to create a publication since we still have the autopublish package installed by default (and we'd be sure to remove it if this app was going into production). We also set up our templates to display the posts.

```javascript
// client/posts.js
if (Meteor.isClient) {
  Template.body.helpers({
    posts: function () {
      return Posts.find({}, {sort: {createdAt: -1}});
    }
  });
}

// client/index.html
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Capote - track your words per day</title>
</head>

<body>
  <div class="container">
    {{#each posts}}
      {{> post}}
    {{/each}}
  </div>
</body>

<template name="post">
  <div class="title">{{title}}</div>
  <div class="content">{{content}}</div>
</template>
```

## Testing Timestamps

Before we can think about measuring the daily word count streak, we need to include a timestamp with each post. The following is a test to check for that.

```javascript
// tests/mocha/client/client
...
  it("should show a clean timestamp", function(){
    chai.assert.equal($(".timestamp:eq(1)").html(), "01-03-2015");
  });
...
```

We'll update our fixtures by adding a date/time for when our posts are created. We'll also include the new field in our templates.

```javascript
// server/app.js

if (Meteor.isServer) {
  Meteor.startup(function () {
    if (Posts.find().count() === 0) {
      Posts.insert({
        title: "Neutra messenger bag",
        content: "Tousled forage trust fund readymade Neutra messenger bag. Drinking vinegar chia Marfa, vegan messenger bag disrupt Wes Anderson try-hard. Small batch scenester raw denim synth cronut cornhole, iPhone try-hard single-origin.",
        createdAt: new Date(2015,0,4) // new timestamp
      });
      Posts.insert({
        title: "fatback filet mignon",
        content: "Bacon ipsum dolor amet alcatra turkey shank cupim corned beef brisket chuck boudin tri-tip t-bone kevin fatback filet mignon. Short loin tongue short ribs.",
        createdAt: new Date(2015,0,3) // new timestamp
      });
      Posts.insert({
        title: "know what I'm sayin'",
        content: "You see? It's curious. Ted did figure it out - time travel. And when we get back, we gonna tell everyone. How it's possible, how it's done, what the dangers are. But then why fifty years in the future when the spacecraft encounters a black hole does the computer call it an 'unknown entry event'?",
        createdAt: new Date(2015,0,1) // new timestamp
      });
    }
  });
}

// client/index.html

<template name="post">
  <!-- our new timestamp below -->
  <div class="timestamp">{{createdAt}}</div>
  <div class="title">{{title}}</div>
  <div class="content">{{content}}</div>
</template>
```

To see whether this passes or fails, we'll manually reset our app by running *meteor reset && meteor* in the command line. Mocha's mirror derives itself from the application your developing, so the fixtures won't reset on their own. The idea is that you wouldn't want to reset the mirror database each time before running tests because it would take too long.

Our test still fails because, by default, Meteor prints out the full timestamp, like: *Sat Jan 03 2015 00:00:00 GMT-0500 (EST)*

We'll need to make use of [Moment JS](http://momentjs.com/) to parse our timestamps. Run the following in the command line to add the moment package to our app.

	$ meteor add moments:moment

Then we'll create a helper to parse the timestamp. It takes the default timestamp as an argument. Next we'll update our template with the cleanDate helper, effectively passing in that long default timestamp and returning our clean timestamp.

```javascript
// client/helper.js
Template.registerHelper('cleanDate', function(date) {
	return moment(date).format('MM-DD-YYYY');
});


// client/index.html
<div class="timestamp">{{cleanDate createdAt}}</div>
```

Now our tests all pass.

## Wrapping Up

If you have any questions or you want me to break down the steps even further, please let me know. If you want to check out the code for this section, you can grab it from [GitHub](https://github.com/austinsamsel/capote/tree/part-2). In the next post we'll write tests for creating and deleting posts.

# Building Capote: Meteor & Mocha BDD (Part 4)

Last week we set up the functionality for users to create posts and delete them. In this part, we're going to build our word count feature.

You can grab the code from the last chapter [here](https://github.com/austinsamsel/capote/tree/part-3). If you want to review the last post, you can [visit it here](http://hightopsnyc.com/blog/building-capote-part-3.html).

Let's make sure there's a word count associated with each post, and that it shows up in the UI. I'll write a quick test based on some of the first ones we wrote. It tests to make sure the first post has a word count of 33 words. I know it will have 33 words because I literally just counted the words in that post.

You can add this test in the "Posts" section, our first set of tests.

```javascript
// tests/mocha/client/client.js
...
it("should show a wordcount", function(){
  chai.assert.equal($('.wordcount:eq(0)').html(), "33");
});
...
```

First, let's update our fixtures and give each post a word count.

```javascript
// tests/mocha/client/client.js
...
Posts.insert({
  title: "Neutra messenger bag",
  content: "Tousled forage trust fund readymade Neutra messenger bag. Drinking vinegar chia Marfa, vegan messenger bag disrupt Wes Anderson try-hard. Small batch scenester raw denim synth cronut cornhole, iPhone try-hard single-origin.",
  createdAt: new Date(2015,0,4),
  wordCount: 33
});
Posts.insert({
  title: "fatback filet mignon",
  content: "Bacon ipsum dolor amet alcatra turkey shank cupim corned beef brisket chuck boudin tri-tip t-bone kevin fatback filet mignon. Short loin tongue short ribs.",
  createdAt: new Date(2015,0,3),
  wordCount: 26
});
Posts.insert({
  title: "know what I'm sayin'",
  content: "You see? It's curious. Ted did figure it out - time travel. And when we get back, we gonna tell everyone. How it's possible, how it's done, what the dangers are. But then why fifty years in the future when the spacecraft encounters a black hole does the computer call it an 'unknown entry event'?",
  createdAt: new Date(2015,0,1),
  wordCount: 54
});
...
```

Cool.

Reset the server *meteor reset && meteor*

Let's make the word count visible in the app. You can add this line in the "post" template at the end.

```javascript
// client/index.html

<div class="wordcount">{{wordCount}}</div>
```

Our test should be passing.

If you create a new post, there's still no way to calculate or record how many words are in the post. It's time to build that feature. We're going to make use of [wordcount](https://www.npmjs.com/package/wordcount), an npm package. It does what you might think it does, it counts the words in a string. In order to use npm packages with Meteor, we'll need to add the meteorhacks:npm package.

In the command line, run:

  	$ meteor add meteorhacks:npm

Adding this package will automatically create a packages.json file. This is where we'll list our packages, like this:

```javascript
// packages.json
{
  "wordcount": "1.1.1"
}
```

Its a good idea to restart the meteor server at this point so npm and the new package can be loaded into our meteor app.

Its important to know that when you use meteorhacks:npm, these packages are only available server side. So we'll set up our word count function in our server code as a method, then call it from the client when we need to make use of it.

Before we write the code, let's also add one more package:

  	$ meteor add check

The [check](https://atmospherejs.com/meteor/check) package helps with security by ensuring your users can only pass through the right types of data. You used to be able to use the audit-argument-checks package for the same purpose. But it seems that the check package has taken over. You can read some more about it [in the documentation](http://docs.meteor.com/#/full/check)

Here we are creating a new method called 'getWordcount' which accepts one argument, words, which will be coming from the client. We check to make sure it is only accepting a string. We also require the wordcount package. Finally, we return our argument after running it through the wordcount function that counts the words in our string.

```javascript
//server/app.js

Meteor.methods({
  'getWordcount': function getWordcount(words) {
    check(words, String);
    var wordcount = Meteor.npmRequire('wordcount');
    return wordcount(words);
  }
});
```

To see if this is going to work, we call the method in our server file and watch the command line console for an update.

```javascript
//server/app.js

Meteor.call('getWordcount', 'hows it going world?', function(err, results){
  if(err){
    console.error(err);
  }
  else{
    console.log('HEY! it worked, it was ' + results + ' words!');
  }
});
```

The message should be logged in the console and look like: "HEY! it worked, it was 4 words!"

Now we need a way to record how many words the user is actually typing as they type and once we have that number we can easily save it into the database when the user submits their post.

We'll write a test to make sure that if a user enters two words into the 'content' section of the post, then it should be reflected that the word count is 2. This is the first post where we need to set a timeout before running our assertion. Apparently there is some minimal lag in updating the session and reflecting that in the UI, so we'll give it half a second to update. Our test is going to look something like this:

```javascript
// tests/mocha/client/client.js

describe("wordcount", function() {
  before(function(done){
    $('[name="content"]').val('a new');
    $('[name="content"]').keyup();
    done();
  });
  it("displays the wordcount when the user types in the form", function(){
    Meteor.flush();
    setTimeout(function(){
      chai.assert.equal($('span.wordcount-num').text(), "2")
    }, 500);
  });
});
```

Let's create a new template.

```javascript
// client/index.html

<template name="wordcount">
  <div class="wordcount-msg">
    word count: <span class="wordcount-num">{{wordcount}}</span>
  </div>
</template>
```

And include it in underneath the createPost helper.

```javascript
// client/index.html

<body>
  {{> createPost}}
  {{> wordcount}} <!-- new helper here -->
  <div class="container">
    {{#each posts}}
      {{> post}}
    {{/each}}
  </div>
</body>
```

We're going to add a helper and an onRendered call. The onRendered call will set the word count to zero. This makes sense because this should only be called on render, before anyone's typed in the form. Then the helper gets the wordcount's session value.

```javascript
//model/posts.js

Template.wordcount.onRendered(function(){
  Session.set('wordcount', 0);
});
Template.wordcount.helpers({
  wordcount: function(){
    return Session.get('wordcount');
  }
});
```

Now we need to set the session value. We will tie this in to the keyup action in the form. I got this idea from [Meteortips](http://meteortips.com/second-meteor-tutorial/managing-todos/). Whenever a user presses a key (and lifts up their finger) we can fire an event in Meteor that will send whatever is in the form to our wordcount package to count the words. In the createPost events block, add in:

```javascript
//model/posts.js

'keyup [name=content]': function(e){
  var wordsToCount = $('[name="content"]').val();
  Meteor.call('getWordcount', wordsToCount, function(err, results){
    if(err){
      console.error(err);
    }
    else{
      Session.set('wordcount', results);
    }
  });
}
```

*Note:* I moved all the content from client/posts.js into model/posts.js -- it didn't make sense to have two posts.js files. The above code gets wrapped by the Meteor.isClient if statement.

This should put our test in the green.

We'll also take care of one more detail. In order to start again with a clean slate after submitting a post and clearing the form, we should also programmatically wipe the word count session. If we don't, the session value will remain at the previous word count until we start typing again. We can update this by adding in our createPost events function:

```javascript
//model/posts.js

$('[name=title]').val('');
$('[name=content]').val('');
Session.set('wordcount', 0); // clears the session.
```

Awesome!!!!

Next up, we're going to add in a daily goals feature and connect it to the word count so we can set a minimum word count for each post.

If you want to check out the code from this section, you can grab it from [GitHub](https://github.com/austinsamsel/capote/tree/part-4)

# Building Capote: Meteor & Mocha BDD (Part 5)

In the last post we built our word count feature. In this part, we're going to create our daily goal feature. We'll be using the daily goal to compare with the word count. Once the user sets the daily goal then we can set a minimum word count for each post and prevent the user from submitting a post that doesn't have enough words.

You can grab the code from the last chapter [here](https://github.com/austinsamsel/capote/tree/part-4). If you want to review the last post, you can [visit it here](http://hightopsnyc.com/blog/building-capote-part-4.html).

Here's the test for our daily goal. I want to make sure I can see the goal in the DOM, and I also want the user to be able to change the goal and we'll see that goal's value in the database.

```javascript
// tests/mocha/client/client.js

describe("change daily words goal", function(){
  after(function(done){
    $('[name="goal"]').val('');
    done();
  });
  it("user can change their goal", function(){
    Meteor.flush();
    $('[name="goal"]').val(2);
    chai.assert.equal($('[name="goal"]').val(), 2)
  });
});
```

Let's create a new template for the daily goal.

```javascript
// client/index.html
…

{{> createPost}}
{{> wordcount}}
{{> dailyGoal}} <!-- include the helper right below the word count -->

…
<template name="dailyGoal">
  <div class="dailyGoal">
    <form>
      Your daily goal: <input value="{{dailyGoal}}" placeholder="your goal" name="goal"> words.
   </form>
 </div>
</template>
```

Our test passes because it just checks the UI. Now we need to create a Goals collection so we can save the goal in our database as well.

I'll add this test to our server tests:

```javascript
//tests/mocha/server/server.js

describe("save daily words goal in database", function(){
  before(function(done){
    var goalSaved = Goals.findOne()._id;
    Goals.update({_id: goalSaved}, {$set: {dailyGoal: 1}});
    done();
  });
  it("saves the goal in the database", function(){
    Meteor.flush();
    chai.assert(Goals.findOne().dailyGoal == 1);
  });
});
```

We'll also need a Goals publication and subscription. We'll take care of feeding our form the daily goal value with a helper. Finally, we'll structure the form to update our goal's value in the database on the keyup event, so that when the user edits their daily goal, it's automatically saved.

```javascript
// model/goals.js
Goals = new Mongo.Collection('goals');

if (Meteor.isClient) {
  Meteor.subscribe('goals');

  Template.dailyGoal.helpers({
    dailyGoal: function() {
      var theGoal = Goals.findOne().dailyGoal;
      return theGoal
    }
  });

  Template.dailyGoal.events({
    'keyup [name=goal]': function(e){
      e.preventDefault();

      var mostRecent = Goals.findOne();
      var documentId = mostRecent._id;
      var goal = $('[name="goal"]').val();
      Goals.update(
        { _id: documentId },
        {$set: { dailyGoal: goal }
      });
    },
    'keyup [name=content]': function(e){
      var wordsToCount = $('[name="content"]').val();
      Meteor.call('getWordcount', wordsToCount, function(err, results){
        if(err) console.error(err);
        else    Session.set('wordCountResult', results);
      });
    }
  })
}

// server/app.js

Meteor.publish("posts", function(){
  return Posts.find();
});

Meteor.publish("goals", function(){
  return Goals.find();
});
```

Finally, for development purposes, let's kick off our Goals collection with one entry since our code only covers updating the daily goal value. We'll add the following to our startup function:

```javascript
// server/app.js

if (Goals.find().count() === 0) {
  Goals.insert({
    dailyGoal: 1,
    createdAt: new Date(),
  });
};
```

Once we reset the database we'll be able to work with our daily goal. Our test should now be passing.

Finally we can begin to enforce a minimum word count that matches the daily goal. I'm not creating a new tests because this code is going to cause some older tests to break since they don't factor in a minimum word count before submitting their posts. I'll update the tests once they start breaking.

In order to disallow the user to post, while still providing some feedback, we can wrap the submit input in an if statement. If there's enough words, then display the submit button... if not, then post the message, "Keep writing!"

```javascript
// client/index.html
<form>
  Create a post:
  <input type="text" placeholder="add a title" name="title">
  <textarea placeholder="write your words here" name="content"></textarea>
  {{#if enoughWords}}
    <input type="submit" value="Submit" />
  {{else}}
    Keep writing!
  {{/if}}
</form>
```

In order to execute this logic, we'll add a new helper method, enoughWords:

```javascript
// model/posts.js

Template.createPost.helpers({
  enoughWords: function(){
    var wordcount = Session.get('wordcount');
    var theGoal = Goals.findOne().dailyGoal;
    return wordcount >= theGoal
  }
});
```

If you were to test this out in the UI, you'll see that everything is working. Unfortunately this one modification wreaks havoc on our tests. The main culprits are the "creating posts" and "deleting posts" blocks. So we'll need to rewrite them in order to handle the new word count limitations on posting.

I updated the tests with setTimeout functions. I experienced a lot of issues with this part in particular. The way to go when there's so much action and dynamism in the UI is to wrap everything in setTimeout functions to give each block of code time to run and for the UI to update itself.

```javascript
// tests/mocha/client/client.js

describe("Creating Posts", function(){
  before(function(done){
    $('[name="goal"]').val(1);
    $('[name="goal"]').keyup();
    $('[name="title"]').val('a new post');
    $('[name="content"]').val('a new sample post');
    $('[name="content"]').keyup();
    setTimeout(function(){
      $('[type="submit"]').click();
    }, 500)
    done();
  });
  after(function(done){
    setTimeout(function(){
      $('.title:contains("a new post")')
        .siblings('.deletePost').first().click();
    }, 500);
    done();
  });
  it("creates a post when I fill in the fields", function(){
    Meteor.flush();
    setTimeout(function(){
      chai.assert.equal($(".title:eq(0)").html(), "a new post");
    }, 500);
  });
});

describe("deleting posts", function(){
  before(function(done){
    Posts.insert({
      title: "delete me",
      content: "please delete me",
      createdAt: new Date()
    });
    done();
  });
  after(function(done){
    setTimeout(function(){
      $('.title:contains("delete me")')
        .siblings('.deletePost')
        .first()
        .click();
    }, 500);
    done();
  });
  it("deletes when I tell it to delete", function(){
    Meteor.flush();
    $('.title:contains("delete me")')
      .siblings('.deletePost')
      .first()
      .click();
    setTimeout(function(){
        chai.assert.equal($(".title:eq(0)").html(), "Neutra messenger bag");
    }, 500);
  });
});
```

Now that we have these tests passing, we've completed a prototype version of Capote. It doesn't have users, and it still has the autopublish and insecure packages. At least now that we have tests written we can continue development without having to worry so much about whether we're breaking functionality with each update.

If you want to check out the code from this section, you can grab it from [GitHub](https://github.com/austinsamsel/capote/tree/part-5)
