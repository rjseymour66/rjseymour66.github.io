---
title: "Build tools"
weight: 5
description: >
  Toolchains to manage your environments and distribute your code.
---

## Project quickstart

1. Generate the package.json file with npm:
   ```shell
   $ npm init -y
   ```
2. Install webpack and the webpack-cli as development dependencies:
   ```shell
   $ npm install webpack webpack-cli --save-dev
   ```
3. Create the following directory structure with src/ and dist/ directories:
   ```shell
   project-root
   ├── dist
   │   └── index.html
   ├── package.json
   ├── package-lock.json
   ├── src
   │   └── index.js
   └── webpack.config.js
   ```
4. Create a webpack.config.js file with at least the following configuration:
   ```js
   const path = require('path');

   module.exports = {
       entry: './src/index.js',
       output: {
           filename: 'main.js',
           path: path.resolve(__dirname, 'dist'),
           // clean: true,
       },
   };
   ```
5. 

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

# uninstall a package
$ npm uninstall <package-name>

# install a specific version with a tag
$ npm install <package-name>@<tag>

# check the node_modules contents
$ ls node_modules
```

_`devDependencies`_ are added to `package.json` when a package is installed as a development dependency. Examples include babel plugins and presets, test runners, and linters. Those listed under _`dependencies`_ are production dependencies. Development dependencies are included in the application so they are present at runtime. Development `dependencies` are available in both development and production, so any packages that are used in both workflows are listed under `dependencies` only.

## Webpack

Read the [guides](https://webpack.js.org/guides/).

Webpack is a bundler. A bundler takes all of your application dependencies and "bundles" them up into a single static file that is available to your application at runtime. As a general rule, you should try to limit the number of production dependencies that your application relies on.

A bundler takes an entry point and builds a dependency graph of your source files and third-party libraries. This is called **dependency resolution**. This is beneficial because it only bundles modules that are in use.

Node.js uses the `require()` statement to import libraries from the filesystem, but this does not work with Javascript in the browser because there is no file system access. The solution is a module bundler.

A Javascript module bundler has a build step that accesses the file system and makes JS files available to the browser. Essentially, it finds all `require()` statements in `.js` files and replaces them with the contents of the required file (npm tracks the file system location of all dependencies).

Read the [Webpack docs](https://webpack.js.org/) and the [Webpack developer docs](https://webpack.js.org/guides/development/).

### Commands

```shell

# start a project
$ mkdir project-name
$ cd project-name
$ npm init -y
$ npm install webpack webpack-cli --save-dev


# manually run webpack to convert
$ ./node_modules/.bin/webpack <source-file>.js --mode=development

# manually run with config file
$ npx webpack --config webpack.config.js
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
    clean: true,
  },
};
```

> `clean` empties the `dist/` directory when you build the project. This helps make sure that the `dist/` directory contains only files that your project actively uses.

loaders
: Transform and bundle non-JS files such as CSS or images. For example, you might link to images in your HTML, CSS, or JS files. You configure a loader so that Webpack doesn't bundle all these assets in with your minified JS.

plugins
: Webpack can perform more advanced features.

mode
: Dynamically configures operations to production or development modes.

Browser compatiblity
: Build bundles that support new and old browsers.

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
### html-webpack-plugin

[Helpful link](https://rapidevelop.org/webpack/setup-html-webpack-plugin)

We only want to write code in the `src/` directory--we do not want to manually add the `index.html` file in the `dist/` directory.

Webpack's `html-webpack-plugin` automatically builds the HTML file in the `dist/` during a build, which includes a script tag for the minified bundle.
Install the package:

```shell
$ npm install --save-dev html-webpack-plugin
```
Now we can use the module as a plugin in `webpack.config.js`. The module accepts an object that defines the HTML file:

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    ...,
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html',
            title: 'Webpack 5 Video Tutorials',
            filename: 'index.html',
            // inject: 'body',
            inject: 'head',
            // scriptLoading: 'defer', // this is the default
        })
    ]
};
```
In the previous example:
- `template` is the path to the HTML template that you provide.
- `title` is the title of the HTML file
- `filename` is the name of the HTML file output in `dist/`
- `inject` is where you want the script tag. By default, it includes the `defer` option.


The [GitHub repo lists](https://github.com/jantimon/html-webpack-plugin#options) the available options.

#### Caching
[Caching article](https://webpack.js.org/guides/caching/)

To prevent caching, you can put hashes in the filename that Webpack generates.

### Asset management

Notes from [this tutorial](https://webpack.js.org/guides/asset-management/).


#### CSS

The following loaders let you import CSS within a JS module:
```shell
$ npm install --save-dev style-loader css-loader
```
Now, you have to update `webpack.config.js` to use the loaders. The `use` array chains loaders, so files are processed by `style-loader`, and then that processed information is passed to `css-loader`:

```js
module.exports = {
    ...
    module: {
        rules: [
            {
                test: /\.css$/i,
                use: ['style-loader', 'css-loader'],
            },
        ],
    },
};
```

After you import and use CSS in a JS file, a `<style>` tag is added to the HTML in the browser after you run webpack.

#### Images

Add the following rule to perform minimal image processing:

```js
    ...
    module: {
        rules: [
            {
                test: /\.css$/i,
                use: ['style-loader', 'css-loader'],
            },
            {
                test: /\.(png|svg|jpg|jpeg|gif)$/i,
                type: 'asset/resource',
            }
        ],
    },
};
```
These rules process images from your `src/` directory and place the product in the output directory. When you import an image into your JS file or CSS, the `css-loader` replaces the path to your image with the final path in your `output.path` directory (defined in `module.exports`, not shown above).

#### Data

This [tutorial section](https://webpack.js.org/guides/asset-management/#loading-data) explains it well.


This [tutorial section](https://webpack.js.org/guides/asset-management/#customize-parser-of-json-modules) covers toml, yaml, and json5.

### Development

When you are in development mode, you want to have strong source maps and a local server that reloads with every change.

You can set your config file in development mode:

```js
module.exports = {
    mode: 'development',
    ...
};
```

#### Source maps

A source map is tool that maps code in your bundle back to your source files. When Webpack bundles all source code, it makes it difficult to locate errors in your source files. You can use a source map so that the terminal tells you were errors occur in the source files, **not** in the bundled JS file:

```js
module.exports = {
    mode: 'development',
    entry: {
        ...
    },
    devtool: 'inline-source-map',
    plugins: [
        ...
    ],
    output: {
        ...
    },
};
```
Read the [devtool docs](https://webpack.js.org/configuration/devtool/) for more options.

#### Watch 

`Watch` watches your source files and recompiles them when there is a change. You can implement this by adding a script with the `--watch` option to your `package.json` file:

```js
  "scripts": {
    ...
    "watch": "webpack --watch",
    "build": "webpack"
```
#### DevServer

`Watch` is great, but you have to manually reload your browser when you make a change to your source files. `DevServer` provides a lightweight server that automatically refreshes your browser. Read the [`DevServer` docs here](https://webpack.js.org/configuration/dev-server/).

Install it with the following command:

```shell
$ npm install --save-dev webpack-dev-server
```

In `webpack.config.js`, tell the dev server where to serve the files from. Use `src/` because you will keep all your work in this directory:

```js
module.exports = {
    mode: 'development',
    ...
    devServer: {
        static: './src',
    },
    ...
};
```

> If you have more than one entrypoint in your build, add the optimization object to your config file:
> 
> ```js
> module.exports = {
>   entry: {
>       index: './src/index.js',
>       print: './src/print.js'
>   },
>   ...
>   optimization: {
>       runtimeChunk: 'single',
>   },
> };
> ```

Finally, update `package.json` with the script to run the dev server:

```js
{
  ...
  "scripts": {
    "start": "webpack serve --open",
    "build": "webpack"
  },
  ...
  "devDependencies": {
    ...
    "webpack": "^5.89.0",
    "webpack-cli": "^5.1.4",
    "webpack-dev-server": "^4.15.1"
  },
  ...
}
```

### Production

Read the [Webpack tutorial](https://webpack.js.org/guides/production/).

In production builds, you want minified bundles, lightweight source maps, and opitimized assets. Set the mode to `production`:

```js
module.exports = merge(common, {
    mode: 'production',
});
```
One benefit of production mode is that it drops dead code (code that is not used in the final build in `dist/`). To better understand the plugins that make this possible, see the [tree shaking guide](https://webpack.js.org/guides/tree-shaking/).

#### Minification

Webpack ships with the [`TerserPlugin`](https://webpack.js.org/plugins/terser-webpack-plugin/) minification module, but another good option is [`ClosureWebpackPlugin`](https://github.com/webpack-contrib/closure-webpack-plugin).

For CSS minification, see [Minimizing for prodcution](https://webpack.js.org/plugins/mini-css-extract-plugin/#minimizing-for-production).

#### Source maps

Source maps should build quickly. Use the `source-map` instead of the `inline-source-map` that is better for development:

```js
module.exports = merge(common, {
    ...
    devtool: 'source-map',
});
```

### Dev + prod

You can specify only one mode in `webpack.config.js`, which is an issue if you want to run both development and production builds from the same project. You can use `webpack-merge` to define the build processes across multiple configuration files:
- `webpack.common.js`: Settings that both dev and prod share, such as entrypoints and output file locations.
- `webpack.dev.js`: Development-specific settings
- `webpack.prod.js`: Production-specific settings.

> If you use `webpack-merge`, make sure you delete the `webpack.config.js` file from the project because Webpack looks for that file by default.

First, install the tool:

```js
npm install --save-dev webpack-merge
```

Next, split up the files. `webpack-common.js` should look familiar:

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: {
        index: './src/index.js',
        print: './src/print.js'
    },
    plugins: [
        new HtmlWebpackPlugin({
            title: 'Development',
        })
    ],
    output: {
        filename: '[name].bundle.js',
        path: path.resolve(__dirname, 'dist'),
        clean: true,
    },
};
```

Next, create the `webpack.{dev | prod}.js` files. These files require that you import both the `merge` tool and the `webpack-common.js` file:

When you define `module.exports`, use the `merge` tool. It accepts the common configuration and an object that defines the settings for that mode.

`webpack.dev.js`:

```js
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
    mode: 'development',
    devtool: 'inline-source-map',
    devServer: {
        static: './dist',
    },
    optimization: {
        runtimeChunk: 'single',
    },
});
```

`webpack.prod.js`:

```js
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
    mode: 'production',
    devtool: 'source-map',
});
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