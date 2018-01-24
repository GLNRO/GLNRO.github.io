---
title: Up And Running With React & Redux & Three.JS without the copying/pasting...Part 1
date: 2018-25-01 16:37:05
tags: react redux three.js
---
I am always tremendously grateful for the pedagogical stewards of 'hacking around on the internet' who take the time to explain how to get up and running with the new Derp.js framework or Whale Box, the containerized deployment platform. After years of procrastination and being too busy to write blog posts, I will humbly attempt to throw in my two pennies about bootstrapping a React/Redux application meant to serve Three.js content.


### Initialization
Packages for creating an app quick like create-a-react-app are nice to spike out a project but obscure what's going on behind the scenes. For that reason, i'll delve into why everything we require is being used and what its purpose is. It's easy to take for granted that readers will know the basics of how to initialize a package or what babel loaders are when you use these tools every day, so I've tried to explain everything in a way that might seem overly detailed. If you think something is incorrect or could use additional explanation, I'm happy to take the feedback.

1. Initialize the application (you should have node/npm installed already)
```
npm init
```

2. Install application dependencies
```
npm install --save react react-dom react-redux redux redux-saga isomorphic-fetch webpack webpack-dev-server
```
We install [react](https://github.com/facebook/react) and [react-dom](https://www.npmjs.com/package/react-dom) for access to the react library and facilitate an entry point for rendering the application into the dom. We install [redux](https://redux.js.org/) for the library and [react-redux](https://github.com/reactjs/react-redux) for the react bindings for redux. Further, we'll install [redux-saga](https://github.com/redux-saga/redux-saga) for when we are managing side-effects/asynchronous tasks. [Isomorphic-fetch]() is the library we'll use for making api calls, and [webpack](https://webpack.js.org/) is the module bundler for the project, and [webpack-dev-server](https://github.com/webpack/webpack-dev-server) for our live-reloading dev serveer. Webpack could take up its own post, so check out this [survivejs post](https://survivejs.com/webpack/what-is-webpack/) for further reading.

3. Install language dependencies

Babel
```
npm install --save babel-core babel-loader babel-polyfill babel-preset-env babel-preset-react html-webpack-plugin
```
We use Babel to transpile javascript to the current version your browser supports. It's effectively future proofing your javascript so you can leverage new syntax while making sure your content is available for everyone.

[babel-core](https://github.com/babel/babel/tree/master/packages/babel-core) Babel's core library
[babel-loader](https://github.com/babel/babel-loader) The Babel plugin used for Webpack
[babel-polyfill](https://babeljs.io/docs/usage/polyfill/) Babel polyfill is the polyfill for older web browsers allowing for the use of new ES6 built-ins (ie Map/Promise)
[babel-preset-env](https://github.com/babel/babel/tree/master/packages/babel-preset-env) Preset env replaces preset-2015 and compiles anything post 2015 down to ES5
[babel-preset-react](https://github.com/babel/babel/tree/master/packages/babel-preset-react) Preset for all react plugins
[html-webpack-plugin]() HTML plugin for webpack

4. Install testing dependencies

For testing and debugging, we'll use [enzyme](https://github.com/airbnb/enzyme) for testing react,  [fetch-mock](https://github.com/wheresrhys/fetch-mock) for mocking our api requests, [react-test-renderer](https://www.npmjs.com/package/react-test-renderer) for testing rendering without the dom, [redux-mock-store](https://github.com/arnaudbenard/redux-mock-store) to mock the store for integration testing, and [redux-logger](https://github.com/evgenyrodionov/redux-logger) for debugging redux actions and state change in the console.

```
npm install --save-dev enzyme fetch-mock react-test-renderer redux-mock-store redux-logger
```


### Project setup

Now that project dependencies installed, go ahead and setup the project directory structure. We'll have and src/ directory for our models/components and services, a test/ directory, and a public/ directory for our bundled application output (or dist/ if you prefer).

In src/ we'll need an App.js, and an index.js. Our public/ directory will only hold our index.html for now.

src/App.js will just return some html at present, but will eventually contain our routes and top level components or containers
```
import React, {Component} from 'react';

export default class App extends Component {

  render(){
    return(
      <div>
        <h1>Hello World</h1>
      </div>);
  }
}
```

src/index.js is tasked with finding the html container in our index.html and rendering our application inside (<div> '#main' in this case, but you can give it any id).
```
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

ReactDOM.render(<App />, document.getElementById('main'));
```

Finally public/index.html will be the entry point for our application to be rendered.
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>App Title</title>
  </head>
  <body>
    <noscript>
      You need to enable JavaScript to run this app.
    </noscript>
    <div id="main"></div>
  </body>
</html>
```

### Project configuration

Now we'll need to go ahead and configure webpack and babel to compile our application code. Make sure to touch a webpack.config.js and .babelrc in the root directory. You can alternatively place configuration files in a separate config/ directory, just make sure to resolve the paths in package.json as well as the webpack config.

Configure your .babelrc to include the presets for our application. Any plugins can be specified as well, in this case, transform-object-rest-spread as an example.
```
{
		"presets": ["react", "env"],
    "plugins": [
      "transform-object-rest-spread"
      ]
}
```

Our webpack config will first set our path variable to resolve absolute paths, and require our html-webpack-plugin

In the module.exports, the entry for our application will be the index.js in the src/ directory. The output (our bundled app) will be named bundle.js and will be built in the public directory.

There are a series of [loaders](https://webpack.js.org/loaders/) to bundle static resources; This project uses loaders for jsx, sass/css, as well as url and image loaders.

The dev server is configured to find the bundled output of the application in the public/ directory, and serve the application on port 3001.

Finally the only plugin used for this project will be the html webpack plugin required earlier. We set the template as the full path and filename of index.html.
```
let path = require('path');
const HtmlWebPackPlugin = require("html-webpack-plugin");

module.exports = {
  entry: './src/index.js',
  output:{
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'public')
  },
  module: {
    loaders: [
      {
        test: /.jsx?$/,
        loader: 'babel-loader',
        exclude: /node_modules/,
        query: {
          presets: ['env', 'react'],
          plugins: ['transform-object-rest-spread'],
        },
      },
      { test: /\.s?css$/, use: [{ loader: 'style-loader' }, { loader: 'css-loader' }, { loader: 'sass-loader' }] },
      { test: /\.woff(2)?(\?v=[0-9]\.[0-9]\.[0-9])?$/, loader: 'url-loader?limit=10000&mimetype=application/font-woff' },
      { test: /\.(ttf|eot)(\?v=[0-9]\.[0-9]\.[0-9])?$/, loader: 'file-loader' },
      { test: /\.(gif|png|jpe?g|svg)$/i, use: ['file-loader', { loader: 'image-webpack-loader', options: { bypassOnDebug: true } }] },
    ],
  },
  devServer: {
        contentBase: path.resolve(__dirname, "public"),
        port: 3001
      },
  plugins: [
        new HtmlWebPackPlugin({
            template: "./public/index.html",
            filename: "./index.html"
          })
        ]
};
```
Finally we'll add two commands, build and start, to the package.json scripts.

```
...
"main": "index.js",
  "scripts": {
    "build": "node_modules/.bin/webpack",
    "start": "node_modules/.bin/webpack-dev-server",
  },
  "repository": {
...
```

### Build

Build the application by running
```
npm run build
```

### Running in dev

Run the application in dev with
```
npm run start
```

You should be able to load the appliction in the browser at localhost:3001



Check out Part 2 coming soon to setup redux. Part 3 will document the setup of a basic three.js scene hooked up to our redux state.
