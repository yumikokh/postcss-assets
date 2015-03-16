PostCSS Assets [![Build Status](https://travis-ci.org/borodean/postcss-assets.svg?branch=develop)](https://travis-ci.org/borodean/postcss-assets)
==============

PostCSS Assets is an asset manager for CSS which uses the [PostCSS postprocessor framework](https://github.com/postcss/postcss). It helps to manage assets, inline their contents and print image sizes directly inside stylesheets.

Functions
---------

PostCSS Assets extends CSS by augmenting helper functions. For example, you can write:

```css
.logo {
  overflow: hidden;
  width: width('background.png');
  padding-top: height('background.png');
  background: inline('background.png') no-repeat center;
}
```

After being processed by PostCSS Assets, the above would become a regular CSS browser understands:

```css
.logo {
  overflow: hidden;
  width: 192px;
  padding-top: 48px;
  background: url('data:image/png;base64,...') no-repeat center;
}
```

### `resolve(asset-path)`
Generates URL for an asset.

```css
body {
  background-image: resolve('page/background.jpg');
}
```

```css
body {
  background-image: url('/assets/images/page/background.jpg');
}
```

### `inline(asset-path)`
Embeds a base64-encoded content of an asset directly inside a stylesheet, eliminating the need for another HTTP request. For small assets, this can be a performance benefit at the cost of a larger generated CSS file.

SVG files would be inlined unencoded, because then [they benefit in size](http://css-tricks.com/probably-dont-base64-svg/).

```css
.icon-sabre {
  background: inline('icons/sabre.png') no-repeat center;
}
```

```css
.icon-sabre {
  background: url('data:image/png;base64,...') no-repeat center;
}
```

### `width(asset-path, [density])`
Prints the width of an asset.

```css
.logo {
  width: width('logo.png');
  height: height('logo.png');
  background-size: size('logo.png');
}
```

```css
.logo {
  width: 132px;
  height: 48px;
  background-size: 132px 48px;
}
```

### `height(asset-path, [density])`
Prints the height of an asset.

### `size(asset-path, [density])`
Prints both width and height of an asset.

Options
-------

Options are set by passing an options object to TODO. Available options are:

### `basePath`
The path to the root of the project.

For example: `"source/"`.

Defaults to the current working directory.

### `baseUrl`
URL of the project when running withing the web server.

For example: `"/wp-content/themes/twentyfourteen"`, `"http://example.com"`.

Defaults to `"/"`.

### `cachebuster`
If cache should be busted. Pass a function to define custom busting strategy.

Defaults to `false`.

### `loadPaths`
Specific directories to look for the files.

For example: `["assets/fonts", "assets/images"]`.

Defaults to an empty array.

### `relativeTo`
Directory to relate to when resolving URLs. When `false`, disables relative URLs.

For example: `"assets/css"`.

Defaults to `false`.

Path resolution
---------------

PostCSS Assets provides a file path resolution algorythm similar to the one used by desktop file systems.

This may come in handy when you have different directories for different types of assets, e.g. images, fonts. You add those to the list of load paths when configuring Assets:

```js
assets({
  loadPaths: ['assets/fonts', 'assets/images']
});
```

Now, instead of writing this:

```css
.icon-create { background-image: url('/assets/images/icons/create.png') }
.icon-read   { background-image: url('/assets/images/icons/read.png') }
.icon-update { background-image: url('/assets/images/icons/update.png') }
.icon-delete { background-image: url('/assets/images/icons/delete.png') }
```

You can write this:

```css
.icon-create { background-image: resolve('icons/create.png') }
.icon-read   { background-image: resolve('icons/read.png') }
.icon-update { background-image: resolve('icons/update.png') }
.icon-delete { background-image: resolve('icons/delete.png') }
```

Apart from the fact that these lines are just shorter, it gives you an opportunity to easily change the environment and the way the urls are being output much quickier.

For instance, if you move all the images from `assets/images` to `client/source/images` you wouldn't need to go through all of your stylesheets to replace the urls, you would just need to edit the corresponding parameter inside your Assets config:

```js
assets({
  loadPaths: ['assets/fonts', 'client/source/images']
});
```

If you divide your stylesheet into different bits and place it's assets in the same folder, you may not bother about their real paths anymore:

```css
/* assets/icon/icon.css */
.icon-create { background-image: resolve('create.png') }
```

Will output to:

```css
.icon-create { background-image: url('/assets/icon/create.png') }
```

When resolving a path, Assets would look for it throught every of the following paths in the listed order:

* source file path;
* load paths;
* base path.

Path resolution also gives an opportunity to easily change the urls structure when directory structure of the project on your computer is not exactly the same as it would be on the server.

For instance, if you have a Wordpress theme project, you may want to append `/wp-content/themes/your-theme-name` to every URL inside of your stylesheet. This is done by providing a `baseUrl` parameter to Assets config:

```js
assets({
  baseUrl: '/wp-content/themes/your-theme-name'
});
```

```css
.icon-create {
  background-image: resolve('images/create.png');
}
```

Now everything would be resolved relative to that base URL:

```css
.icon-create {
  background-image: resolve('/wp-content/themes/your-theme-name/images/create.png');
}
```

Installation
------------

### Gulp

You can use the `gulp-postcss` plugin for Gulp with PostCSS Assets and other PostCSS plugins.

```js
gulp.task('assets', function () {
  var postcss = require('gulp-postcss');
  var assets  = require('postcss-assets');

  return gulp.src('source/*.css')
    .pipe(postcss([
      assets({
        cachebuster: true,
        loadPaths: ['assets/fonts', 'assets/images']
      })
    ]))
    .pipe(gulp.dest('build/'));
});
```

### Grunt

You can use the `grunt-postcss` plugin for Grunt with PostCSS Assets and other PostCSS plugins.

```js
var assets  = require('postcss-assets');

grunt.initConfig({
  postcss: {
    options: {
      processors: [
        assets({
          cachebuster: true,
          loadPaths: ['assets/fonts', 'assets/images']
        })
      ]
    },
    dist: { src: 'build/*.css' }
  },
});
```

### JavaScript

You may use PostCSS Assets directly inside your Node.js application:

```js
var assets = require('postcss-assets');
var result = assets.process('.icon { background: inline("icon.svg") }').css;
```

PostCSS Assets can also be used as a [PostCSS](https://github.com/postcss/postcss) processor, so you can cobine it with [other processors](https://github.com/postcss/postcss#built-with-postcss) and parse CSS only once:

```js
var assets = require('postcss-assets');
var autoprefixer = require('autoprefixer-core');

postcss()
  .use(assets({
    cachebuster: true,
    loadPaths: ['assets/fonts', 'assets/images']
  }))
  .use(autoprefixer)
  .process(css);
```
