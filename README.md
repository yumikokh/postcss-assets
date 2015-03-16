PostCSS Assets [![Build Status](https://travis-ci.org/borodean/postcss-assets.svg?branch=develop)](https://travis-ci.org/borodean/postcss-assets)
==============

PostCSS Assets is an asset manager for CSS which uses the [PostCSS postprocessor framework](https://github.com/postcss/postcss). It helps to manage assets, inline their contents and print image sizes directly inside stylesheets.

Functions
---------

### `resolve(asset-path)`
Generates a path to an asset.

### `inline(asset-path)`
Embeds the contents of an image directly inside your stylesheet, eliminating the need for another HTTP request. For small images, this can be a performance benefit at the cost of a larger generated CSS file.

### `width(asset-path, [density])`
Returns the width of the image found at the path supplied by $image relative to your project's images directory.

### `height(asset-path, [density])`
Returns the height of the image found at the path supplied by $image relative to your project's images directory.

### `size(asset-path, [density])`
Returns the size of the image found at the path supplied by $image relative to your project's images directory.

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

Installation
------------

### Gulp

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

```js
var assets = require('postcss-assets');
var result = assets.process('.icon { background: inline("icon.svg") }').css;
```

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
