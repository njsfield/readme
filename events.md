### The observer pattern (and the publish/subscribe pattern)

The observer pattern is a software design pattern in which objects ('subjects') are able to notify other objects ('observers') of events or changes of state.

Subjects and observers are also known as publishers and subscribers, so the observer pattern is also known as the publish/subscribe (or pub/sub) pattern.

Unlike callbacks, where a single 'listener' is called on completion of a specific task, the observer pattern allows for a one-to-many relationship between objects - in other words, a subject can notify several observers when its state changes.

The pattern is mainly used within event handling systems, where a single event might trigger any number of functions.

The observer pattern offers huge performance gains over alternative implementations - compared with, for example, iterating over every possible dependent object to determine whether it needs to be affected by a state change.

The observer pattern is one of the fundamental patterns underlying the Model-View-Controller architecture, which is ubiquitous in everything from game design to web development (one of Angular's key selling points is that it functions as an MVC framework).

As a result of the observer pattern's utility and popularity, it has been implemented in several libraries (including the code libraries of Java and Node), and even languages (such as the event keyword in C#).

### EventEmitters and event listeners

Node is more than just a way to use JS outside the browser. It also brings with it an asynchronous style of programming based around events.

We are already familiar with events, like in the following example:

```javascript
document.getElementById('btn').addEventListener('click', function(e) {
  console.log('omg stop poking me!');
});
```

With this bit of code we listen out for a 'click' event and register a function to call whenever we receive a click event.

Node takes this concept and generalises it so that you can listen for and emit all kinds of custom events, not just user input.

The benefit of doing this is that we can write code that is loosely-coupled and asynchronous by default. By communicating primarily with events, our functions and modules don't need to worry about how other parts of code are implemented - they just need to agree on what kind of events are fired and when.

#### Emitting events

In node all objects that can emit and listen to events inherit from the `EventEmitter` class. Node provides a few utility functions so that you can merge the relevant methods and properties into objects that you define yourself, but to keep things simple we'll show an example below that just creates a new `EventEmitter` object:

```javascript
var EventEmitter = require('events').EventEmitter; // This is the EventEmitter class.

var emitter = new EventEmitter(); // Create an object 

var remonstrate = function() {
  console.log('Shut the door, it\'s cold!');
};

emitter.on('doorOpen', remonstrate); // Just like with addEventListener, we name the event to listen for and register a function
emitter.emit('doorOpen'); // Fire off an event
```
This code will print a plea to shut the door.

By creating an `EventEmitter` object (or we could have created an object that inherits from it), we gain the `on` and `emit` methods that allow us to handle events. `on` takes two arguments: the name of the event to listen out for (as a string) and a function to call whenever the event happens. When we want to fire off an event we just tell the object to `emit` the event.

To link back to the discussion above, we can see that `on` subscribes its object to events and `emit` publishes an event.

In this example the same object emits and handles the event, but this is not a requirement. We can also send data with our event in the form of an argument.

Let's think back to the Node Girls CMS workshop for a more practical example. Whenever a blog post is submitted, we want it to save it to the website's list of posts. In this example, assume that `blogList` is an object that inherits from `EventEmitter`:

```javascript
var querystring = require('querystring');

var handler = function(req, res) {
  if(req.url === '/submit-post' && req.method === 'POST') {
    var body = '';

    req.on('data', function(chunk) {
      body += chunk;
    });

    req.on('end', function() {
      var blogPost = querystring.parse(body);
      req.emit('newPost', blogPost);
    });
  }
};

/* TOTALLY SOMEWHERE ELSE IN YOUR CODE */

blogList.on('newPost', function(post) {
  listOfPosts.push(post);
});
```

The key point to note here is that `blogList` doesn't know anything about where the event came from. All is it cares about is whether a 'newPost' event has been dispatched. In this example the 'newPost' event is fired as part of the HTTP request handling but it could easily come as part of loading a backup from a file on disk or from elsewhere. We could completely replace how new posts are submitted and the listener would not need to change as long as we continue to provide it with a valid 'post' argument.

Using events means that we can easily extend our code without having to make changes to existing code. Let's say we want to automatically generate a new HTML page for every blog post that gets submitted. All we'd need to do is make sure that the page generation code includes a listener for our 'newPost' event:

```javascript
pageGenerator.on('newPost', function(post) {
  createNewPageFor(post);
});
```

#### Inheriting from EventEmitter

Whenever you see an object using methods like `on` that means it's using the functionality provided by `EventEmitter`. In these examples we just created new `EventEmitter` objects to get access to these methods. Of course, in real code you'd probably define your own objects for different bits of functionality. It would be good if we could pull the `EventEmitter`'s functionality into our own objects.

Hooray, we can! The syntax is a little funky so we haven't covered it in this README but [this article](https://code.tutsplus.com/tutorials/using-nodes-event-module--net-35941) covers it clearly.

### Resources

https://www.sitepoint.com/nodejs-events-and-eventemitter/
https://www.packtpub.com/mapt/book/web-development/9781783287314/1/ch01lvl1sec12/the-observer-pattern
https://en.wikipedia.org/wiki/Publish–subscribe_pattern
https://en.wikipedia.org/wiki/Observer_pattern
http://blog.yld.io/2015/12/15/using-an-event-emitter