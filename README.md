# Backbone.Radio

[![Travis Build Status](http://img.shields.io/travis/marionettejs/backbone.radio.svg?style=flat)](https://travis-ci.org/marionettejs/backbone.radio)
[![Coverage](http://img.shields.io/codeclimate/coverage/github/marionettejs/backbone.radio.svg?style=flat)](https://codeclimate.com/github/marionettejs/backbone.radio)
[![Dependency Status](http://img.shields.io/david/marionettejs/backbone.radio.svg?style=flat)](https://david-dm.org/marionettejs/backbone.radio)
[![Gitter chat room](https://img.shields.io/badge/gitter-backbone.radio-brightgreen.svg?style=flat)](https://gitter.im/marionettejs/backbone.radio)


Use Backbone.Radio to build large, maintainable, and decoupled applications.

Backbone.Radio is a collection of messaging patterns for Backbone applications. It uses Backbone.Events as a
pub-sub message bus, then adds semantics to your communications through the addition of a new messaging
pattern, Requests. The two systems are bound together as Channels, which provide explicit
namespacing to your communications.

## Installation

Clone this repository or install via [Bower](http://bower.io/) or [npm](https://www.npmjs.org/).

```
bower install backbone.radio
npm install backbone.radio
```

## Documentation

- [Getting Started](#getting-started)
  - [Backbone.Events](#backboneevents)
  - [Radio.Requests](#backboneradiorequests)
  - [Channels](#channels)
  - [Using With Marionette](#using-with-marionette)
- [API](#api)
  - [Radio.Requests](#requests)
  - [Channel](#channel)
  - [Radio](#radio)
  - [Top-level API](#top-level-api)

## Getting Started

### Backbone.Events

Anyone who has used Backbone should be quite familiar with Backbone.Events. Backbone.Events is what facilitates
communications between objects in your application. The quintessential example of this is listening in on a
Model's change event.

```js
// Listen in on a model's change events
this.listenTo(someModel, 'change', myCallback);

// Later on, the model triggers a change event when it has been changed
someModel.trigger('change');
```

Let's look at a diagram for Backbone.Events:

<p align='center'>
  <img src='https://i.cloudup.com/u9oC3S1LxE.svg' alt='Backbone.Events diagram'>
</p>

It goes without saying that Backbone.Events is incredibly useful when you mix it into instances of Classes. But what
if you had a standalone Object with an instance of Backbone.Events on it? This gives you a powerful message bus to utilize.

```js
// Create a message bus
var myBus = _.extend({}, Backbone.Events);

// Listen in on the message bus
this.listenTo(myBus, 'some:event', myCallback);

// Trigger an event on the bus
myBus.trigger('some:event');
```

This is the first principle of Backbone.Radio: building a message bus out of Backbone.Events is useful. But before we go more
into that, let's look the other messaging system of Backbone.Radio.


### Backbone.Radio.Requests

You use Requests just like Events: mix it into an object.

```js
_.extend(myObj, Backbone.Radio.Requests);
```

Requests are semantic, by which I mean that there is an intention when making a
request. Of course, the intent here is that you are asking for information to be returned. Unlike events, requests have a one-to-one system;
you can't have multiple 'listeners' to the triggerer.

Let's look at a basic example.

```js
// Set up an object to reply to a request. In this case, whether or not its visible.
myView.reply('visible', this.isVisible);

// Get whether it's visible or not.
var isViewVisible = myView.request('visible');
```

The handler in `reply` can either return a flat value, like `true` or `false`, or a function to be executed. Either way, the value is sent back to
the requester.

Here's a diagram of the Requests pattern:

<p align='center'>
  <img src='https://i.cloudup.com/tEVU_tuRIX.svg' alt='Backbone.Requests diagram'>
</p>

### Channels

The real draw of Backbone.Radio are channels. A Channel is simply an object that has Backbone.Events and Radio.Requests mixed into it;
it's a standalone message bus comprised of both systems.

Getting a handle of a Channel is easy.

```js
// Get a reference to the channel named 'user'
var userChannel = Backbone.Radio.channel('user');
```

Once you've got a channel, you can attach handlers to it.

```js
userChannel.on('some:event', function() {
  console.log('An event has happened!');
});

userChannel.reply('some:request', 'food is good');
```

You can also use the 'trigger' methods on the Channel.

```js
userChannel.trigger('some:event');

userChannel.request('some:request');
```

You can have as many channels as you'd like

```js
// Maybe you have a channel for the profile section of your app
var profileChannel = Backbone.Radio.channel('profile');

// And another one for settings
var settingsChannel = Backbone.Radio.channel('settings');
```

The whole point of Channels is that they provide a way to explicitly namespace events in your application. It gives you greater
control over which objects are able to talk to one another.

If you're having difficulty remembering the API of Channels here's a useful mnemonic for you.

Events is the API that you know; `on`, `off`, `stopListening` and so on. Requests, which starts with an R,
only uses verbs that start with R: `request`, `reply`, `stopReplying` and so on.

### Using With Marionette

[Marionette](https://github.com/marionettejs/backbone.marionette) does not use Radio by default, although it will in the next major release: v3. However, you can use Radio today by including a small shim after you load Marionette, but before you load your application's code. To get the shim, refer to [this Gist](https://gist.github.com/jmeas/7992474cdb1c5672d88b).

## API

Like Backbone.Events, **all** of the following methods support both the object-syntax and space-separated syntax. For the sake of brevity,
I only provide examples for these alternate syntaxes in the most common use cases.


### Requests

#### `request( requestName [, args...] )`

Make a request for `requestName`. Optionally pass arguments to send along to the callback. Returns the reply, if one
exists. If there is no reply registered then `undefined` will be returned.

You can make multiple requests at once by using the space-separated syntax.

```js
myChannel.request('requestOne requestTwo');
```

When using the space-separated syntax, the responses will be returned to you as an object, where
the keys are the name of the request, and the values are the replies.

#### `reply( requestName, callback [, context] )`

Register a handler for `requestName` on this object. `callback` will be executed whenever the request is made. Optionally
pass a `context` for the callback, defaulting to `this`.

To register a default handler for Requests use the `default` requestName. The unhandled `requestName` will be passed as the first argument.

```js
myChannel.reply('default', function(requestName) {
  console.log('No reply exists for this request: ' + requestName);
});

// This will be handled by the default request
myChannel.request('someUnhandledRequest');
```

To register multiple requests at once you may also pass in a hash.

```js
// Connect all of the replies at once
myChannel.reply({
  'some:request': myCallback,
  'some:other:request': someOtherCallback
}, context);
```

Returns the instance of Requests.

#### `replyOnce( requestName, callback [, context] )`

Register a handler for `requestName` that will only be called a single time.

Like `reply`, you may also pass a hash of replies to register many at once. Refer to the `reply` documentation above
for an example.

Returns the instance of Requests.

#### `stopReplying( [requestName] [, callback] [, context] )`

If `context` is passed, then all replies with that context will be removed from the object. If `callback` is
passed then all requests with that callback will be removed. If `requestName` is passed then this method will
remove that reply. If no arguments are passed then all replies are removed from the object.

You may also pass a hash of replies or space-separated replies to remove many at once.

Returns the instance of Requests.

### Channel

#### `channelName`

The name of the channel.

#### `reset()`

Destroy all handlers from Backbone.Events and Radio.Requests from the channel. Returns the channel.

### Radio

#### `channel( channelName )`

Get a reference to a channel by name. If a name is not provided an Error will be thrown.

```js
var authChannel = Backbone.Radio.channel('auth');
```

#### `DEBUG`

This is a Boolean property. Setting it to `true` will cause console warnings to be issued
whenever you interact with a `request` that isn't registered. This is useful in development when you want to
ensure that you've got your event names in order.

```js
// Turn on debug mode
Backbone.Radio.DEBUG = true;

// This will log a warning to the console if it goes unhandled
myChannel.request('show:view');

// Likewise, this will too, helping to prevent memory leaks
myChannel.stopReplying('startTime');
```

#### `debugLog(warning, eventName, channelName)`

A function executed whenever an unregistered request is interacted with on a Channel. Only
called when `DEBUG` is set to `true`. By overriding this you could, for instance, make unhandled
events throw Errors.

The warning is a string describing the type of problem, such as:

> Attempted to remove the unregistered request

while the `eventName` and `channelName` are what you would expect.

#### `tuneIn( channelName )`

Tuning into a Channel is another useful tool for debugging. It passes all
triggers and requests made on the channel to

[`Radio.log`](https://github.com/jmeas/backbone.radio#log-channelname-eventname--args-).
Returns `Backbone.Radio`.

```js
Backbone.Radio.tuneIn('calendar');
```

#### `tuneOut( channelName )`

Once you're done tuning in you can call `tuneOut` to stop the logging. Returns `Backbone.Radio`.

```js
Backbone.Radio.tuneOut('calendar');
```

#### `log( channelName, eventName [, args...] )`

When tuned into a Channel, this method will be called for all activity on
a channel. The default implementation is to `console.log` the following message:

```js
'[channelName] "eventName" args1 arg2 arg3...'
```

where args are all of the arguments passed with the message. It is exposed so that you
may overwrite it with your own logging message if you wish.

### 'Top-level' API

If you'd like to execute a method on a channel, yet you don't need to keep a handle of the
channel around, you can do so with the proxy functions directly on the `Backbone.Radio` object.

```js
// Trigger 'some:event' on the settings channel
Backbone.Radio.trigger('settings', 'some:event');
```

All of the methods for both messaging systems are available from the top-level API.

#### `reset( [channelName] )`

You can also reset a single channel, or all Channels, from the `Radio` object directly. Pass a
`channelName` to reset just that specific channel, or call the method without any arguments
to reset every channel.

```js
// Reset all channels
Radio.reset();
```
