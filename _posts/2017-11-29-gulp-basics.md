---
layout: post
title:  "Gulp basics"
categories: gulp angularjs
---


  
## Project Setup

Create a working folder and initialize it for npm modules.

```
mkdir test && cd test
yarn init -y  // (1)
```
  
**(1)** Use the -y flag if you don't want to answer the yarn initialization questions one by one, this will quietly create the package.json file with all the defaults in place

To use gulp, you need to install it globally and on the project folder where you want to use it.
```
yarn add -g gulp          // (1)
yarn add gulp --save-dev  // (2)
```
(1) By using the **-g** flag, this installs gulp globally
(2) This installs gulp locally, on the project folder. We'd like to save it on package.json as a dev dependency, hence the **--save-dev** flag

Next, create a gulp file named `gulpfile.js` in the working folder. If you execute `gulp` in the local folder without passing any parameter to it, it will look up for `gulpfile.js` by default. 

```javascript
const gulp = require('gulp');
gulp.task('default', () => {
  console.log('Ready to go');
});
```







