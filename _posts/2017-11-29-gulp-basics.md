---
layout: post
title:  "Gulp basics"
categories: angularjs tools gulp
---

Description of the whole thing. What are the key points to remember

* It's a JS file
* Does it use nodeJS?
* It's a stream system, unlike grunt
* Plugin system
* How is it compared to webpack, should I compare? No. I don't care about that. That is a question for another day
* Explain pipe
* THe globbing function
* gulp.src and gulp.dest


## The Project files

The following listings shows the structure and content our sample project

> **TIP :** if you want to follow along this exercise, you can get the project source files, plus the completed gulpfile from [github.com/angularjstools][angularjstools]. Downloading the project source files will save you from a lot of typing

_Example 1. Project Structure_
```
├── index.html
├── js
│   ├── 1.js
│   ├── controllers.js
│   ├── directives.js
│   ├── filters.js
│   └── services.js
```

_index.html_
```html
<html ng-app="app">
<body ng-controller="myController">

  <dir-one></dir-one>
  <dir-two></dir-two>
  <p> {{ "James"  | greet }}</p>

  <script src="node_modules/angular/angular.js"></script>
  <script src="node_modules/angular-route/angular-route.js"></script>

  <script src="js/1.js"></script>
  <script src="js/controllers.js"></script>
  <script src="js/directives.js"></script>
  <script src="js/filters.js"></script>
  <script src="js/services.js"></script>

</body>
</html>
```

_1.js_
```javascript
var mymod = angular.module('app', []);
```

_controllers.js_
```javascript
mymod.controller('myController', function($scope, myService){
  console.log('myController');
});
```

_directives.js_
```javascript
mymod.directive('dirOne', function(){
  console.log("dirOne");
  return {
    template: "<h2>Dir One</h1>"
  }
});

mymod.directive('dirTwo', function(){
  console.log("dirTwo");
  return {
    template: "<h2>Dir Two</h1>",
    controller: function($scope, myService) {
      console.log("controller - Dir Two");
    }
  }
});
```

_filters.js_
```javascript
mymod.filter('greet', function() {
  return function(name) {
    return "Hei "  + name;
  }
});
```

_services.js_
```javascript
mymod.factory('myService', function($http) {
  console.log("myService");
  return {
    foo : function(){
      return "Hello Foo";
    }
  }
});
```

## Testing and Running the project

Get the http-server node package from the npm repos, if you don't have it yet. We will use this to run a local web server for our development. On a terminal window, run the following command

```
yarn add http-server -g
```

When the command completes, run the http server in the current directory in order to serve its contents at "http://localhost:8080"

```
http-server -c-1 .
```

> **NOTE :** The flag `-c-1`  tells the http-server to get rid of the cache every 1 second. This way, we won't run into cache problems during development

Open a browser and navigate to `http://localhost:8080`

![]({{"/images/gulp-indexhtml.png" | absolute_url}})
_Fig 1. index.html_

Figure 1 (above) shows the app in Chrome browser with the Developer tools open. 

## Project Setup

Create a working folder and initialize it for npm modules.

```
mkdir test && cd test
yarn init -y  // (1)
```
  
(1) Use the -y flag if you don't want to answer the yarn initialization questions one by one, this will quietly create the package.json file with all the defaults in place

> **NOTE :** You don't have to use yarn, you can simply use npm instead of it. But if you want to try it out, you can find information about it at [yarnpkg.com](https://yarnpkg.com)

To use gulp, you need to install it globally and on the project folder where you want to use it.

```
yarn add -g gulp-cli      // (1)
yarn add gulp --save-dev  // (2)
```

(1) By using the **-g** flag, this installs gulp globally

(2) This installs gulp locally, on the project folder. We'd like to save it on package.json as a dev dependency, hence the **--save-dev** flag

Next, create a gulp file named "gulpfile.js" in the working folder. If you execute `gulp` in the local folder without passing any parameter to it, it will look up for "gulpfile.js" by default, while it is possible to use another name, following the convention is much easier and frictionless because if you name it something else, then you will have to specify the name of gulp file using a command line flag. So, just name it "gulpfile.js" 

Listing 1 shows a minimally written gulp file. You may immediately notice that it's written in Javascript, so web developers will be right at home immediately.

_Listing 1. boilerplate code_
```javascript
let gulp = require('gulp');          // (1)

gulp.task('default', function() {    // (2)
  console.log('Ready to go');        // (3)
});
```

(1) Gulp uses the commonJS style of loading the library, like NodeJS. Let's get a reference to the gulp module and store it in the `gulp` variable

(2) A task is defined by calling the method `gulp.task`, first param is the name of the task, and second param is a function. The task naming is up to you, you can choose whatever name you please, but this task called "default" is special. When you execute gulp without any parameter, or without specifying any task, it will look for a task named "default" and it will execute that

(3) When the task is called, the statements in function body is executed

Run gulp from the command line.

```
gulp
```

## Bundling

Gulp can be used to bundle a bunch of files and combine them all into one file. Consider the project structure as shown in Example 1 (below)

_Example 1. Simple web project structure_
```
├── index.html
├── js
│   ├── 1.js
│   ├── controllers.js
│   ├── directives.js
│   ├── filters.js
│   └── services.js
```

The "js" folder contains multiple files, each one must be included in the HTML file. We can combine all these files into one and simpy reference a single file. In the HTML file, what we want to do is reference the combined JS file as shown in Listing 2 (below)

_Listing 2.index.html_
```html
<html ng-app="app">
<body ng-controller="myController">

  <script src="node_modules/angular/angular.js"></script>
  <script src="node_modules/angular-route/angular-route.js"></script>

  <script src="dist/js/bundle.js"></script>     // (1)
</body>
</html>
```

(1) Instead of 5 &lt;script&gt; tags, we only want to reference one. We will combine the 5 Javascript files into "bundle.js" The **dist/js** directory does not exist yet, we will also create this in the gulp file.

To combine multiple files into one, we will use the "gulp-concat" plugin. Let's pull it from the repo and add it to the project dev dependencies.

```
yarn add gulp-concat --save-dev
```

_Listing 3.gulp-concat_
```javascript
let gulp = require('gulp');
let concat = require('gulp-concat'); // (1)        

gulp.task('default', function() {    
  return gulp.src('js/**/*.js')      // (2)
      .pipe(concat('bundle.js'))     // (3)
      .pipe(gulp.dest('dist/js'))    // (4)
});
```

(1) Let's get a reference to concat by requiring it

(2) Let's glob every file in the "js" directory. This js directory is being referenced relative to the location of "gulpfile.js". If you have a different directory structure, you need to adjust this globbing expression. This line returns a stream which can now be piped into whatever plugin

(3) This line takes the stream returned from (2) and then pipes it to the gulp-concat plugin, `concat` combines everything and saves it into "bundle.js", but this isn't a file yet, it is still a stream

(4) This takes stream input from (3) and saves it into a file. The specified file path in the `gulp.dest` method will be subsequently created.

To execute the default task, run `gulp` again from the command line.

_Example 3.Final structure of the project_
```
├── dist                  // (1)
│   └── js
│       └── bundle.js
├── gulpfile.js       
├── index.html
├── js                    // (2)
│   ├── 1.js
│   ├── controllers.js
│   ├── directives.js
│   ├── filters.js
│   └── services.js
```

(1) Created by `gulp.dest`

(2) The original js folder

### Order of files



## Watcher

Each time you make a change to the source files in the "js" folder, you need to run `gulp` from the command line so can see the effect of the changes. This task can be automated by setting up a watcher in gulp. In this way, whenever there are changes on the file system, gulp will automatically run a task. 

Listing 4 (below) shows a refactored gulp file. All the statements in our previous default task is now under another task named "script" and the "default" task contains the watch statement.

_Listing 4. Refactored gulpfile.js_
```javascript
let gulp = require('gulp');
let concat = require('gulp-concat');
let watch = require('gulp-watch');           // (1) 

gulp.task('default', ['script'] , function(){  // (2)
  gulp.watch('js/**/*.js', ['script']);      // (3)
});

gulp.task('script',function(){
  console.log("Ready to go");
  return gulp.src('js/**/*.js')
   .pipe(concat('bundle.js'))
   .pipe(gulp.dest('dist/js'));
});
``` 

(1) The watcher is gulp plugin, so you need to get gulp-watch from the repo using the command `yarn add gulp-watch --save-dev`

(2) This task takes on 3 parameters, the first is the name of the task (default), third is the usual function which we want to execute when the task is called and the second one is an array of task. The tasks specified in this array needs to be executed first before we can execute the function in the default task. This is how gulp specifies dependencies. If a task has a dependency on other task, it will be specified in this second parameter.
 
(3) The watch function takes on two parameters. The first one is a glob expression which specifies what directory or what files to watch, in our case, we'd like to watch for any changes in the "js" directory. The second parameter is an array which contains the tasks you want to run whenever a change is detected, in our case, we want to run the "script" task again

When you run `gulp` this time, it won't go back to the command prompt because  it is running a process (foreground process) that watches for any change in the file system. If you want to terminate the process, press `CTRL+C`

## Minification

After bundling all your source files into one, you might want to consider minifying the source. Minification is a common practice for web applications because it reduces the size of the file that users will download when they want to view your webapp. To minify our bundled source file, we can use the  "gulp-uglify" plugin.

_Listing 5. Uglify_
```javascript
let gulp = require('gulp');
let concat = require('gulp-concat');
let watch = require('gulp-watch');           

gulp.task('default', ['script'] , function(){  
  gulp.watch('js/**/*.js', ['script']);      
});

gulp.task('script',function(){
  console.log("Ready to go");
  return gulp.src('js/**/*.js')
   .pipe(concat('bundle.js'))
   .pipe(uglify())
   .pipe(gulp.dest('dist/js'));
});
```

You need to stop the gulp process and restart it again so that our changes in the gulp file can take effect.

## New problems  

The minification process will introduce some problems. As you can see in Fig 2, the console is showing some errors.

![]({{"/images/gulp-indexhtml-error-minify.png" | absolute_url}})
_Fig 2.Minification problem_

From the obscure error message shown in Fig 2, it appears that the error is rooted somewhere in our controller codes, but remember that we are no longer referencing "controller.js", all of our codes have been bundled and minified by gulp and this error is being thrown from "dist/js/bundle/js". Let's examine bundle.js, specifically the area of the controller code. A snippet of bundle.js is shown in Listing  6 (below)

_Listing 6. Snippet of bundle.js_
```javascript
mymod.controller("myController",function(o,r){console.log("myController")})
```

The original code in our controller.js is shown in Listing 7 (below)

_Listing 7.controller.js_
```javascript
mymod.controller('myController', function($scope, myService){
  console.log('myController');
});
```

It appears that the minification process mangled our original code, instead of passing the arguments `$scope` and `myService` to the controller function, it replaced these arguments with the letters "o" and and "r" respectively. But this is expected behavior, this is really one of the side effects of minification. How do we solve it? There are two ways. One of them involves writing our controllers and services a bit differently than what we're used to, see listing 8 (below)

_Listing 8.How to write a controller for minification_
```javascript
mymod.controller('myController', ["$scope", "myService", function($scope, myService){
  console.log('myController');
}]);
```
The second parameter to the controller function is now an array in which the last element is function containing the definition of the controller and the previous elements are the things we need to inject to the controller function. This is a bit unnatural and bit harder to remember. 

Fortunately, we can solve this issue with tooling. The ng-gulp-annotate plugin can do this injection for us automatically, this way, we can still write AngularJS codes in the way we're used to and still minify it. 

## ngAnnotate

To use ngAnnotate, pull it from the repo and save it as a dev dependency. 

```
yarn add gulp-ng-annotate --save-dev
```

_Listing 9. ngAnnotate_
```javascript
let gulp = require('gulp');
let concat = require('gulp-concat');
let watch = require('gulp-watch');           
let ngAnnotate = require('gulp-ng-annotate');

gulp.task('default', ['script'] , function(){  
  gulp.watch('js/**/*.js', ['script']);      
});

gulp.task('script',function(){
  console.log("Ready to go");
  return gulp.src('js/**/*.js')
   .pipe(ngAnnotate())              // (1)
   .pipe(concat('bundle.js'))
   .pipe(uglify())
   .pipe(gulp.dest('dist/js'));
});
```

(1) We'll make ngAnnotate the first one in the pipeline, we'd like the codes annotated before it they get bundled and minified 

Restart the gulp process to reflect our changes.

## Sourcemaps and Browser Debugging

## Obfuscation



## Cleaning

## Code Hinting

## Targeting Environments


[angularjstools]: https://github.com/tedhagos/angularjstools


