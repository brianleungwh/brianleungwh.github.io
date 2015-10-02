---
layout: post
title: Visualizing JavaScript Classical Inheritance
comments: True
---
Being used to Java's classical inheritance, understanding JavaScript's version was a challenge for me. Fortunately, [this article](http://eli.thegreenplace.net/2013/10/22/classical-inheritance-in-javascript-es5) does a great job on explaining what goes on under the hood and so I have decided to make a diagram out of the code example used there and have this post serve as an extension to it. For viewing conveniences, I have copied and pasted the code below. Notice I have also added two additional lines of code on line 35 and 36 to actually instantiate the two objects that is being defined.

{% highlight javascript linenos %}
// Shape - superclass
// x,y: location of shape's bounding rectangle
function Shape(x, y) {
  this.x = x;
  this.y = y;
}

// Superclass method
Shape.prototype.move = function(x, y) {
  this.x += x;
  this.y += y;
}

// Circle - subclass
function Circle(x, y, r) {
  // Call constructor of superclass to initialize 
  // superclass-derived members.
  Shape.call(this, x, y);

  // Initialize subclass's own members
  this.r = r;
}

// Circle derives from Shape
Circle.prototype = Object.create(Shape.prototype);
Circle.prototype.constructor = Circle;

// Subclass methods. Add them after Circle.prototype 
// is created with Object.create
Circle.prototype.area = function() {
  return this.r * 2 * Math.PI;
}

// Creating an instance of Shape and an instance of Circle
var shape = new Shape(2, 2);
var circle = new Circle(5, 5, 12);
{% endhighlight %}

The code above may be presented as the following relationship.
![Diagram]({{ site.baseurl }}/assets/images/2015-08-30-visualize-javascript-inheritance/js_inheritance.png)

If we go back to the code, line 25 and 26 is where we setup the prototype chain. For a more in-depth explaination, you may want to read the original article. But in short, what line 25 is essentially doing is that it sets `Circle.prototype.__proto__` to be `Shape.prototype`. So if we were to log 
{% highlight javascript %}
Circle.prototype.__proto__ === Shape.prototype
{% endhighlight %}
 it will evaluate to true.

Let's say we invoke `circle.move()`. JavsScript will first look for a `move` method in `Circle.prototype`. No `move` method found, it will then look in the object where `Circle.prototype.__proto__` is delegated to, which in this case we set it to the `Shape.prototype` and `move` is found there. In a case where JavaScript doesn't find the method in the superclass, it will look in the `Object.prototype` and if the method isn't found there either, it will throw an error because `Object.prototype` is the end of the prototype chain.

When we assign `Circle.prototype` to `Shape.prototype` we break the existing `Circle.prototype.constructor`. If we eliminate line 26, running `circle instanceof Circle` will return false. Line 26 is explcitly restoring that relationship.

If you would like to dig deeper into JavaScript's prototype chain you might find this useful: [JavaScript. The Core](http://dmitrysoshnikov.com/ecmascript/javascript-the-core/).
