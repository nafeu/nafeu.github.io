---
layout: post
title: "Real-time Presentation Slides with Socket.io, Node.js and Javascript"
description: "A beginner friendly tutorial on adding realtime functionality in nodejs apps using Socket.io"
date: 2018-10-14
tags: [web, nodejs, socketio, markdown, javascript, dev, realtime]
comments: true
share: true
---

<img src="/images/realtime-slides-tut-sca-1.gif" style="width: 480px;">

Prototyping and building realtime web applications has never been as easy as it is today. There are many libraries that have taken away the complexity of using websocket technology in your projects. In this tutorial, we will look at using **Socket.io** and **Javascript** in conjunction with **Express** and **Node** to build an incredibly simple, minimal presentation slides app that lets you update slides live off any internet connected device. Once completed, you can totally show it off at your next tech talk or lunch & learn.

The demo app for this tutorial is available at: [github.com/nafeu/realtime-slides-tut](https://github.com/nafeu/realtime-slides-tut)

Requirements:

- Node.js >= v7

Some things to note:

- We will try and use as little Socket.io code as possible. This means we won't modify any configurations and will just use the library as it is right out of the box.
- We will add a few basic console logs to help debug and visualize the connection flow but these aren't needed for the app the work.
- For simplicity's sake, we will be writing _on-page_ javascript inside our html files.
- We will use ES6 syntax on both the server and client.

The point of this tutorial is to show you how easily you can add a real-time, interactive component to your applications.

Lets being by creating a new directory for our project and create a **package.json** file inside it:

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
- **[Socket.io](https://socket.io/)** is the real-time engine which uses websockets
- **[Showdown](http://showdownjs.com/)** converts markdown (a simple plain text formatting syntax) into HTML for us

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

Here we are using Express to instantiate an HTTP server that is connected to an Express `app`. When we run it, the console log should show us which port the server is running on (default is `8000`).

Test it out by running `node server.js` and you should see the following output:

```
[ server.js ] Listening on port 8000
```

Everytime we update our **server.js** file we will have to restart our server, thats pretty annoying so lets use a tool called [Nodemon](https://nodemon.io/), install it with `npm install -g nodemon`. Make sure you've ended your original server process, open a new shell instance aside from your main one and run `nodemon server.js`. This way we can keep making changes to our files and the server will restart automatically for us.

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

Back in our **server.js**, lets set up the routes so our server can send our users the correct html files. At the top, require the `path` module, create a new section for our routes and add the following:

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

Don't be thrown off by the routing code, `app.get( ... )` handles a **GET request** to the specified endpoint and the code inside `(req, res) => { ... }` is where we decide what we want to return using the `res` object.

For our app, we want to handle a **GET request** from someone at `/` (the root) and send them our **show.html** file, and then if they go to `/edit` we want to send them **edit.html**.

`path.join(__dirname, 'views/show.html')` builds a clean path to our **show.html** file and `res.sendFile( ... )` sends whichever file exists at that specified path.

Now if we open a web browser to `localhost:8000`, we should see the following:

<img src="/images/realtime-slides-tut-sc-1.png" style="width: 400px;">

And if we go to `localhost:8000/edit`, we should have:

<img src="/images/realtime-slides-tut-sc-2.png" style="width: 400px;">

We can also test out our routes using [curl](https://curl.haxx.se/) to generate **GET Requests**, using `curl localhost:8000` or `curl localhost:8000/edit`, we should see:

```
$ curl localhost:8000/
<h1>view: show</h1>
$ curl localhost:8000/edit
<h1>view: edit</h1>
```

Now that our views and routes are set up, lets get our socket connections up and running. In our **server.js**, require `socket.io` and create an `io` object connected to our `server`. Then add a section for our socket events as follows:

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

The `io` object is what will manage all of our websocket communication from the server-side. Socket.io gives us a very straightforward server-side [API](https://socket.io/docs/server-api/) for event handling.

`io.on('connection', (socket) => { ... })` registers a handle for the `connection` event which is fired off once a client successfully connects. The `socket` object represents our line of communication directly with **one** individual client.

So what we are doing here is logging the `id` for a client once they've connected and logging it again once they've disconnected. Before we can get this to work we have to write our client-side code. Open **show.html** and add the following:

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

The important things to note here is how we are importing the client side library using `<script src="/socket.io/socket.io.js"></script>`. You might be curious, how can we link to this script from our project directory when we never took any steps to put it there in the first place? It seems to be a local link and we never downloaded a `socket.io.js` or placed anything of the sort into a `/socket.io` directory. The reason we can import the library like this is because, by default, Socket.io configures to serve it at `/socket.io/socket.io.js` from our **server.js** file during this step:

<div class="file-path">server.js</div>

```
...
const io = require('socket.io')(server);
...
```

This is super useful when building simple apps like ours but can also be turned off if needed. We will keep it as is for now.

Looking back at our **show.html** file, in the _on-page_ script, we create an instance of the main client-side Socket.io library with `const socket = io();`. Now if we open `localhost:8000` in a web browser and then look back to our server process, we should see something like this:

<img src="/images/realtime-slides-tut-sca-2.gif" style="max-width: 480px;">

The generated `id` will be unique for every connected client. Try opening `localhost:8000` in multiple browser tabs and you will see a bunch of different connection ids logged.

[BLOG POST WIP...]