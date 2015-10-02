---
layout: post
title: Javascript Hoisting
comments: True
---
{% highlight javascript linenos %}
var msg = 'JavaScript is awesome';
var fn = function() {
  console.log(msg);
}
fn();
{% endhighlight %}

Invoking `fn()` will log `JavaScript is awesome`.  
Okay, no surprises.   
What about this...

{% highlight javascript linenos %}
var msg = 'JavaScript is awesome';
var fn = function() {
  msg = 'JavaScript is weird';
  console.log(msg);
  var msg;
}
fn();
{% endhighlight %}
In this case, it will log `JavaScript is weird`.
This is called variable hoisting.
Turns out, in JavaScript, variable and function declarations are hoisted to the top of their *containing scope* before execution time, with function declarations taking priority over variables. When a function declaration gets hoisted, the entire function body will be hoisted as well. Whereas for variables, only the variable name is hoisted and they are assigned `undefined` by the JavaScript interpreter until the actual assignment statement is ran during execution time. 
Lets take a look at the following example.

{% highlight javascript linenos %}
var foo = function() {
  console.log("outside foo");
};

var test = function() {
  foo();
  var foo = function() {
    console.log("inside foo");
  }
};
test(); // Error: undefined is not a function
{% endhighlight %}

When we invoke `test()`, the variable `foo` is invisibly hoisted to the top of the local scope and assigned `undefined` before execution. We can imagine this taking place before line 6 but after line 5. Therefore, when we invoke `foo()` inside the `test` scope in line 6, the interpreter throws an error as `foo` is not assigned a value until line 7. Notice I refer to `foo` as a variable because anything that takes the form `var someName;` is considered a variable declaration. In this case, `foo` just so happen to be declared and be assgined to a function at the same time in line 7. But of course, with JavaScript hoisting, it gets delcared and assigned `undefined` before exeution. 
Experiment with this piece of code <a href="http://www.pythontutor.com/javascript.html#code=var+foo+%3D+function(%29+%7B%0A++console.log(%22outside+foo%22%29%3B%0A%7D%3B%0A%0Avar+test+%3D+function(%29+%7B%0A++foo(%29%3B%0A++var+foo+%3D+function(%29+%7B%0A++++console.log(%22inside+foo%22%29%3B%0A++%7D%0A%7D%3B%0Atest(%29%3B&mode=display&origin=opt-frontend.js&cumulative=false&heapPrimitives=false&textReferences=false&py=js&rawInputLstJSON=%5B%5D&curInstr=0" target="_blank">here</a>.

Here we are declaring `foo` as a function instead of a variable.
{% highlight javascript linenos %}
var foo = function() {
  console.log("outside foo");
};

var test = function() {
  foo();
  function foo() {
    console.log("inside foo");
  }
};
test(); // inside foo
{% endhighlight %}

JavaScript logs `inside foo` in this case because, as mentioned earlier, with function declaration not only does its name gets hoisted, so does its body. Anything that takes the form `function funcName() {};` in JavaScript is interpreted as a function declaration and when they get hoisted, their function definition get hoisted too. Experiement with this piece of code <a href="http://www.pythontutor.com/javascript.html#code=var+foo+%3D+function(%29+%7B%0A++console.log(%22outside+foo%22%29%3B%0A%7D%3B%0A%0Avar+test+%3D+function(%29+%7B%0A++foo(%29%3B%0A++function+foo(%29+%7B%0A++++console.log(%22inside+foo%22%29%3B%0A++%7D%0A%7D%3B%0Atest(%29%3B+//+inside+foo&mode=display&origin=opt-frontend.js&cumulative=false&heapPrimitives=false&textReferences=false&py=js&rawInputLstJSON=%5B%5D&curInstr=0" target="_blank">here</a>. 

With this in mind, it is generally good practice to declare all our variables at the top of each scope to avoid confusions down the road. For more dedicated readings <a href="http://www.adequatelygood.com/JavaScript-Scoping-and-Hoisting.html" target="_blank">go here</a>.
