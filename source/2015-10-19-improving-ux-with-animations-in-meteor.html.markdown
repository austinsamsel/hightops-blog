---
title: Improving User Experience Meteor Reactivity and CSS Animations
date: 2015-10-19 00:16 UTC
tags: meteor, ux, animations
---

[Nice Track](https://nicetrack.meteor.com) is one of the first apps I created using Meteor. It allows you to privately create topics which you can then rate on a daily basis in order to track your progress, satisfaction, or feelings. You can also add a few notes to each rating. After you've rated your topics some statistics are provided so you can analyze your ratings.

When I was creating this app, I was mostly interested in improving my skills within the Meteor framework. I didn't put a lot of emphasis on how it looked or felt. As a result, it's basically a prototype and not much more. The looks are mostly derived from [Semantic UI](http://semantic-ui.com/) (my favorite modular, prototyping library). Now I'd love to make this its own thing. I'll be doing it with small improvements. One thing I want to improve on is the animations and to provide better context to users.

If you want to checkout the before and after you can:

**Before:**
[Live version](https://nicetrack-anims-before.meteor.com)
 / [GitHub](https://github.com/austinsamsel/endless-race/tree/css-animations-before)

**After:**
[Live version](https://nicetrack-anims-after.meteor.com)
 / [GitHub](https://github.com/austinsamsel/endless-race/tree/css-animations-after)

Currently, when a user rates a topic, here's the code I used to slide up the item and fade it out of view. It fires once a rating is successfully submitted. It works and looks fine, however when a user goes to another page and comes back, all the animations are undone and the DOM resets itself showing posts that were just recently rated. This was fine for prototyping and executing an effect, but as I improve the app, this is one thing I want to take care of. I'd like to encourage users to rate their topics once a day.

<pre><code class="language-javascript">// client/templates/ratings/rating_submit.js

this.$(e.target).closest('.topicItem').fadeTo('fast', 0.00, function() {
  $(this).slideUp("fast", function(){
    $(this).remove();
    //$(this).addClass('hideIt');

    Meteor.setInterval(function(){
      Session.clear('hideIt')
    }, 5 * 1000 * 6);
  });
});</code></pre>

This code resulted in an effect like this:

<img src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==" data-src="/images/blog/animation-before.gif" alt="ui before">

In order to bring the animation in line to the data... I'm using this logic. If a topic has been rated within the last 6 hours, its form should be hidden. So I created a helper that looks like this:
`<pre><code class="language-javascript">// client/templates/topics/topic_item.js
Template.topicItemRate.helpers({
  hideRecent: function(parentContext){
    var timeRange = moment().subtract(12, 'hours')._d;
    if (this.lastRating > timeRange) {
      return "hideThis";
    }
  }
});</code></pre>

I make use of [Moment JS](http://momentjs.com/) to create a variable for the time set at 6 hours ago. Then I use that timeframe to check if the last rating happened within the last 6 hours. If it did, then I'll fade out that topic's rating form and remove it from the UI.

The template looks like this:

    // client/templates/topics/topic_item.js

    <template name="topicItemRate">
      {{#if recentlyRated}}
      {{else}}
        <div class="ui segment list topicItem {{hideRecent}}"> <!-- added in the {{minRecent}} helper here -->
          <div class="item">
            <div class="header"><h3>{{title}} </h3></div>
            {{> ratingSubmit}}
          </div>
        </div>
      {{/if}}
    </template>

When the {{minRecent}} helper is active, it adds the class, "hideThis" which includes this CSS:

<pre><code class="language-css">// client/stylesheets/style.scss

@keyframes fadeOut {
  0% {
    max-height: 500px;
    opacity: 1;
  }
  39% {
    max-height: 300px;
    opacity: .80;
  }
  78.75% {
    max-height: 50px;
    opacity: .3;
  }
  99.999999% {
    max-height: 0px;
    opacity: 0;
    position:relative;
    z-index:1;
    padding: 0 1em; /* makes animation smoother */
  }
  100%{
    position:absolute;
    top:-9999999px;
    left:-9999999px;
    z-index:-999999999;
  }
}
.topicItem {
  opacity:1.0;
  max-height:10000px;
  overflow:hidden;
  transition:translateY 0.5s linear;
  z-index:1;
  position:relative;
  transform:translateY(0)
}
.topicItem.hideThis{
  animation: fadeOut 500ms linear 0s 1 alternate forwards;
}</code></pre>

Our new animation looks like this:

<img src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==" data-src="/images/blog/animation-after.gif" alt="ui afterwards">

It's basically the same look, except that even if a user refreshes the page or navigates away and comes back, topics that have already been rated within the past 12 hours will not be hidden.

In the same way many apps use notifications, I'll implement a counter for how many items are ready to be rated. I want to compel people to check out the Ratings page since the activity on there drives the app. Also, the number of topics that need ratings may change from time to time, so it'd be good to give the user an idea of what remains unrated even while elsewhere in the app.

I created a todoCount helper which counts how many topics have not been rated within the past 6 hours.

<pre><code class="language-javascript">// client/templates/includes/header.js

todoCount: function(){
	var timeRange = moment().subtract(12, 'hours')._d;
	var topicsCount = Topics.find({ 'lastRating' : { $lte: timeRange } });

	return topicsCount.count();
}</code></pre>

Then the corresponding helper in the header template:

    // client/templates/includes/header.html
    ...
    <a class="{{activeRouteClass 'home' 'toRateTopics'}} item" href="{{pathFor 'toRateTopics'}}">
    	<i class="checkmark icon"></i> <span class="menu-title">Rate <span class='todo'>{{todoCount}}</span></span> <!-- added the {{todoCount}} helper here -->
    </a>
    ...

And I'll give it some nice CSS styling so it fits in more as a "notification count" which will appear in a little pink bubble.

<pre><code class="language-css">// client/stylesheets/style.scss

span.todo{
  background: deeppink;
  position: relative;
  color: #fff;
  font-weight: bold;
  border-radius: 5px;
  font-size: 10px;
  padding: 2px 5px 2px 6px;
  text-align: center;
  vertical-align: 3px;
}</code></pre>

While its nice to have an animation that hides each topic item, it'd be nice if it didn't run the animation when coming to the ratings page from another page in the app. I'd really like for it to only start an animation after the user makes a rating.

<pre><code class="language-javascript">//client/topics/topic_item.js
Template.topicItemRate.onRendered (function() {
  $('.hideThis').remove();
});</code></pre>

This will remove any rating cards that have a .hideThis class attached once the items have rendered. This is a quick and easy way of skipping the loading animation when we don't really need it.

I think these updates will make using Nice Track a little bit more enjoyable and sensible. If you have any suggestions for improvements please let me know!
