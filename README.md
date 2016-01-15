# Intro to Node

# Prerequisites:

- Knowledge of basic JavaScript
- Knowledge of basic Terminal commands
- [`node`][node] installed on your computer

# Tutorial

First start by creating a new folder, and navigating to it in your terminal:

```sh
$ mkdir node-intro && cd $_
```

Create a new file called `index.js`, and give it the contents:
`console.log('hello world')`.

```sh
$ echo "console.log('hello world');" >> index.js
```

Now run the node program by typing the following:

```sh
$ node index
hello world
```

If you got the output `hello world`, then you've got node set up correctly!

Let's start by creating a simple http server. We will include the core `http`
module. Remove the `console.log` statement from `index.js` and replace it with
the following content.

```js
var http = require('http');

http.createServer(function(request, response) {
  // This function gets called on every http request.

  console.log(request.url);

  // This writes 'Hello world' to the response and ends it.
  response.end('Hello world');

}).listen(8000, function() {
  // This call starts the server listening on port 8000
  console.log('Server listening on port 8000');
});
```

Now in your browser, type in `http://localhost:8000`. If you get `Hello world`
in the browser, you're server is running correctly. Notice that every time you
refresh your page, it adds logs of the url.

Also notice that no matter what path you put into the address bar after the
`localhost:8000`, it always says `Hello world`. He haven't hooked up any
routing or anything.

To quit the server, press `ctrl` + `c`.

Let's expand this to be a static file web server. We'll take in the request
path and route it to files in a directory. So lets create a new directory and
create an `indes.html` file in it. Let's call the directory `public`.

```sh
$ mkdir public
$ touch public/index.html
```

Now add in some html boilerplate and an `<h1>` tag in the body with a title of
your choosing.

Let's add some file reading to our `index.js`. We're going to use the `fs`
core module to read the index.html file we just created. We're also going to use
the `path` module to normalize our file paths for both Windows and Unix file
paths. We will also make use the node's `__dirname` to get the current directory
of the `index.js` file:

```js
var http = require('http');
var fs = require('fs');
var path = require('path');

var PUBLIC_DIR = path.join(__dirname, 'public');

http.createServer(function(request, response) {
  // This function gets called on every http request.

  console.log(request.url);

  // This joins the PUBLIC_DIR and 'index.html' into a single path by joining
  // them by the operating system's path separator.
  var filepath = [ PUBLIC_DIR, 'index.html' ].join(path.sep);
  // now we read the index.html file's contents
  var contents = fs.readFileSync(filepath);

  response.end(contents);

}).listen(8000, function() {
  // This call starts the server listening on port 8000
  console.log('Server listening on port 8000');
});
```

Now start the server up again. Now when you hit the server, it should respond
with your html page. Without having to restart the server, you should be able
to make changes to `public/index.html` and have it reflect in subsequent
requests because it reads the file on every request.

This isn't the most efficient way of reading the file because every time the
file gets read, it will block other requests until it finishes reading the file.

Here is how we could change that call:

```js
var http = require('http');
var fs = require('fs');
var path = require('path');

var PUBLIC_DIR = path.join(__dirname, 'public');

http.createServer(function(request, response) {
  // This function gets called on every http request.

  console.log(request.url);

  // This joins the PUBLIC_DIR and 'index.html' into a single path by joining
  // them by the operating system's path separator.
  var filepath = [ PUBLIC_DIR, 'index.html' ].join(path.sep);
  // now we read the index.html file's contents
  // This does it asynchronously, which means it won't block other requests as
  // it reads the file.
  fs.readFile(filepath, function(err, contents) {
    response.end(contents);
  });

}).listen(8000, function() {
  // This call starts the server listening on port 8000
  console.log('Server listening on port 8000');
});
```

The server should work the same way as before, just be slightly more performant.
Let's make it a little more dynamic. We'll take the request url and translate it
to a filepath within the public directory.

```js
var http = require('http');
var fs = require('fs');
var path = require('path');

var PUBLIC_DIR = path.join(__dirname, 'public');

http.createServer(function(request, response) {
  // This function gets called on every http request.

  var filepath = request.url;
  // Make a trailing `/` be converted to `/index.html`
  if (filepath[filepath.length - 1] === '/') { filepath += 'index.html'; }
  // Split by the browser path separator
  filepath = filepath.split('/');
  // Replace the empty string in the first element with the PUBLIC_DIR
  filepath[0] = PUBLIC_DIR;
  // Join them by the path separator
  filepath = filepath.join(path.sep);

  // now we read the index.html file's contents
  // This does it asynchronously, which means it won't block other requests as
  // it reads the file.
  fs.readFile(filepath, function(err, contents) {
    // Check for an error
    if (err) {
      response.writeHead(404);
      response.end('Not Found');
      return; // End the response
    }

    response.end(contents);
  });

}).listen(8000, function() {
  // This call starts the server listening on port 8000
  console.log('Server listening on port 8000');
});
```

Now we have a working static file server. This is a lot of custom code to get
a static file server working, but it's a lot less code than Apache.

Later, we'll get into `npm` and other modules that make making servers a lot
easier than having to do it manually.

[node]: https://nodejs.org/en/
