---
layout: post
title: Building React Without Create React App
subtitle: What does CRA do?
thumbnail-img: /assets/img/react.png
tags: [JS, React]
comments: true
author: Hank Kim
---

# Building React Without Create React App

## What is CRA?

Create React App is Facebook's command-line tool for generating React applications. It comes with transpiling, code bundling, and all the environmental setup already configured. Pretty convenient, right?

But here's the thing - while CRA makes our lives easier, it's pretty hard to customize the build environment. Sure, you can use `eject`, but once you do that, you kind of lose the whole point of using CRA in the first place.

So let's dive into building a React app from scratch and see what's really going on behind the scenes.

## Setting Up Our Project

### 1. Creating Files and npm init

First things first - let's create our `index.html` and `index.js` files, then initialize npm:

```bash
mkdir my-react-app
cd my-react-app
npm init -y
```

### 2. Installing Dependencies

```bash
npm install react react-dom
```

These are the main React libraries we need:

```javascript
// src/App.js
import React from "react";

const App = () => {
  return <div>HI</div>;
};

export default App;
```

The reason we can use JSX syntax here is because we're importing the React library. In CRA, this import happens automatically behind the scenes, so you don't have to write it explicitly.

```javascript
// src/index.js
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";

ReactDOM.render(<App />, document.getElementById("root"));
```

ReactDOM is what actually renders our component into the `root` element in our HTML file.

## Setting Up Webpack

Now for the fun part - let's install webpack and its friends:

```bash
npm install webpack webpack-cli webpack-dev-server html-webpack-plugin --save-dev
```

Let me break down what each of these does:

### webpack-cli

This is the webpack Command Line Interface. It gives us convenient commands for webpack configuration like `webpack serve` and `webpack build`.

### webpack-dev-server

This provides a dev server that auto-reloads when you save files. Remember how CRA automatically refreshes your browser when you make changes? That's because it uses webpack-dev-server internally.

### html-webpack-plugin

This automatically loads our bundled JS file into the HTML file. Instead of manually adding:

```html
<script src="index.js" type="module"></script>
```

to our HTML, the plugin does it for us. The entire webpack bundle is essentially one big immediately-invoked function expression (IIFE).

### About --save-dev

The `--save-dev` option tells npm that these packages are only needed during development, not in production. They get installed to `devDependencies`.

The opposite would be `--save-prod` (or just `--save`), which adds packages to regular `dependencies`. When you run `npm install`, it downloads everything regardless, but if you use `npm install --production`, it skips the devDependencies.

## Installing Babel

```bash
npm install @babel/core babel-loader @babel/preset-react @babel/preset-env --save-dev
```

Here's what each babel package does:

### @babel/core

This is the core Babel library that handles the main functionality.

### babel-loader

Loaders tell webpack how to process different file types and what format to return them in. For example:

- `sass-loader` converts SASS files to CSS
- `css-loader` works in webpack and lets you import CSS files as modules in JS
- `style-loader` injects CSS into the DOM using style tags
- `babel-loader` compiles JS files to different versions/polyfills

### @babel/preset-react

A preset is basically a pre-configuration. JSX stands for JavaScript XML and lets you write HTML-like code inside JavaScript. This library helps Babel understand and process JSX files.

### @babel/preset-env

This helps support the latest JavaScript features and reduces bundle size by only including the polyfills you actually need based on your target browsers.

## Webpack Configuration

Now let's create our `webpack.config.js` file in the root folder:

### Setting the Entry File

The entry file becomes the root of our dependency graph. When one file depends on another, that's called a dependency. Webpack starts from the entry file and recursively builds the dependency graph. This is important because modules that other modules depend on need to be built first.

### Output Configuration

```javascript
output: {
  path: path.join(__dirname, "/dist"),
  filename: "bundle.js",
}
```

This tells webpack where to save the bundled JS file and what to name it.

### Plugins

```javascript
plugins: [
  new HTMLWebpackPlugin({
    template: "./src/index.html",
  }),
];
```

This applies the plugin to the template file we specify.

### Module Rules

```javascript
module: {
  rules: [
    {
      test: /.js$/,
      exclude: /node_modules/,
      use: {
        loader: "babel-loader",
        options: {
          presets: ["@babel/preset-env", "@babel/preset-react"],
        },
      },
    },
  ],
}
```

This tells webpack how to handle different types of modules. In this case: "For files ending in `.js`, excluding `node_modules`, use babel-loader to transpile them."

### Complete webpack.config.js

```javascript
const path = require("path");
const HTMLWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  entry: "./src/index.js",
  output: {
    path: path.join(__dirname, "/dist"),
    filename: "bundle.js",
  },
  plugins: [
    new HTMLWebpackPlugin({
      template: "./src/index.html",
    }),
  ],
  module: {
    rules: [
      {
        test: /.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader",
          options: {
            presets: ["@babel/preset-env", "@babel/preset-react"],
          },
        },
      },
    ],
  },
};
```

## Creating npm Scripts

Let's add these scripts to our `package.json`:

```javascript
"scripts": {
  "start": "webpack-dev-server --mode development --open --hot",
  "build": "webpack --mode production"
}
```

- `start`: Runs webpack-dev-server in development mode, opens a new browser window, and enables hot reloading for instant changes
- `build`: Runs webpack in production mode to create build files

If you run `npm start` now, everything should work! But we still can't use CSS yet.

## Adding CSS Support

```bash
npm install css-loader style-loader --save-dev
```

### Adding a Webpack Rule for CSS

```javascript
{
  test: /\.css$/,
  use: ["style-loader", "css-loader"],
}
```

This means "for files ending in `.css`, process them through style-loader and css-loader."

The css-loader takes imported CSS files, randomizes the selectors, and converts them to JSON format, then passes them to style-loader. The style-loader then injects them into the DOM using style tags.

If you need other assets like images, you can install `file-loader` and apply it the same way.

## Adding ESLint and Prettier

```bash
npm install eslint --save-dev
npm install eslint-config-prettier eslint-plugin-prettier eslint-plugin-react --save-dev
```

This installs ESLint along with Prettier-related packages and React-specific packages.

### Creating .eslintrc

The "rc" stands for "run command":

```javascript
{
  "env": {
    "browser": true,
    "es2021": true,
    "node": true
  },
  "extends": [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:prettier/recommended",
    "eslint-config-prettier"
  ],
  "parserOptions": {
    "ecmaFeatures": {
      "jsx": true
    },
    "ecmaVersion": 12,
    "sourceType": "module"
  },
  "plugins": ["react", "prettier"],
  "rules": {
    "prettier/prettier": "warn",
    "@typescript-eslint/no-var-requires": 0
  }
}
```

### Creating .prettierrc

```javascript
{
  "printWidth": 100,
  "singleQuote": true,
  "tabWidth": 2,
  "useTabs": false
}
```

## Wrapping Up

And that's it! We've successfully created a React application without CRA and configured webpack, babel, eslint, and prettier from scratch.

If you want to go further, you could add TypeScript support and move the babel configuration to a separate `babel.config.js` file for better organization. For performance improvements, consider adding `CleanWebpackPlugin`.

A quick note about file handling: instead of `file-loader`, you can use `copyWebpackPlugin`. The difference is that `file-loader` lets you import file URLs in your code, while `copyWebpackPlugin` simply moves files from one location to another.
