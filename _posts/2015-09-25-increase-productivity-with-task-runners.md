---
layout: post
title: Increase Our Productivity with Task Runners
comments: true
---
During software development, there are often many commands and scripts we would want to run before we git commit and push it to github or even deploy. We may want to run tests for our entire stack, and we may also want to run our style checker to make sure we have not violated the style guide. Without task runners, we would probably have to do something like this:
{% highlight bash %}
$ jshint files/to/check
$ mocha -R spec path/to/client/spec/*.js
$ npm test
$ and more...
$ and more...
{% endhighlight %}
While there is nothing wrong with doing this, we can increase our productivity by utilizing something called task runners to run all those commands for us automatically. Two of the most poplular tasks runners for web development are currently Grunt and Gulp, and they are pretty easy to setup considering how much time we could potentially save down the road. For this example, I will demonstrate how to get Gulp setup. Before we dive in, make sure you have `npm` installed.

###### Installing Gulp on our computer
To begin, run
{% highlight bash %}
$ npm install -g gulp
{% endhighlight %}
For this step, it doesn't matter what our current directory is. The `-g` flag tells npm to install Gulp for us *globally* .

###### Installing Gulp to our project
Then, we go to the root of our project repo and run 
{% highlight bash %}
$ npm install --save-dev gulp
{% endhighlight %}
This will add Gulp to our `package.json` and install all it's dependencies to our `node_module` so that we can `require` it in our gulpfile. 

###### Setting up our Gulpfile.js
Now, we can go ahead and create a `gulpfile.js` at the root of our project repo. Note that we can't name this file anything other than that because when we run `gulp` later, it looks specifically for this file. 

Inside this file, we want to `require` our modules. Apart from gulp itself, we would want to `require` modules that facilitate the commands we want Gulp to run for us. For this example, assuming our tests are written in mocha, and our client tests use Karma, we would want to `require` their respective gulp plug-ins after we npm installed them. Our `gulpfile.js` will start off looking like this
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

Task runners can do a whole lot more than simply running our tests for us. While I can't cover all of their capabilities in this post, I hope it helped you in the process of getting setup, or at least motivated you to find out more about them.
