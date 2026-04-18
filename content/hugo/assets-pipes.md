+++
title = 'Assets and Pipes'
date = '2025-07-13T09:32:43-04:00'
weight = 90
draft = false
+++



The extended version of Hugo has pipes that let you transform your assets from within Hugo. For example:
- Minify JS and CSS makes your files transfer faster, which improves speed
- Fingerprint to each file, so when you deploy a modified version, the browser invalidates the cached version and fetches the new one.
- Image processing, including resizing images
- Hugo doesn't support advanced features like JS modules, so it integrates with Webpack.

### Directory structure

Pipelines require that you move your assets from the `static/` directory to `assets/`:

```bash
mkdir themes/docsite/assets
mv themes/docsite/static/css themes/docsite/assets
```

### Stylesheets

We're going to migrate to Sass, and minify and fingerprint the CSS assets:
- Sass lets has extended features like functions and lets you break your stylesheets into multiple files
- Minifying removes whitespace, non-printable characters, and comments so your files transfer faster.
- Fingerprinting creates a unique filename for your CSS files each time you modify it. When you serve content files, the browser caches it. A unique filename helps the browser notice that the file changed.

To use Sass, change the name of your CSS file:

```bash
mv themes/docsite/assets/css/style.css themes/docsite/assets/css/style.scss
```
With Sass, you can use partial files. For example, extract all navbar settings to `themes/docsite/assets/css/_navbar.scss`, and then import them into the `.../style.scss` stylesheet with the `@import "navbar";` statement at the top of the file.


To process the CSS assets, we store the CSS content location in a variable. Here, that is `$css`. `resources.Get` takes one argument: the path to the stylesheet within the `themes/docsite/assets` directory. The full path to this stylesheet is `themes/docsite/assets/css/style.css`:

```html
<!-- themes/docsite/layouts/_partials/head.html -->
{{ $css := resources.Get "css/style.scss" | toCSS | minify | fingerprint }}
```
Storing your CSS in a variable lets you apply transformations to the file:
- `toCSS` transforms your `.scss` file to `.css`
- `minify` minifies the files for faster file transfers
- `fingerprint` adds a unique ID to your files for caching

After you create the variable, use it in the `href` attribute for the `<link>`. This templating creates a relative link to the CSS file when it builds the `public/css` files:

```html
{{ $css := resources.Get "css/style.css" }}
<link rel="stylesheet" href="{{ $css.RelPermalink }}">
```

These lines create these files in `public/`:

```bash
project-root/
│
... 
├── public
...
│   ├── css
│   │   └── style.min.6df00e795ff8de8f8f9b48a3b95d2f845a69f643faab7bc711f49736090b8738.css
│   ...
├── resources
│   └── _gen
│       └── assets
│           └── css
│               ├── style.scss_77b10c8e87ff110a62c52933fe3f7f11.content
│               └── style.scss_77b10c8e87ff110a62c52933fe3f7f11.json
...

```


#### Managing images

Theme and site static directories are merged together. You don't have to specify static in the path

This image is in `static/images/hugo-logo-wide.svg`.

```html
<!-- themes/docsite/layouts/_partials/footer.html -->
<p>Powered by <a href="https://gohugo.io">
        <img src="{{ "images/hugo-logo-wide.svg" | relURL }}" alt="image" width="128" height="38">
    </a>
</p>
```

You can include this in your markdown files like this:

```md
[This is an image](/hugo-logo-wide.svg)
```

### Page bundles

A page bundle is a content directory that includes all resources for a page. To create a page bundle, include the page bundle directory in the command. Here, `my-vacation` is the page bundle directory:

```bash
hugo new posts/my-vacation/index.md
project-root
│
...
├── content
│   ├── posts
│   │   ├── my-vacation                 # page bundle
│   │   │   ├── images
│   │   │   │   ├── badlands.jpg
│   │   │   │   ├── bison.jpg
│   │   │   │   └── rushmore.jpg
│   │   │   └── index.md
...
```

### Shortcodes

A shortcode is a template that outputs HTML. Store them in the `layouts/shortcodes/` directory:

```bash
mkdir themes/docsite/layouts/shortcodes
```

Here is a shortcode that shows how you can access resources in a page bundle:
1. `(.Get 0)` retrieves the first argument passed to the shortcode. Here, its a path.
   `$.Page.Resources.GetMatch` finds the page resource that matches the first argument path.
   `$.` calls the page context. Inside a shortcode, the context is the shortcode itself.
2. `$image.Resize "1024x"` resizes the image to 1024 pixels with the [Resize](https://gohugo.io/methods/resource/resize/) method. This uses `<width>x<height>` format. Specify width OR height so the image scales proportionately.
3. `.Get 1` retrieves the second argument to the shortcode. Here, its the figcaption text.

```html
(1) {{ $image := $.Page.Resources.GetMatch (.Get 0) }}
(2) {{ $smallImage := $image.Resize "1024x" }}

    <figure class="post-figure">
        <a href="{{ $image.RelPermalink }}">
            {{ with $smallImage }}
            <img 
                src="{{ .RelPermalink }}" 
                alt="{{ $.Get 1 }}"
                width="{{ .Width }}"
                height="{{ .Height }}">
            {{ end }}
        </a>
(3)     <figcaption>{{ .Get 1 }}</figcaption>
    </figure>
```

Here is how you use it in the content file:

```md
\{\{< postimage "images/rushmore.jpg" "Mount Rushmore" >}}
\{\{< postimage "images/badlands.jpg" "The Badlands" >}}
\{\{< postimage "images/bison.jpg" "Bison at Custer National Park" >}}
```

### Image processing

Processing images with Hugo can dramatically increase the build time. To speed things up, Hugo caches the converted images and saves them in the `resources/_gen/images/<path>` directory so it doesn't have to build them each time. 

If you change these image files, Hugo doesn't automatically clean the `resources/` directory, so you have to explicitly clear the cache on build:

```bash
hugo --gc
```

### Javascript

Put all your JS libraries in minified, fingerprinted file so you don't have to make network requests.

To use your JS files in a pipeline, move them from the `static/` directory to `assets/`:

```bash
mv themes/docsite/static/js/ themes/docsite/assets/
```

Make sure you have the full, unminified JS libraries. Here, we download the Lunr search library ino the `assets/` directory:

```bash
curl https://unpkg.com/lunr@2.3.6/lunr.js > themes/docsite/assets/js/lunr.js
```

Create your asset pipes where you import the library with script tags, in `themes/docsite/layouts/search.html`:
1. Get the `lunr.js` file resource object and store it in `$lunr`.
2. Get the `search.js` file resource object and store it in `$search`.
3. Combine the `$lunr` and `$search` resource objects into a slice named `$libs`.
4. `resources.Concat "js/app.js"` concatenates all files in `$libs` into a single file named `app.js`, then minifies and fingerprints the file.

```html
<!-- themes/docsite/layouts/search.html -->
...
(1) {{ $lunr := resources.Get "js/lunr.js" }}
(2) {{ $search := resources.Get "js/search.js" }}
 
(3) {{ $libs := slice $lunr $search }}
(4) {{ $js := $libs | resources.Concat "js/app.js" | minify | fingerprint }}
...
```
Here is the file after you build the project:

```bash
public/
│
...
├── js
│   └── app.min.4e058...20eaf.js
...
```

### Webpack

Hugo can minify, concatenate, and fingerprint files, but it can't do more complicated tasks like handle JS modules and transpilation.

First, create a `package.json` file. You can do this manually or with `npm init -y`:

```json
{
  "name": "docsite",
  "version": "1.0.0",
  "description": "Docsite theme",
  "private": true,                          // don't publish code to package repo
  "scripts": {
    "build": "hugo --cleanDestinationDir",
    "hugo-server": "hugo server --disableFastRender"    // nothing gets cached - hugo rerenders all pages on every change so its good if you're editing templates, styles, config files because all pages are updated
  },
  "author": ""
}
```

Build the site with the `build` command:

```bash
npm run build
```
To add webpack, you have to install it as a development dependency:

```bash
npm install --save-dev webpack webpack-cli
```

Install packages for the JS libraries that you want to include in the site. This package replaces any JS files that you have in `themes/docsite/assets/js`. These are production dependencies, so use `--save` instead of `--save-dev`:

```bash
npm install --save lunr
```
Webpack reads `webpack.config.js` for instructions on how to build your project. We are going to create a config file that looks for a file named `index.js`, resolves its dependencies (e.g. imports libraries), then outputs an asset file named `app.js`.

Create this file in the root of your site:

```js
(1) const path = require('path');

(2) module.exports = {
(3)     entry: './themes/docsite/assets/js/index.js',
(4)     output: {
(5)         filename: 'app.js',
(6)         path: path.resolve(__dirname, 'themes', 'docsite', 'assets', 'js')
        }
    };
```
1. Import the Node.js `path` module so you can join filesystem paths
2. Exports an object that Webpack uses as its configuration
3. Defines the entry point for your app. When building a dependency graph, Webpack begins with this file.
4. File name for the final bundled JS file
5. Path where the final bundles `app.js` file is written. `__dirname` is a special variable in Node.js that resolves to the current directory.

Because this is still in the `../assets/` folder, you can still use Hugo's minifier and fingerprinter. If you want to use Webpack for these pipeline stages, you should place the output in the `themes/../static/` directory and configure Webpack accordingly:

```js
...
module.exports = {
...
        path: path.resolve(__dirname, 'themes', 'docsite', 'static', 'js')
    }
};
```

Place any JS code you have in the `/themes/docsite/assets/js/index.js` file. Here, we had a file named `search.js` that we are converting into the entrypoint:

```bash
mv themes/docsite/assets/js/search.js themes/docsite/assets/js/index.js
```

Without webpack, we downloaded the JS library files and linked them in the `search.html` layout file. Because we downloaded the external libraries with Webpack, we can import them in our `index.js` instead:

```js
'use strict';

import lunr from 'lunr';
...
```

Webpack is downloading our JS dependencies, so we no longer need to concatenate the JS files as resource objects with `resources.Get` in `themes/docsite/layouts/search.html`. Instead, we retrieve the `app.js` resource that Webpack outputs. Because it is in the `assets/` directory, we can still use Hugo's `minify` and `fingerprint` functions:

```html
...
{{ $js := resources.Get "js/app.js" | minify | fingerprint }}
<script src="{{ $js.RelPermalink }}"></script>
```

In `package.json`, create a script that runs Webpack to build the `app.js` file before you run the Hugo server. This is necessary because Hugo needs to find this file in the `assets/` directory to build.

Webpack is installed as a project dependency, not as a global package, so you have to run Webpack directly from the binary in `node_modules/`. However, it is a project dependency, so you can run it with `package.json` without specifying the binary path in `node_modules`:

```js
{
  ...
  "scripts": {
    ...
    "webpack": "webpack"
  },
  ...
}
```

Webpack has a development server that watches for changes with the `webpack --watch` command, but that requires that you open two terminal servers to run Webpack and Hugo in parallel. Instead, you can install the `npm-run-all` command to run multiple tasks simultaneously:

```bash
npm install --save-dev npm-run-all
```

Next, add two scripts to `package.json`:

```js
{
  ...
  "scripts": {
    ...
    "webpack-watch": "webpack --watch",
    "dev": "npm-run-all webpack --parallel webpack-watch hugo-server"
  },
  ...
}
```

The `dev` script references the `webpack-watch` command that runs the Webpack dev server.

Finally, create a build script that runs Webpack and then Hugo. Rename the existing `build` script to `hugo-build`, and reference it in the new `build` command:

```js
{
  ...
  "scripts": {s
    "build": "npm-run-all webpack hugo-build",
    "hugo-build": "hugo --cleanDestinationDir",
    ...
  },
  ...
}
```