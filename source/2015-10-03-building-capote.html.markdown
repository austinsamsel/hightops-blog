---
layout: layout_article
title: Building Capote Meteor & Mocha TDD (Part1)
date: 2015-10-03 21:34 UTC
tags: meteor, mocha, tdd
---

# Building Capote: Meteor & Mocha TDD (Part 1)

You know what's annoying? Testing your app by physically logging in and going through your app to make sure everything works. Also, it takes a lot of time. Also, its scary. If I was doing that while working for a client, I'd really be stressed.

That's why there's TDD. Writing tests as you write your software. It helps your software suck less, gives you a little more peace of mind, and even though it means writing extra code, there's a really good chance it will save you a lot of time overall.

When you're learning a framework, you don't get much exposure to testing. Usually your learning resources are focused on covering the framework itself. They don't cover tests because there's so many different ways to do it. With Meteor there's a bunch of suites to test your app: cucumber, jasmine, mocha, etc.

If you're looking to get started with TDD or want to see some examples of using Mocha in TDD to build a Meteor app, this tutorial is for you.

If you just want to see the code for this post, you can checkout the [GitHub repository](https://github.com/austinsamsel/capote/tree/part-1).

## Resources

##### Meteor TDD bible:

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

<pre><code class="language-bash">$ meteor create capote
</code></pre>

Then let's trash the default files *capote.css*, *capote.html* and *capote.js*

Let's also add in our testing package:

<pre><code class="language-bash">$ meteor add mike:mocha
</code></pre>

Let's set up our tests directory structure like this:


	<pre><code class="language-javascript">tests/
	- mocha/
	- - client/
	- - server/
</code></pre>

This type of pattern is required for velocity to read the tests and you'll need to use it while testing in Meteor with Cucumber, Jasmine, too. What is velocity? Velocity is Meteor's reactive test-runner and it supports a whole bunch of testing frameworks.

## Our first test

Let's start with the server. There's just a few parts to each post we need right now: a timestamp, a title, and the body of the post.

We'll write our first test. Any mocha tests are first going to be wrapped in the following:

<pre><code class="language-javascript">// tests/mocha/server/server.js
if (!(typeof MochaWeb === 'undefined')){
  MochaWeb.testOnly(function(){
    /* some test code*/
  });
}
</code></pre>

There isn't much documentation I can find about why we ought to do this... It looks like it helps to only run tests in the dev or mirror environment, but I'm honestly not sure. I've been able to run tests without it. For now, I include it because the Velocity developers do it.

Moving along. We'll write our first test, which just makes sure we have at least one post in the database.

<pre><code class="language-javascript">describe("Server initialization", function(){
  it("should insert posts into the database on server startup", function(){
    chai.assert(Posts.find().count() > 0);
  });
});
</code></pre>

If you run meteor now you'll see that velocity pops up. In your logs you'll also see a message saying, "You can see the mirror logs at: tail -f /path/to/your/logs". You can also pop open another command line window and paste in *tail -f /path/to/your/logs* and get some extra insight.

Both the logs and our UI tell us the same thing. "1 test failed" and "Posts is not defined." I love TDD. It tells us what comes next. We can go ahead and define Posts now.

<pre><code class="language-javascript">// model/posts.js
	Posts = new Mongo.Collection('posts');
</code></pre>

When the tests run again, we're told we have a failed "Server initialization." Let's add some posts on server start up.

<pre><code class="language-javascript">// server/app.js
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
</code></pre>

I used [some](http://hipsum.co/) [pretty](https://baconipsum.com/) [sweet](http://slipsum.com/) ipsum generators to create some sample content. We're inserting three posts with a title and content only if there are no posts in the database upon start up.

If you check the logs or the velocity helper in the UI, you'll notice that our first test passed. Great!

## Picking up the pace

I'm going to cap this post here, but in the next posts as we build out this app, I'm going to pick up the pace with these tests and we'll get a whole bunch done, real fast.

You can grab the completed code for this post [here](https://github.com/austinsamsel/capote/tree/part-1).

[Continue to Part 2](http://hightopsnyc.com/blog/building-capote-part-2.html)
