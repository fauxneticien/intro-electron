# 1. FAQs

## Q. What is the difference between JavaScript and Node.js?

> All browsers have JavaScript engines that run the JavaScript of web pages. Firefox has an engine called Spidermonkey, Safari has JavaScriptCore, and Chrome has an engine called V8. 

> Node.js is simply the V8 engine bundled with **some libraries to do I/O** and networking, **so that you can use JavaScript outside of the browser**, to create shell scripts, backend services or run on hardware (https://tessel.io/).

From [https://www.quora.com/What-is-the-difference-between-JavaScript-and-Node-js](https://www.quora.com/What-is-the-difference-between-JavaScript-and-Node-js)

### Examples:

```javascript
// Runs in both Chrome and in Node
console.log("Hello world");

// Runs only in Node
const fs = require('fs') 	// Load filesystem library for I/O

// Runs only in Chrome (i.e. the browser)
document.location      		// The DOM is not available in Node

// Runs neither in Chrome nor Node
typeof JSONscriptRequest	// This is a Safari-specific function
```

## Q. What is Electron?

> **Electron is a framework for building cross-platform desktop apps in Javascript, HTML, and CSS**. The folks at GitHub somehow managed to cram the Node.js runtime into the Chromium web browser, letting developers combine the flexibility of HTML and CSS with the ever expanding ecosystem of over 380,000 Node modules.
> 
> ...
> 
> This post has covered all of my favourite things about Electron, but those things come at a high cost. **Since every Electron app is essentially a Chromium web browser running your JavaScript code, even the most basic applications have a base size of 150MB (~60MB zipped) and consume 100MB of RAM**.

> You may want to think twice before choosing Electron to build that slick new weather widget of yours 🙊

From [https://insomnia.rest/blog/electron-is-a-dream/](https://insomnia.rest/blog/electron-is-a-dream/)

# 2. A simple web page (`simple-page.html`)

When you click the `Do the thing` button, the action triggers the function `doTheThing()`, and this produces an alert with the message `You did the thing`.

```html
<!DOCTYPE html>
<html>
<head>
	<title>Doing the thing</title>
</head>
<body>

<input type="button" onclick="doTheThing()" value="Do the thing">

<script type="text/javascript">
	
	doTheThing = function() {
		alert("You did the thing")
	}

</script>

</body>
</html>

```

# 3. Bundling `simple-page.html` into an Electron app

## Run `npm init`

`cd` into where you have `simple-page.html` saved, and run `npm init` to initialise the directory as a node module (this essentially helps you create a `package.json` in the folder). Press enter for all the prompts, and you'll end up with something like this:

```json
{
  "name": "intro-electron",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

## Run `npm install electron --save-dev`

Run `npm install electron --save-dev` to save electron as a development dependency for your app. After it finishes, you'll notice your `package.json` change to:

<pre>
{
  "name": "intro-electron",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  <b style="color:green">"devDependencies": {
    "electron": "^1.7.5"
  }</b>
}
</pre>

## Create `index.js`

Notice the `"main": "index.js"` part of the `package.json` file above. This is the "entry point" of your app, so we'll need to create `index.js` in order to tell Electron/node what to do when the app starts up.

It looks somewhat busy, since do have to start up a bunch of modules for the interface (notice various `BrowserWindow`, `mainWindow.WebContents`, etc.), as V8 in node usually does not concern itself with these things. The main thing to notice for now is the `mainWindow.loadURL` function, and how we're pointing it to `simple-page.html`.

<pre>
const electron = require('electron')
const app = electron.app
const BrowserWindow = electron.BrowserWindow

const path = require('path')
const url = require('url')

let mainWindow

function createWindow () {
  mainWindow = new BrowserWindow({width: 800, height: 600})

  mainWindow.loadURL(url.format({
    pathname: path.join(__dirname, '<b style="color:green">simple-page.html</b>'),
    protocol: 'file:',
    slashes: true
  }))

  // mainWindow.webContents.openDevTools()

  mainWindow.on('closed', function () { mainWindow = null })
}

// Event handlers

app.on('ready', createWindow)

app.on('window-all-closed', function () {
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

app.on('activate', function () {
  if (mainWindow === null) {
    createWindow()
  }
})
</pre>

## Run `./node_modules/.bin/electron .`

This command calls `electron` in the current directory, and now you should see your `simple-page.html` launched as an Electron app (i.e. bundled with an instance of Chromium).

### Make life easier

You can also specify this long-ish command (or any others) as an npm script inside `package.json`, as shown below. So now instead of having to type `./node_modules/.bin/electron .` each time, just type `npm start`.

```json
{
  "name": "intro-electron",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "./node_modules/.bin/electron ."
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "electron": "^1.7.5"
  },
  "dependencies": {
    "electron": "^1.7.5"
  },
  "description": ""
}
```

# 4. Some filesystem I/O with `node`

In a new Terminal session, type:

1. `node`, to enter into a node environment (e.g. same as `python`, `R`, etc.).
2. `const fs = require('fs')`, to load the filesystem module into the constant `fs`.
3. `fs.readdirSync('.')`, to read the contents of whatever directory you started `node` in.

# 5. Electron: using node modules from the browser

## 5.1 Simple example with `fs`

As we discussed above, the `fs` module is a node-specific feature, and can't be used in the browser. Try it out in the Chrome Javascript Console—it doesn't work! But with Electron, we can indeed use node modules within "client-side" Javascript (the line between "client" and "server" gets really blurry, but that's a topic for another day).

Replace your `doTheThing` function in `simple-page.html` with the snippet below, then run your electron app with `electron .`. Notice now that the alert is no longer `You did the thing`, but is a list of files.

```html
<script type="text/javascript">

const fs = require('fs');

doTheThing = function() {
	var files = fs.readdirSync('.').join('\n')
	alert(files)
}

</script>
```

## 5.2 Using Electron dialog boxes

Let's add a `<pre id="file_contents"></pre>` element to load some file's data into.

```html
<pre id="file_contents"></pre>

<input type="button" onclick="doTheThing()" value="Do the thing">

<script type="text/javascript">

	const { dialog } = require('electron').remote;
	const fs = require('fs');
	
	doTheThing = function() {
		var files    = dialog.showOpenDialog({ properties: ['openFile'] });
		var contents = fs.readFileSync(files[0]).toString();
	
		document.getElementById("file_contents").innerHTML = contents;
	}

</script>

</body>
```

# 6 Packaging your Electron app

1. Grab the `electron-packager` package

```
npm install electron-packager --save-dev
```

2. Build the Electron app into a `.app` file and it will appear in the `build/osx` file:

```
./node_modules/.bin/electron-packager . build/osx --platform=darwin
```
