---
title: "Build tools"
weight: 30
description: >
  Toolchains to manage your environments and distribute your code.
---


## History

In its simplest form, Javascript files are linked directly in the HTML `<head>` with the `<script>` tag. If you want to add an external library, you would have to download the library, store the library in a file, and then link that file with another `<script>` tag:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    ...
    <script src="library.min.js"></script>
    <script src="index.js"></script>
  </head>
  ...
</html>
```

This creates a global variable (maybe named `library`) that is avaiable to any file loaded under the `library.min.js` script tag. Managing packages like this is tedious, so frontend developers use a package manager like `npm` to handle packages. `npm` creates a `package.json` file to track project dependencies, which allows you to easily share projects.

After you add a library to the project, npm creates the `node_modules/` directory which contains all downloaded libraries as modules. There is an issue--Javascript runs in the browser, and the browser cannot access local directories. You could explicitly link the desired library in `node_modules` with a script tag containing the path (`node_modules/librar/library.min.js`), but that is tedious, too.

## npm

`npm` is the node package manager, which handles dependencies for Node.js. It consists of the following:

- [website](https://www.npmjs.com/)
- [CLI](https://docs.npmjs.com/cli/v10/commands/npm)
- [registry](https://docs.npmjs.com/cli/v10/using-npm/registry): the database of JS software

You can use it to manage dependencies for your Javascript projects with the following commands:

```shell
# create package.json file
$ npm init

# create default package.json file
$ npm init -y

# set default options for init command
$ npm set init-author-email "example-user@example.com"
$ npm set init-author-name "example_user"
$ npm set init-license "MIT"

# save as prod dependency and included in production version
$ npm install <package-name> --save

# save as dev dependency, NOT included in production version
$ npm install <package-name> --save-dev

# install the dependencies lists in package.json
$ npm install

# install a specific version with a tag
$ npm install <package-name>@<tag>

# check the node_modules contents
$ ls node_modules
```

_`devDependencies`_ are added to `package.json` when a package is installed as a development dependency. Examples include babel plugins and presets, test runners, and linters. Those listed under _`dependencies`_ are production dependencies. Development dependencies are included in the application so they are present at runtime. Development `dependencies` are available in both development and production, so any packages that are used in both workflows are listed under `dependencies` only.

## Webpack

Webpack is a bundler. A bundler takes all of your application dependencies and "bundles" them up into a single static file that is available to your application at runtime. As a general rule, you should try to limit the number of production dependencies that your application relies on.

A bundler takes an entry point and builds a dependency graph of your source files and third-party libraries. This is called **dependency resolution**.

Node.js uses the `require()` statement to import libraries from the filesystem, but this does not work with Javascript in the browser because there is no file system access. The solution is a module bundler.

A Javascript module bundler has a build step that accesses the file system and makes JS files available to the browser. Essentially, it finds all `require()` statements in `.js` files and replaces them with the contents of the required file (npm tracks the file system location of all dependencies).

Read the [Webpack docs](https://webpack.js.org/) and the [Webpack developer docs](https://webpack.js.org/guides/development/).

### Key concepts

These concepts were taken from [this article](https://snipcart.com/blog/javascript-module-bundler) and defined in the [official docs](https://webpack.js.org/concepts/).

entry
: The file that Webpack uses to initiate its dependency graph:

```js
module.exports = {
  entry: "./app/index.js",
};
```

output
: The location that Webpack outputs the packaged code. This usually includes the output path and the output filename:

```js
module.exports = {
  entry: "./app/index.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "webpack-app.bundle.js",
  },
};
```

loaders
: Transform and bundle non-JS files.

plugins
: Webpack can perform more advanced features.

mode
: Dynamically configures operations to production or development modes.

Browser compatiblity
: Build bundles that support new and old browsers.

### Commands

```shell

# start a project
$ mkdir project-name
$ cd project-name
$ npm init -y
$ npm install webpack webpack-cli --save-dev


# manually run webpack to convert
$ ./node_modules/.bin/webpack <source-file>.js --mode=development
```

You can simplify the webpack command with the `webpack.config.js` file:

```js
module.exports = {
  mode: "development",
  entry: "./index.js",
  output: {
    filename: "main.js",
    publicPath: "dist",
  },
};
```

Now, the webpack command reads this config file, and you can run it with the following format:

```shell
$ ./node_modules/.bin/webpack
```

### babel

The following command installs these packages:

- `@babel/core`: main package
- `@babel/preset-env`: defines which JS features to transpile
- `babel-loader`: lets babel work with webpack

```shell
# install babel
$ npm install @babel/core @babel/preset-env babel-loader --save-dev
```

You also have to configure webpack to use the `babel-loader`:

```js
module.exports = {
  ...
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader",
          options: {
            presets: ["@babel/preset-env"],
          },
        },
      },
    ],
  },
};
```

This configuration tells webpack to find any files ending in `.js` except for those in `node_modules`, and transpile them with `babel-loader` using the `@babel/preset-env` presets. This means we can use newer syntax, like the `import` statement:

```js
import library-name from 'library';
```

### Task runners

A task runner is a tool that automates parts of the build process. This includes:

- minifying code
- optimizing images
- running tests
- automatically refreshing the browser

We can execute task runners with npm directly in the `scripts` object in `package.json`:

```js
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  // production minifies the js files
  "build": "webpack --progress --mode=production",
  // watch runs the webpack command when a .js file changes
  "watch": "webpack --progress --watch"
},
```

The script commands do not need to specify the full path to webpack (`./node_modules/.bin/webpack`) because npm knows the file system location of each node module.

Execute these scripts with `npm run <script-key>`:

```shell
$ npm run build
$ npm run watch
```

### dev server

> THIS DOES NOT WORK

A better alternative to `npm run watch` is the webpack-dev-server. Install it with the following command:

```shell
$ npm install webpack-dev-server --save-dev
```

Then add it to `package.json`:

```js
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "build": "webpack --progress --mode=production",
  "watch": "webpack --progress --watch",
  "serve": "webpack-dev-server --open"
},
```

Run the dev server with the following command to open your JS file on `localhost:8080`:

```shell
$ npm run serve
```