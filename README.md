# postcss-pxtoremorem [![NPM version](https://badge.fury.io/js/postcss-pxtoremorem.svg)](http://badge.fury.io/js/postcss-pxtoremorem)

A plugin for [PostCSS](https://github.com/ai/postcss) that generates rem or em units from pixel units.

## Install

```shell
$ npm install postcss-pxtoremorem --save-dev
```

## Usage

Pixels are the easiest unit to use (*opinion*). The only issue with them is that they don't let browsers change the default font size of 16. This script converts every px value to a rem or em from the properties you choose to allow the browser to set the font size.


### Input/Output

*With the default settings, only font related properties are targeted.*

```css
// input
h1 {
    margin: 0 0 20px;
    font-size: 32px;
    line-height: 1.2;
    letter-spacing: 1Px;
}

// output
h1 {
    margin: 0 0 20px;
    font-size: 2rem;
    line-height: 1.2;
    letter-spacing: 0.0625em;
}
```

### Example

```js
var fs = require('fs');
var postcss = require('postcss');
var pxtoremorem = require('postcss-pxtoremorem');
var css = fs.readFileSync('main.css', 'utf8');
var options = {
    replace: false
};
var processedCss = postcss(pxtoremorem(options)).process(css).css;

fs.writeFile('main-rem.css', processedCss, function (err) {
  if (err) {
    throw err;
  }
  console.log('Rem file written.');
});
```

### options

Type: `Object | Null`  
Default:
```js
{
    rootValue: 16,
    unitPrecision: 5,
    propList: ['font', 'font-size', 'line-height', 'letter-spacing'],
    selectorBlackList: [],
    replace: true,
    mediaQuery: false,
    minPixelValue: 0,
    exclude: /node_modules/i
}
```

- `rootValue` (Number | Function) Represents the root element font size or returns the root element font size based on the [`input`](https://api.postcss.org/Input.html) parameter
- `unitPrecision` (Number) The decimal numbers to allow the REM or EM units to grow to.
- `propList` (Array) The properties that can change from px to rem or em.
    - Values need to be exact matches.
    - Use wildcard `*` to enable all properties. Example: `['*']`
    - Use `*` at the start or end of a word. (`['*position*']` will match `background-position-y`)
    - Use `!` to not match a property. Example: `['*', '!letter-spacing']`
    - Combine the "not" prefix with the other prefixes. Example: `['*', '!font*']` 
- `selectorBlackList` (Array) The selectors to ignore and leave as px.
    - If value is string, it checks to see if selector contains the string.
        - `['body']` will match `.body-class`
    - If value is regexp, it checks to see if the selector matches the regexp.
        - `[/^body$/]` will match `body` but not `.body`
- `replace` (Boolean) replaces rules containing rems or ems instead of adding fallbacks.
- `mediaQuery` (Boolean) Allow px to be converted in media queries.
- `minPixelValue` (Number) Set the minimum pixel value to replace.
- `exclude` (String, Regexp, Function) The file path to ignore and leave as px.
    - If value is string, it checks to see if file path contains the string.
        - `'exclude'` will match `\project\postcss-pxtoremorem\exclude\path`
    - If value is regexp, it checks to see if file path matches the regexp.
        - `/exclude/i` will match `\project\postcss-pxtoremorem\exclude\path`
    - If value is function, you can use exclude function to return a true and the file will be ignored.
        - the callback will pass the file path as  a parameter, it should returns a Boolean result.
        - `function (file) { return file.indexOf('exclude') !== -1; }`

### Use with gulp-postcss and autoprefixer

```js
var gulp = require('gulp');
var postcss = require('gulp-postcss');
var autoprefixer = require('autoprefixer');
var pxtoremorem = require('postcss-pxtoremorem');

gulp.task('css', function () {

    var processors = [
        autoprefixer({
            browsers: 'last 1 version'
        }),
        pxtoremorem({
            replace: false
        })
    ];

    return gulp.src(['build/css/**/*.css'])
        .pipe(postcss(processors))
        .pipe(gulp.dest('build/css'));
});
```

### A message about ignoring properties
Currently, the easiest way to have a single property ignored is to use a capital in the pixel unit declaration.

```css
// `px` is converted to `rem`
.convert {
    font-size: 16px; // converted to 1rem
}

// `Px` is converted to `em`
.convert {
    font-size: 16Px; // converted to 1em
}

// `PX` is ignored by `postcss-pxtoremorem` but still accepted by browsers
.ignore {
    border: 1PX solid; // ignored
    border-width: 2PX; // ignored
}
```
