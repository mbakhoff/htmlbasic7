# Webapps with a sprinkle of Javascript

So far our forum page has been using the good old server side rendering pattern.
When the client does something on the page, the browser sends a request to the server and the server will respond with a new page to show.
This works great, but sometimes we would like to update the page without reloading it completely.

This is where Javascript comes in.
It's the only programming language you can currently run in the browser and despite its problems, it has become hugely popular.
The language itself is quite old and has changed a lot over the years.
It's often also called ECMAScript, which is the name of the standardized language specification.

In this tutorial we will add a "who's writing" feature to our forum threads:
* when someone is writing a new post in a forum thread, the browser should send a notification to the server.
  let's define writing as changing the contents of the "new post" form inputs.
* the server receives the notifications and keeps track which users are writing in each forum thread.
* the server should send a notification to other users in the same thread when someone starts writing.
* the browser should start listening to notifications from the server when a thread view is opened.
  the list of users who are writing to the thread should be displayed next to the new post form.
  the list should be updated immediately after receiving an update from the server, without reloading the entire page.

## Javascript in 10 minutes

Javascript is a dynamically typed language - you only need to write type names when creating new objects.
Javascript is a managed language with garbage collection - all memory is managed by the browser, unused objects are automatically cleaned up.
Javascript is single threaded - no need to use synchronized blocks or thread-safe data structures.

To use Javascript on a page, you first write the code in a separate file (file extension *.js*).
Next, you add the script to your page using a `<script>` tag.
When the page loads, the script is run.
At this point the script should add event listeners to the page:
you can let the browser run your Javascript functions when the user clicks a button, types something in a `<input>`, resizes the window etc.

Your script doesn't have a main method.
The browser will run the methods when the expected events happen.
**While your script is running, the browser tab is blocked and not responding to any user interaction.**
This means if your script takes too long, the browser will act like is has crashed and the user will be very annoyed.

Some Javascript samples follow.

### Including a script

```html
<html xmlns:th="http://www.thymeleaf.org">
<head>
  <!-- using thymeleaf -->
  <script th:src="@{/js/example.js}" defer></script>
</head>
```

The `defer` keyword tells the browser to only run the script after it has loaded the page.
Otherwise the html elements do not exist yet and it's difficult to attach event listeners to them.

You may also need to update the *Content-Security-Policy* header to allow scripts from your server.

### Basic syntax examples

*js/example.js*
```js
'use strict';

let someString = 'someValue';
let someNumber = 42;
let someArray = ['some', 'Value', 4, 2];

for (let elem of someArray) {
    // print to dev tools console
    console.log(elem);
}

let map = new Map();
map.set('key', 'value');
let found = map.get('key');
if (found === 'value') {
    console.log('map works!');
} else if (found !== 'value') {
    console.log('cannot happen');
}

function printSize(someMap) {
    console.log('map size is ' + someMap.size);
}

printSize(map);

function printHi() {
    console.log('hi');
}

// printHello is a variable that points to the function printHi
let printHello = printHi;
printHello(); // actually runs printHi

// declare an anonymous function and assign to a variable:
let anon = function() {
    console.log('function has no name');
};
anon(); // logs 'function has no name'

// anonymous functions are often inlined;
// declare a function and pass it to a method:
callArgumentFunctionLater(function() {
    console.log('called later');
});

let someObject = {
    a: 1,
    b: 2
};
someObject['c'] = 3;
console.log('a -> ' + someObject['a']); // 1
console.log('c -> ' + someObject['c']); // 3

let asInteger = parseInt('42'); // string to int
```

Some notes:
* every *.js* file should start with `'use strict';`
  this is just a string literal, but it's magical - it turns on [strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode) which makes the code behave more sanely.
* use `===` and `!==` for comparison.
  the regular `==` and `!=` work too, but behave funny.
  for example `'1' == true` is true, but `'2' == true` is false.
* many old tutorials recommend using `var x = whatever;`.
  **never use `var`, always use `let` (as in `let x = whatever;`).**
  `var` has has insane scoping rules, e.g. you can use the variable before declaring it and after it has gone out of scope.
* many old tutorials recommend using `for (let key in collection) { }`.
  the [`for (.. in ..)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in) syntax does not iterate the items in the collection, it iterates the *indexes if the collection*.
  **you almost always want to use [`for (.. of ..)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of) which iterates the elements of the collection.**
* to use the latest Javascript features, configure your IDE to use ECMAScript 6.
  in IntelliJ you can change it in "Settings" -> "Languages and Frameworks" -> "Javascript".
  pick ECMAScript 6 and enable "Prefer strict mode".

### HTML DOM API

You can change the contents of a HTML page directly from Javascript.

Given a HTML body
```html
<body>
  <div id="users">
    <div class="user">Alice</div>
    <div class="user">Bob</div>
  </div>
  <input id="name" />
  <button id="addbutton">Add user</button>
</body>
```

You can use the script
```javascript
'use strict';

// browsers provide a global variable 'document'
// this is used for accessing different html elements on the page
// https://developer.mozilla.org/en-US/docs/Web/API/Document

// find a reference to html element
let users_div = document.getElementById('users');
console.log(users_div.childNodes.length); // 2

// find all elements matching a CSS selector
let user_divs = document.querySelectorAll('.user');
console.log(user_divs.length); // 2

// create a new element
let mallory = document.createElement('div');
// assign html content and class to an element
mallory.innerHTML = 'Mallory';
mallory.classList.add('user');
// add a element to html
users_div.appendChild(mallory);

// print all users to console
for (let user_div of document.querySelectorAll('.user')) {
    console.log(user_div.innerHTML);
}

// add event handlers;
// addEventListener takes two parameters:
// 1) event type (string)
// 2) event listener (function)
let name_input = document.getElementById('name');
name_input.addEventListener('input', function(event) {
    // type 'input' is fired when the value of an <input> element is changed.
    // different objects create different types of events;
    // most event type are listed in the events reference (see below)
    console.log('input changed; new value is ' + name_input.value);
});

let add_button = document.getElementById('addbutton');
add_button.addEventListener('click', function(event) {
     let newUser = document.createElement('div');
     newUser.innerHTML = name_input.value;
     newUser.classList.add('user');
     users_div.appendChild(newUser);
     name_input.value = '';
});
```

See the docs for [addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener) and the [events reference](https://developer.mozilla.org/en-US/docs/Web/Events).

## Sending writing notifications to the server

The first thing we need to do is send notifications to the server when the user starts writing a new post in a thread.

We can make requests to the server using Javascript, so that the request is sent in the background and the entire page is not refreshed.
The right API to use is [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest):

```javascript
let request = new XMLHttpRequest();
request.addEventListener('load', function(event) {
    console.log('request finished');
});
request.addEventListener('error', function(event) {
    console.log('request failed');
});
request.open("POST", "/some/server/mapping");
request.send("some string"); // this will be the request body
```

Most Javascript APIs are asynchronous.
Calling the `send` method starts the sending of the request and returns immediately (before the response arrives).
The request is actually sent out by the browser in the background.
Later, when the response arrives, the browser will fire a `load` event on the request object and our event handler function is started.
If the request fails, then the browser will instead fire an `error` event and our error event hander is called.
If the `send` method would wait for the response to arrive before returning, then the entire browser tab would be blocked for potentially very long time.

Note that despite the name *XMLHttpRequest*, the request can be used to send any kind of data (not limited to XML).

In our case, we also need to add the CSRF token.
First, add the token to the HTML page:

```html
<html xmlns:th="http://www.thymeleaf.org">
<head>
  <!-- the _csrf values are generated by spring security filters -->
  <!-- thymeleaf will render the token into the html -->
  <meta name="_csrf" th:content="${_csrf.token}"/>
  <meta name="_csrf_header" th:content="${_csrf.headerName}"/>
</head>
```

Then add the CSRF token to the `XMLHttpRequest` as a header:

```javascript
let csrf_name = document.querySelector("meta[name='_csrf']").getAttribute('content');
let csrf_value = document.querySelector("meta[name='_csrf_header']").getAttribute('content');

let request = new XMLHttpRequest();
request.open("POST", "/some/server/mapping");
request.setRequestHeader(csrf_name, csrf_value);
request.send("some string");
```

You now have all the pieces to implement sending the notifications to the server.
* create a new Javascript file in src/main/resources/public.
  include the file in the forum thread html.
  update the CSP header to allow Javascript (from our server only).
* create a new controller in the server and a request handler for receiving the writing notifications.
  the request handler takes one parameter - a forum thread's id where the logged in user is currently writing.
* write a Javascript function that has one parameter: the forum thread's id.
  the function should send a `XMLHttpRequest` to the new request hander and send the forum thread's id.
* add an event hander to the new post html form inputs.
  when the input changes, call the function to send the forum thread's id to the server.

There are different ways to get the forum thread's id in the script:
* create a new `<meta>` tag, use thymeleaf to fill it with the forum thread's id on the server and read it with Javascript like the csrf token.
* parse the page URL using Javascript and pick it out of the */threads/{id}* part (see [`window.location`](https://developer.mozilla.org/en-US/docs/Web/API/Window/location))

There are different ways to send the thread id to the server using `XMLHttpRequest`:
* pass the id to the request's [`send`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/send) method, and add a `@RequestBody String id` parameter to the request handler.
* send an empty body (no argument to the `send` method) and add the id as a @PathVariable
* send the id as a *request parameter*.
  the request url should look like */server/mapping?id=123*.
  the parameter can be accessed by adding the `@RequestParam Long id` parameter to the request hander.

What's the difference between path variables (`@PathVariable`) and request parameters (`@RequestParam`)?
*Path variables* are part of the *Request URI* (the resource path).
You cannot have optional path variables without creating several request handlers in the server.
*Request parameters* are the classic solution to passing arguments to the server.
You can have several parameters in a single request: `/mapping?name=john&lastname=smith`.
The parameters can be optional and the *Request URI* still stays the same.
The request parameters can be freely used in both *GET* and *POST* requests.

What should the server do with the thread id?
The controller should contain a map-like structure where it can store the writing users for each thread.
When a new thread id is sent, the server can check who is logged in and associate the user's id/name with the thread.
The server should keep track when an user was last writing to a thread - if there has been no activity for 30 seconds, then the user is likely no longer writing a post.
The cleanup of inactive users can be done when a new notification arrives.
Note that the controller can service multiple requests in parallel - the data store must be thread safe.

A naive solution would send a notification to the server for each letter the user types.
This is crazy inefficient.
The event listeners should only send a notification when they haven't done so in the last 3 seconds.
You can use [`Date.now()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/now) to get the current time in Javascript.
You can store the last sent timestamp in [*sessionStorage*](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage) like this:
```javascript
// store the timestamp
sessionStorage.setItem('last_notification', Date.now());
// read the timestamp
let lastNotification = sessionStorage.getItem('last_notification');
```

## Receiving updates from the server

We need to send the new list of writing users to all other users in the same thread whenever the list of writers changes.
However, there is a problem - the server cannot make a request to the browser, it can only respond to the request sent by the browser.
The solution is *Server Sent Events (SSE)*.
The client make a request, but server keeps the connection open.
Whenever the server needs to send something to the client, it uses the existing connection.
The connection is only closed when the client no longer wants to receive the events and closes the connection.

How to create a request mapping in the server that keeps the connection open?
You can use the `SseEmitter` class like this:

```java
List<SseEmitter> emitterList = new CopyOnWriteArrayList<>();

@RequestMapping
public SseEmitter eventSource() {
  SseEmitter emitter = new SseEmitter();
  emitterList.add(emitter);
  emitter.onCompletion(() -> {
    // client closed the connection
    emitterList.remove(emitter);
  });
  return emitter;
}

public void sendToAll(String value) {
  for (SseEmitter emitter : emitterList)
    emitter.send(value);
}
```

When you return `SseEmitter` from a request handler, the response is not sent immediately.
Spring will keep the connection to the client open.
When you need to send something to the client later, use the `SseEmitter` instance you returned earlier.

How do you open a connection to the server from the browser to receive the events?
You an use the [`EventSource`](https://developer.mozilla.org/en-US/docs/Web/API/EventSource) class like this:

```javascript
let events = new EventSource('/mapping');
events.addEventListener('message', function(event) {
    console.log('server sent: ' + event.data);
});
```

We now have all the pieces to implement receiving updates from the server:
* add a new request handler to the controller.
  create, store and return a `SseEmitter`.
  the request handler should take the thread id as a parameter, either with a path variable or query parameter.
* add a element to the html thread view for showing the currently writing users for the open thread.
  create the `EventSource` in Javascript which receives events from the server and updates the list of writing users in the html element.
* change the controller in the server, so that when the list of writing users changes, then the new list is sent to all users that have that thread open (but not to other users).

## JSON

JSON (JavaScript Object Notation) is a very popular data format, next to plain text and xml.
Recall that you can declare objects in Javascript like this:
```javascript
let obj = {
    field1: value1,
    field2: value2
};
```

Some people decided that this is a pretty reasonable way to write down data.
Thus JSON was born.
You can use the [`JSON.stringify()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) method to convert a Javascript object to a string that looks just like the Javascript code you would use to create that object.
You can also use the [`JSON.parse()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse) method to convert such string to an actual Javascript object.

You can send java objects from the server to the browser's EventSource or XMLHttpRequest using JSON:
```java
class Train {
  String destination;
  int seats;
}

@RequestMapping
@ResponseBody
public Train getTrainInfo() {
  return new Train("Tartu", 128);
}

public void sendToAll(Train trainInfo) {
  for (SseEmitter emitter : emitterList)
    emitter.send(trainInfo);
}
```

The object will be converted into the JSON format and sent to the browser.
You can parse it into a regular object there:

```javascript
let events = new EventSource('/mapping');
events.addEventListener('message', function(event) {
    let train = JSON.parse(event.data);
    console.log('going to: ' + train.destination);
    console.log('seat count: ' + train.seats);
});

let request = new XMLHttpRequest();
request.addEventListener('load', function(event) {
    let train = request.response;
    console.log('going to: ' + train.destination);
    console.log('seat count: ' + train.seats);
});
request.responseType = 'json';
request.open("GET", "/trains");
request.send();
```
