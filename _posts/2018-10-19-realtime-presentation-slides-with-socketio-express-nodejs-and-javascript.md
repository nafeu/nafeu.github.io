---
layout: post
title: "Real-time Presentation Slides with Socket.io, Express, Node.js and JavaScript"
description: "A beginner friendly tutorial on adding real-time functionality in Node.js apps using Socket.io"
date: 2018-10-19
tags: [web, nodejs, express, socketio, markdown, javascript, dev, realtime]
comments: true
share: true
---

<img src="/images/realtime-slides-tut-sca-1.gif" style="width: 480px; max-width: 100%;">

Prototyping and building realtime web applications has never been as easy as it is today. There are many libraries that have taken away the complexity of utilizing websocket technology. In this tutorial, we will look at using **Socket.io** and **JavaScript** in conjunction with **Express** and **Node** to build an incredibly simple, minimal presentation slides app that lets you update slides live off any internet connected device. Once completed, you can totally show it off at your next tech talk or lunch & learn.

The demo app for this tutorial is available at: [github.com/nafeu/realtime-slides-tut](https://github.com/nafeu/realtime-slides-tut)

Some things to note before we get started:

- This tutorial requires Node.js v7 or higher.
- We will try and use as little **Socket.io** code as possible. This means we won't modify any configurations and will just use the library as it is right out of the box.
- We will add basic console logging to help debug and visualize the connection flow but these aren't needed for the app the work.
- For simplicity's sake, we will be writing _on-page_ javascript inside our html files. In practice, our JavaScript would exist in different files and we would use a build config system like webpack to bundle it all together in an appropriate fashion. We aren't going to bother with that for now. Let's just have some fun.
- We will use ES6 syntax on both the server and client.

#### The point of this tutorial is to show you how **EASILY** you can begin adding real-time interactivity as a part of the apps you build.

Let's begin by creating a new directory for our project and a **package.json** file inside it:

```
mkdir realtime-slides-tut
cd realtime-slides-tut
touch package.json
```

Inside our **package.json**, lets fill in some basic information:

<div class="file-path">package.json</div>

```json
{
  "name": "realtime-slides-tut",
  "version": "0.0.1",
  "description": "Real-time slides tutorial with Socket.io",
  "dependencies": {}
}
```

Now lets install our dependencies:

```
npm install --save express socket.io showdown
```

- **[Express](https://expressjs.com/)** allows us to create the webserver and REST API needed to run our app
- **[Socket.io](https://socket.io/)** is the real-time engine (using websockets)
- **[Showdown](http://showdownjs.com/)** converts **Markdown** (a simple plain text formatting syntax) into HTML for us

The installation should generate a **package-lock.json** file and **node_modules** folder, these are just responsible for maintaining our dependencies.

Lets get our server up and running. Create a **server.js** file and add in the following:

<div class="file-path">server.js</div>

```javascript
const express = require('express');
const http = require('http');
const app = express();
const server = http.Server(app);

// Configuration

server.listen(process.env.PORT || 8000, () => {
  console.log(`[ server.js ] Listening on port ${server.address().port}`);
});
```

Here we are using **Express** to instantiate an HTTP server that is connected to an **Express** `app`. When we run it, the `console.log( ... )` should show us which port the server is running on (default is `8000`).

Test it out by running `node server.js` and you should see the following output:

```
[ server.js ] Listening on port 8000
```

Everytime we update our **server.js** file we will have to restart our server, that is pretty annoying so let's use a tool called [Nodemon](https://nodemon.io/), install it with `npm install -g nodemon`.

Make sure you've ended your original server process, open a new shell instance aside from your main one and run `nodemon server.js`. This way we can keep making changes to our files and the server will restart automatically for us.

Now we want to set up some basic views and routes. Lets create a folder for our views and add **show.html** and **edit.html**:

```
mkdir views
touch views/show.html views/edit.html
```

Your project structure should look like this for the rest of the tutorial:

```
├── node_modules
├── package-lock.json
├── package.json
├── server.js
└── views
    ├── edit.html
    └── show.html
```

Now inside our **show.html** and **edit.html** files, add the following:

<div class="file-path">views/show.html</div>

```html
<h1>view: show</h1>
```

<div class="file-path">views/edit.html</div>

```html
<h1>view: edit</h1>
```

Back in our **server.js**, let's set up the routes so our server can send our users the correct html files based on which page they go to.

At the top, require the `path` module, then create a new section called `Routes` below our server configurations and add the following:

<div class="file-path">server.js</div>

```javascript
const express = require('express');
const http = require('http');
const path = require('path');
const app = express();
const server = http.Server(app);

// Configuration

server.listen(process.env.PORT || 8000, () => {
  console.log(`[ server.js ] Listening on port ${server.address().port}`);
});

// Routes

app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, 'views/show.html'));
});

app.get('/edit', (req, res) => {
  res.sendFile(path.join(__dirname, 'views/edit.html'));
});
```

For those who may be a bit unfamiliar with **Express**, don't be thrown off by the routing code. `app.get( ... )` handles a **GET request** to the specified endpoint and the code inside `(req, res) => { ... }` is where we decide how we want to handle that request. The `res` object allows us to give a response.

Our goal is to do the following:

- User goes to `/` (ie. performs a **GET request** to `localhost:8000/`) => respond with **show.html**
- User goes to `/edit` (ie. performs a **GET request** to `localhost:8000/edit`) => respond with **edit.html**

We want to build a clean path to our html files, so we use `path.join(__dirname, ...)` to build a that path. Then we use `res.sendFile( ... )` to send the file at that specific path.

Now if we open a web browser to `localhost:8000`, we should see the following:

<img src="/images/realtime-slides-tut-sc-1.png" style="width: 400px;">

And if we go to `localhost:8000/edit`, we should have:

<img src="/images/realtime-slides-tut-sc-2.png" style="width: 400px;">

We can also test out our routes using [curl](https://curl.haxx.se/) like so:

```
$ curl localhost:8000/
<h1>view: show</h1>
$ curl localhost:8000/edit
<h1>view: edit</h1>
```

#### Now that our views and routes are set up, lets get our socket connections up and running.

In our **server.js**, require `socket.io` at the top and create an `io` object connected to our `server`. Then add a `Socket Events` section below our routes as follows:

<div class="file-path">server.js</div>

```javascript
const express = require('express');
const http = require('http');
const path = require('path');
const app = express();
const server = http.Server(app);
const io = require('socket.io')(server);

// Configuration

server.listen(process.env.PORT || 8000, () => {
  console.log(`[ server.js ] Listening on port ${server.address().port}`);
});

// Routes

app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, 'views/show.html'));
});

app.get('/edit', (req, res) => {
  res.sendFile(path.join(__dirname, 'views/edit.html'));
});

// Socket Events

io.on('connection', (socket) => {
  console.log(`[ server.js ] ${socket.id} connected`);

  socket.on('disconnect', () => {
    console.log(`[ server.js ] ${socket.id} disconnected`);
  });
});
```

The `io` object is what will manage all of our websocket communication from the server-side. **Socket.io** gives us a very straightforward [server-side API](https://socket.io/docs/server-api/) for event handling.

`io.on('connection', (socket) => { ... })` registers a handle for the `connection` event which is fired off once a client successfully connects. The `socket` object represents our line of communication directly with that **one** client connection.

So take a look at the following:

<div class="file-path">server.js - Socket Events</div>

```
// Socket Events

io.on('connection', (socket) => {
  console.log(`[ server.js ] ${socket.id} connected`);

  socket.on('disconnect', () => {
    console.log(`[ server.js ] ${socket.id} disconnected`);
  });
});
```

What we are doing here is logging the `id` for a client once they've connected, then logging it again once they've disconnected. It will make a lot more sense once we start connecting and disconnecting ourselves, but before we can get this to work, our client needs to know how to connect, so lets write our client-side code.

Open **show.html** and fill it with the following:

<div class="file-path">views/show.html</div>

```html
<!DOCTYPE html>
<html>
<head>
  <title>Real-time Slides Tutorial | Show</title>
  <script src="/socket.io/socket.io.js"></script>
</head>
<body>
  <h1>view: show</h1>
  <script type="text/javascript">

    const socket = io();

  </script>
</body>
</html>
```

An important thing to note here is how we are importing the client-side **Socket.io** library using `<script src="/socket.io/socket.io.js"></script>`. You might be curious, how can we link to this script from our project directory when we never took any steps to put it there in the first place? We never downloaded `socket.io.js` or placed anything of the sort into a `/socket.io` directory.

The reason we can import the library like this is because, by default, **Socket.io** is configured to serve it at this path: `/socket.io/socket.io.js`. This is done inside our **server.js** file during the following step:

<div class="file-path">server.js - Top</div>

```
const io = require('socket.io')(server);
```

This is super useful when building simple apps like ours but can also be turned off if needed. We will keep it as is for now.

Look back at the JavaScript in our **show.html** file:

<div class="file-path">views/show.html - On-Page JavaScript</div>

```
  <script type="text/javascript">

    const socket = io();

  </script>
```

Here we create an instance of the main client-side **Socket.io** library with `const socket = io();`. This `socket` object is what will form the websocket connection with our server.

Now if we open `localhost:8000` in a web browser and then look back to our server process, we should see something like this:

<img src="/images/realtime-slides-tut-sca-2.gif" style="width: 480px; max-width: 100%;">

The generated `id` will be unique for every connected client. Try opening `localhost:8000` in multiple browser tabs and you will see a bunch of different connection ids logged.

#### Now lets begin implementing the logic we need to actually get our presentation slides app going.

We can start by understanding how to handle socket events on the client.

Within the _on-page_ script on **show.html**, add the following:

<div class="file-path">views/show.html - On-Page JavaScript</div>

```
  <script type="text/javascript">

    const socket = io();

    socket.on('update slide', () => {
      alert("UPDATE SLIDE");
    });

  </script>
```

This adds a handle for a new event type we create called `update slide`. We want to have an `alert` pop up on screen anytime we recieve an `update slide` event from our socket connection.

Now that we have that ready, lets figure out how we can actually fire that event within our server.

Go back to the `Socket Events` section in our **server.js**, and underneath our first `console.log()` add `socket.emit('update slide')` like so:

<div class="file-path">server.js - Socket Events</div>

```javascript
// Socket Events

io.on('connection', (socket) => {
  console.log(`[ server.js ] ${socket.id} connected`);

  socket.emit('update slide');

  socket.on('disconnect', () => {
    console.log(`[ server.js ] ${socket.id} disconnected`);
  });
});
```

This is temporary and just for us to demonstrate how the event emission process works between the **Socket.io** server and client. What this will do is fire an `update slide` event directly to any client that connects. It occurs in this order:

1. A client establishes a websocket connection with the server (ie. a single user opens our page).
2. The server (`io` object) handles that connection and gets access to it's relating `socket` object.
3. The server (`io` object) emits an `update slide` event to that client through the `socket` object.
3. The client recieves that `update slide` event and shows an `alert`.

Open `localhost:8000` in your browser to try it out, you should see the `alert`.

#### Now lets modify it a bit so we can send some data along with the events we emit.

<div class="file-path">server.js - Socket Events</div>

```javascript
// Socket Events

io.on('connection', (socket) => {
  console.log(`[ server.js ] ${socket.id} connected`);

  socket.emit('update slide', `Hello ${socket.id}`);

  socket.on('disconnect', () => {
    console.log(`[ server.js ] ${socket.id} disconnected`);
  });
});
```

Here in `socket.emit('update slide', ... )` we are sending `` `Hello ${socket.id}` `` as a payload along with our `update slide` event.

<div class="file-path">views/show.html - On-Page JavaScript</div>

```html
  <script type="text/javascript">

    const socket = io();

    socket.on('update slide', (data) => {
      alert(data);
    });

  </script>
```

In `socket.on('update slide', (data) => { ... })` we are grabbing the payload that we've sent, I've named it `data` here but we can name it however we wish within the relevancy of the data that is being transmitted.

Now if we open `localhost:8000` we should see the following alert message (please note that the generated `id` will be different for you):

<img src="/images/realtime-slides-tut-sc-3.png" style="width: 480px; max-width: 100%;">

Pretty nifty huh? This rough demonstration shows us how we can begin pushing data and firing events between the server and the client. I highly recommend that you bookmark **[Socket.io's Emit Cheatsheet](https://socket.io/docs/emit-cheatsheet/)** for future reference.

For the purpose of this tutorial, we don't have to understand all forms of emit/broadcast interaction between the server and client(s), but it is important to understand the following:

- On the server, `io.emit( ... )` emits the event to **ALL** connected clients
- On the server and client, `socket.emit( ... )` emits the event between a specific client and the server.

#### Now what if we want to trigger a real-time action (like broadcasting a message to all connected clients) using an HTTP request?

Go back to our **server.js** file and update our `Socket Events` like so:

<div class="file-path">server.js - Socket Events</div>

```javascript
// Socket Events

io.on('connection', (socket) => {
  console.log(`[ server.js ] ${socket.id} connected`);

  socket.on('disconnect', () => {
    console.log(`[ server.js ] ${socket.id} disconnected`);
  });
});

function updateSlide(html) {
  io.emit('update slide', html);
}
```

Here we removed that `socket.emit( ... )` code and added a new helper function `updateSlide(html)` which emits an `update slide` event to all clients along with given html as the payload.

Now add an `API` section underneath the `Socket Events` section and fill it in with the following:

<div class="file-path">server.js - API</div>

```javascript
// API

app.get('/api/updateSlide', (req, res) => {
  console.log(`[ server.js ] GET request to 'api/updateSlide' => ${JSON.stringify(req.query)}`);

  const { html } = req.query;

  if (html) {
    updateSlide(html);
    res.status(200).send(`Received 'updateSlide' request with: ${html}\n`);
  } else {
    res.status(400).send('Invalid parameters.\n');
  }
});
```

Now the endpoint `/api/updateSlide` handles a **GET** request, if the query params contain the var `html` (ie. `localhost:8000/api/updateSlide?html=hello%20world`) then we call the `updateSlide(html)` function with the given value, which in turn calls `io.emit('update slide', html)`.

Let's see this in action! Keep a browser window open to `localhost:8000` and in your shell, put in `curl localhost:8000/api/updateSlide?html=hello%20world` (alternatively you can just open `localhost:8000/api/updateSlide?html=hello%20world` in another browser window)

In your shell you should see:

```bash
$ curl localhost:8000/api/updateSlide?html=hello%20world
Received 'updateSlide' request with: hello world
```

In your server's process you should see:

```
[ server.js ] GET request to 'api/updateSlide' => {"html":"hello world"}
```

And in your browser window you should see:

<img src="/images/realtime-slides-tut-sc-4.png" style="width: 480px; max-width: 100%;">

Isn't that awesome? We can use our existing knowledge of REST APIs to add a layer of real-time interaction to our app. It should be noted that you can generate requests using vanilla JavaScript without any extra libraries, so if you wanted to create a kind of **remote control** app to interact with a real-time interface, that **remote control** app doesn't even need to use **Socket.io**, it just has to hit the right API with the right HTTP requests. Neat!

Anyways, back to our app. Let's get our actual page content to update according to these requests. Go to the **show.html** file and update the entire `body` (including the JavaScript) as so:

<div class="file-path">views/show.html - body</div>

```html
<body>
  <div id="slide">
    <h1>Real-time Slides ⏱</h1>
    <p>Improvise your presentations, one slide at a time.</p>
  </div>
  <script type="text/javascript">

    const socket = io();

    socket.on('update slide', (html) => {
      document.querySelector('#slide').innerHTML = html;
    })

  </script>
</body>
```

We've added a `div` with id `slide` and some basic html to create a "slide". In our JavaScript we updated the `update slide` handler to select our `slide` div and replace it's inner html with whatever new html comes in from the server.

Now with the browser window open to `localhost:8000`, when we run those similar `curl` commands, we should get:

<img src="/images/realtime-slides-tut-sca-3.gif" style="width: 480px; max-width: 100%;">

Sweet! We are slowly getting there. We don't actually ever want to manually type html to update our page, so it would be cool if we could type in something much more common and human readable which can then turn into html for us, something familiar like **[Markdown](https://en.wikipedia.org/wiki/Markdown)**.

For this we need a library to do the actual **Markdown** processing for us, this is where we use **[Showdown](http://showdownjs.com/)**.

If you aren't familiar with **Markdown**, here is a quick example. It takes an input like this:

```markdown
#Hello World
Let's build a real-time app!
```

and turns it into:

```html
<h1>Hello World</h1>
<p>Let's build a real-time app!</p>
```

This bars us from having to write all the markup language by hand.

Let's go to our **server.js** and do some simple modifications to incorporate **Showdown**. At the top, require **Showdown** and then create a `converter` object as shown:

<div class="file-path">server.js - Top</div>

```
const express = require('express');
const http = require('http');
const showdown = require('showdown');
const path = require('path');

const app = express();
const server = http.Server(app);
const io = require('socket.io')(server);

const converter = new showdown.Converter();
```

The converter object does what you would expect, it takes **Markdown** and converts it into **html**. We also did a bit of reordering here to make the require statements easier to read.

Now go to the `Socket Events` section and update the `updateSlide(html)` function to `updateSlide(markdown)` as follows:

<div class="file-path">server.js - Socket Events</div>

```javascript
// Socket Events

io.on('connection', (socket) => {
  console.log(`[ server.js ] ${socket.id} connected`);

  socket.on('disconnect', () => {
    console.log(`[ server.js ] ${socket.id} disconnected`);
  });
});

function updateSlide(markdown) {
  io.emit('update slide', converter.makeHtml(markdown));
}
```

And for the last modification to our entire **server.js** file, update the `API` section like so:

<div class="file-path">server.js - API</div>

```javascript
// API

app.get('/api/updateSlide', (req, res) => {
  console.log(`[ server.js ] GET request to 'api/updateSlide' => ${JSON.stringify(req.query)}`);

  const { markdown } = req.query;

  if (markdown) {
    updateSlide(markdown);
    res.status(200).send(`Received 'updateSlide' request with: ${markdown}\n`);
  } else {
    res.status(400).send('Invalid parameters.\n');
  }
});
```

With our existing code, how would we get the following to show up on our screen?:

```html
<h1>Hello World</h1>
<p>Let's build a realtime app!</p>
```

We know that this is generated by `#Hello World\nLet's build a real-time app!` using **Markdown**, so lets safely encode that into our request `localhost:8000/api/updateSlide?markdown=...`

For URL [Percent-encoding](https://en.wikipedia.org/wiki/Percent-encoding):
- hash: `#` is `%23`
- space is `%20`
- newline: `\n` is `%0A`
- apostrophe: `'` is `%27`
- hyphen: `-` is `%2D`

So a **GET** request to our api endpoint using `curl` like this should work:

```bash
curl localhost:8000/api/updateSlide?markdown=%23Hello%20World%0ALet%27s%20build%20a%20real%2Dtime%20app!
```

Let's try it out:

<img src="/images/realtime-slides-tut-sca-4.gif" style="width: 520px; max-width: 100%;">

Voila! We are able to fully update our "slide" with new html derived from simple **Markdown**.

**OBVIOUSLY** we are never going to encode URLs manually, JavaScript can do that for us using the built-in `encodeURIComponent()` function.

Now that our **show** view is working accordingly, let's work on our **edit** view. Open up **edit.html** and replace it's content with the following:

<div class="file-path">views/edit.html</div>

```html
<!DOCTYPE html>
<html>
<head>
  <title>Real-time Slides Tutorial | Edit</title>
</head>
<body>
  <textarea rows="8" placeholder="Enter markdown"></textarea>
  <div id="submit-button" onclick="handleSubmit()">Submit</div>

  <script type="text/javascript">
    const textarea = document.querySelector("textarea");

    function sendUpdateSlideRequest(markdown) {
      const { protocol } = window.location;
      const url = `${protocol}/api/updateSlide?markdown=${encodeURIComponent(markdown)}`;
      const xhttp = new XMLHttpRequest();
      xhttp.open("GET", url, true);
      xhttp.send();
    }

    function handleSubmit() {
      if (textarea.value.length > 0) {
        sendUpdateSlideRequest(textarea.value);
      }
      textarea.value = "";
    }

  </script>
</body>
</html>
```

Here we have a simple `textarea` where we can enter our **Markdown** and we've got a `div` with the id `submit-button` that once clicked will trigger the `sendUpdateSlideRequest(markdown)` helper function which generates a **GET** request to the endpoint `api/updateSlide`. It does so with the appropriately encoded data in the URL params.

Now if we open one browser window to `localhost:8000` and one to `localhost:8000/edit`, this is what we get:

<img src="/images/realtime-slides-tut-sca-5.gif" style="width: 520px; max-width: 100%;">

#### Functionality wise, we are close to complettion. Now we will add some additional features and styling for polish.

Let's add the ability to pre-load a slide deck and save submitted slides in our **edit** view.

Open up **edit.html** and modify it as so:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Real-time Slides Tutorial | Edit</title>
</head>
<body>
  <textarea rows="8" placeholder="Enter markdown"></textarea>
  <div id="submit-button" onclick="handleSubmit()">Submit</div>
  <div id="deck"></div>
  <script type="text/javascript">
    const textarea = document.querySelector("textarea");
    const deck = document.querySelector("#deck");
    const slides = [
      '#Have Some Fun! 🎉\nFork this project and experiment with the real-time logic.',
      '#Clever Libraries 🛠\nPowered by Express.js, Socket.io, Showdown and some sweet Vanilla JS',
    ];

    function sendUpdateSlideRequest(markdown) {
      const { protocol } = window.location;
      const url = `${protocol}/api/updateSlide?markdown=${encodeURIComponent(markdown)}`;
      const xhttp = new XMLHttpRequest();
      xhttp.open("GET", url, true);
      xhttp.send();
    }

    function updateDeck() {
      deck.innerHTML = "";
      slides.forEach((markdown) => {
        const slideNode = document.createElement("p");
        slideNode.innerText = markdown;
        slideNode.onclick = () => {
          sendUpdateSlideRequest(markdown);
        }
        deck.appendChild(slideNode);
      })
    }

    function handleSubmit() {
      if (textarea.value.length > 0) {
        slides.unshift(textarea.value);
        updateDeck();
        sendUpdateSlideRequest(textarea.value);
      }
      textarea.value = "";
    }

    (() => {
      updateDeck();
    })();
  </script>
</body>
</html>
```

We aren't doing anything particularly clever here, so I'll let you debunk how the additional code works.

In layman's, we are storing a set of slide data (like a deck) and generating some `div`s out of that data. Clicking on those `div`s submits that slide again to our API so it can update what is visible on the **show** view. Everytime we submit a new slide using the text field, that data gets added into our deck. This is what we get:

<img src="/images/realtime-slides-tut-sca-6.gif" style="width: 520px; max-width: 100%;">

Now for some finishing touches!

Add the following styles to your **show.html** file:

<div class="file-path">views/show.html - Style</div>

```html
  <style type="text/css">
    body {
      background-color: #f1f2f6;
      color: #222f3e;
      font-family: 'Helvetica', 'Arial', sans-serif;
    }

    h1, h2, h3, h4, h5, p {
      margin-bottom: 0px;
    }

    #slide {
      margin: auto;
      height: 60%;
      width: 80%;
      position: fixed;
      top:0;
      bottom:0;
      left:0;
      right:0;
      font-size: 5vmin;
      display: inline-block;
    }
  </style>
```

And finally, add the following styles to your **show.html** file:

<div class="file-path">views/edit.html - Style</div>

```html
  <style type="text/css">
    body {
      background-color: #f1f2f6;
      color: #222f3e;
      font-family: 'Helvetica', 'Arial', sans-serif;
      font-size: 1.25em;
      width: 100%;
      margin: 0;
    }

    *:focus {
      outline: none;
    }

    textarea {
      width: 100%;
      border: none;
      margin-bottom: 20px;
      resize: none;
      padding: 25px;
      font-size: 1.25em;
    }

    #submit-button, #deck p {
      cursor: pointer;
    }

    #submit-button:active, #deck p:active {
      opacity: 0.5;
    }

    #submit-button {
      padding: 25px 25px 25px 25px;
      text-align: center;
      margin: 0px 25px 25px 25px;
      border-radius: 5px;
      background-color: #01a3a4;
      color: white;
    }

    #deck {
      padding: 0px 25px 0px 25px;
    }

    #deck p {
      background-color: white;
      padding: 25px;
      border-radius: 5px;
      margin-top: 0px;
    }
  </style>
```

And you finish off with:

<img src="/images/realtime-slides-tut-sca-1.gif" style="width: 100%;">

Hope you enjoyed the tutorial! If you have any questions, feel free to comment below.

The demo project is available at [github.com/nafeu/realtime-slides-tut](https://github.com/nafeu/realtime-slides-tut) so check it out, fork it and have some fun!

Cheers