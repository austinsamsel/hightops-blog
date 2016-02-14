---
title: Working with a React View Layer in Meteor
date: 2015-12-03 19:04 UTC
tags: meteor, react, javascript
---

I've been trying to stay focused in terms of what new technologies I'll try out or learn. I've been reading about React since the beginning of the year, but I've held off on doing anything more than a tutorial. Now it seems like [React might be more fully integrated with Meteor](https://forums.meteor.com/t/next-steps-on-blaze-and-the-view-layer/13561) as a front end engine, beyond official support by the Meteor Development Group. According to the discussion its possible Blaze 2.0 could be some kind of merger with React. Once I read that, it gave me the excuse I needed to start working with React.

## Still <3 you Blaze

Blaze made development really fast and productive. Wrapping HTML in template tags felt like a pretty good separation of concerns and was really easy to conceptualize. I had my HTML, my CSS, and my Javascript. If I needed some logic in my templates I had a lot of options with spacebars, helpers. However, it seems like we can get all that same functionality with React.

## Cleaner Code

To test it out, I'm creating an app that lets users send letters (or messages), snail mail style — where it takes a few days for each message to be delivered. It's an idea I've been thinking about for a while.

By now I've run the [official Meteor React tutorial](https://www.meteor.com/tutorials/react/creating-an-app) a couple times and used that basic functionality as a starting point for this app. I feel like I finally hit that react style "functional" (using this word lightly here) programming moment when I refactored a bunch of functionality I was storing in variables... and moved it out into functions which could be used a little bit more open-endedly and repeatedly. Of course you don't need React to code like this, but this style is more encouraged... As a reference for the following example, you can see the full code at my [Github](https://github.com/austinsamsel/snail).

In my user.jsx file, my code was looking like this. This is probably how I would have written my apps in Meteor before. I'd use a lot of variables and make it kind of human readable by naming them well. It always worked! But there's probably a better way…

  ```javascript
  // components/User.jsx
  render(){
    var currentUserId = Meteor.userId();
    var followedUserId = this.props.user._id;
    var notCurrentUser = this.props.user._id != currentUserId;
    var notFollowedByCurrentUser = Relationships.findOne({$and : [{owner : currentUserId}, {saveContact : followedUserId}] }) == null;
    var followUserButton = notCurrentUser && notFollowedByCurrentUser;

    var followedByCurrentUser = Relationships.findOne({$and : [{owner : currentUserId}, {saveContact : followedUserId}] }) != null;

    var unfollowUserButton = notCurrentUser && followedByCurrentUser;

  return (
    <li>
      {this.props.user.username}
      { followUserButton ? (
        <button className="save" onClick={this.saveThisContact}>save contact</button>
        ) : ''}

      { unfollowUserButton ? (
        <button className="remove" onClick={this.removeThisContact}>remove contact</button>
      ): ''}
    </li>
    )
  }
  ```

I searched around for how other people were working with React, especially in terms of Meteor and came across [this article](http://blog.differential.com/react-for-meteor-developers/) which inspired me to rearrange my code like this:

  ```javascript
  // components/User.jsx
  render(){
    return (
      <li>
        {this.props.user.username}
        {this.followButton()}
        {this.unfollowButton()}
      </li>
    )
  },
  //method calls
  saveThisContact() {
    Meteor.call("saveContact", this.props.user._id);
  },
  removeThisContact(){
    var currentUserId = Meteor.userId();
    var followedUserId = this.props.user._id;
    Meteor.call("removeContact", currentUserId, followedUserId);
  },
  // render conditionals
  followButton(){
    if ( ! this.isCurrentUser() && ! this.isFollowed() ) {
      return <button
        className="save"
        onClick={this.saveThisContact}>
        save contact
      </button>;
    }
  },
  unfollowButton(){
    if ( ! this.isCurrentUser() && this.isFollowed() ) {
      return <button
        className="remove"
        onClick={this.removeThisContact}>
        remove contact
      </button>;
    }
  },
  // helper functions
  isCurrentUser(){
    if (this.props.user._id == Meteor.userId())
      return true;
  },
  isFollowed(){
    if (Relationships.findOne({$and: [{owner: Meteor.userId()}, {saveContact: this.props.user._id}] }) != null)
      return true;
  },
  ```

Here, I moved the render() function towards the top, so if I'm searching through my files, I'll immediately see what's being rendered in the UI, rather than a bunch of functions. I replaced all the variables with functions. I also moved the if/else logic out of the render() function and into helper functions. On the one hand, the first render() function looks a lot more clean, but also a lot more empty. This definitely spreads out the code more, but I think it'd also be faster to trace. And as the app grows, or if more is included in the component, it should be easy to maintain.

## Atmosphere packages

It isn't explicitly said anywhere, so maybe this is just something that is obvious to everyone but me, but you can use atmosphere packages really easily with React in your Meteor projects. The Meteor-react tutorial gave some special care to the Accounts UI package, setting it up in a wrapper before placing it in the DOM. This got me thinking I'd need to drop Atmosphere and start working with meteorhacks:npm and cosmos:browserify packages.

However, yes, you can use Atmosphere as your package manager. When I wanted to include a timestamp for each message, I ran "meteor add momentjs:moment" in the command line as usual to include [Moment](http://momentjs.com/).

I like to do things step by step, so I first tried to print the timestamp with a prop like:

  ```javascript
  {this.props.letter.createdAt}
  ```

so that I could at least see the object I'd be working with. However this returned a pretty dense error message:

  ```plaintext
  Uncaught Error: Invariant Violation: Objects are not valid as a React child (found: Sun Nov 29 2015 17:31:47 GMT-0500 (EST)). If you meant to render a collection of children, use an array instead or wrap the object using createFragment(object) from the React add-ons. Check the render method of `Letter`.
  ```

After some googling I caught a hint that the date object needed to be converted to a string first.

So I created a function to do just that

  ```javascript
  letterCreatedAt(){
    var a = this.props.letter.createdAt;
    return a.toString();
  }
  ```

And added in a call from the render() function:

  ```javascript
  {this.letterCreatedAt()}
  ```

Seeing that that worked. I updated my function to covert the timestamp with Moment JS.

  ```javascript
  letterCreatedAt(){
    var a = this.props.letter.createdAt;
    return moment(a).format('MMMM Do YYYY, h:mm a');
  }
  ```

## React packages

Some atmosphere packages won't work well with React. For example, I wanted to build an input field with autocomplete functionality that would help search for users. I began installing the [typeahead package](https://github.com/sergeyt/meteor-typeahead/) which was based on meteor templates and found out it was not going to be easily compatible with React.

So for a case like this I needed browserify and meteorhacks:npm as described in the [React-in-Meteor docs](http://react-in-meteor.readthedocs.org/en/latest/client-npm/)

I found the [cosmos:browserify](https://github.com/elidoran/cosmos-browserify) docs contradicted the React-in-Meteor docs suggesting to use [exposify](https://www.npmjs.com/package/browserify-exposify) rather than [externalify](https://www.npmjs.com/package/externalify) in order to instruct any npm modules to use Meteor's version of react. The cosmos:browserify docs also suggested to place the browserify files in a client/lib folder so the libraries are only loaded once, instead of twice (client and server).

Once following the instructions for set up I was able to get it working with [React Typeahead](https://www.npmjs.com/package/react-typeahead). I tried several different similar packages, but this was the easiest to get working.


## Onward

At this point in my app, I've created the barebones functionality with the backend Meteor logic and simple react components to create views. I'm looking forward to animating with React and turning this into a full featured application.
