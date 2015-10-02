---
layout: post
title: Increase Your Productivity with Task Runners
comments: true
---
During software development, there is often many commands and scripts you want to run before you git commit and push it to github or even deploy. You may want to run tests for your entire stack, and you may also want to run your style checker to make sure you have not violated the style guide. Without task runners, you would probably have to do something like this:
{% highlight bash %}
$ jshint files/to/check
$ mocha -R spec path/to/spec
$ npm test
$ and more...
$ and more...
{% endhighlight %}
While there is nothing wrong with doing this, we can increase our productivity by using something called task runners to run all those commands for us automatically. Two of the most poplular tasks runners for web development are currently Grunt and Gulp, and they are pretty easy to setup considering how much time you could potentially save down the road. For this example, I will demonstrate how you can get gulp setup. Before we dive in, make sure you have `npm` installed.

To begin, run
{% highlight bash %}
$ npm install -g gulp
{% endhighlight %}
It doesn't matter what your current directory is. The `-g` flag tells npm to install gulp for you *globally* . Then, go to the root of your project repo and run 
{% highlight bash %}
$ npm install --save-dev gulp
{% endhighlight %}
This will add gulp and all it's dependencies to your `node_module` so that you can import it in your gulpfile. Then, you would want to create a `gulpfile.js` at the root of your project repo. Note that you can't name this file anything other than that because when you run `gulp`, it looks specifically for this file. 

Inside this file, you would want to require your modules. Apart from gulp itself, you would probably want to require modules that facilitate the commands you want gulp to run for you. For example, if your tests are written in mocha, and you client tests use Karma you would want to require their respective gulp plug-in after you npm installed them. Your `gulpfile.js` will start off looking like this
{% highlight javascript %}
var gulp = require('gulp');
var mocha = require('gulp-mocha');
var Server = require('karma').Server;
var jshint = require('gulp-jshint');

// specifications for running server-side tests
gulp.task('server-tests', function () {
  return gulp.src('path/to/server/specs/*.js', {read: false})
    .pipe(mocha({reporter: 'spec'}));
});

// specifications for running client-side tests
gulp.task('client-tests', function (done) {
  new Server({
    configFile: __dirname + '/karma.conf.js',
    singleRun: true
  }, done).start();
});

// specifications for jshint linter
gulp.task('linter', function () {
  return gulp.src(['path/to/client/files/*.js', 'path/to/server/files/*.js', 'path/to/spec/files/*.js'])
    .pipe(jshint())
    .pipe(jshint.reporter('default'))
    .pipe(jshint.reporter('fail'));
});

// registering default task to run linter, server-tests 
// and client-tests in such order
gulp.task('default', ['linter', 'server-tests', 'client-tests']);
{% endhighlight %}
Now with this gulpfile, I can run `$ gulp` in my command line and gulp will run all my tests for me under the default task.

Task runners can do a whole lot more than simply running your tests for you. While I can't cover all of their capabilities in this post, I hope it helped you in the process of getting setup, or at least motivated you to find out more about them.
