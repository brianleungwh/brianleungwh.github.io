---
layout: post
title: Increase test modularity by using SinonJS
comments: True
---
I was recently working on an API server that is backed by a `mysql` database and my team had decided to use [Sequelize](http://sequelize.readthedocs.org/en/latest/) as our ORM. I wanted to write unit tests for the API server but I don't necessary want to have the test to actually interact with the live database server that is running on my local machine. After considering setting up a seperate database that is dedicated for testing, I have decided to use [SinonJS](http://sinonjs.org/docs/#stubs) and simply *stub* the query methods in my tests. This way, I don't even need to worry about wiping the tables in my database when running test repeatedly. On top of that, having standalone tests is always better than having tests tightly coupled with external resources.

Here is the code snippet of my test for the methods in a module called `userController`. Note that userController would normally interact with the database through the built-in Sequelize model methods. To avoid that, I am stubbing the build-in method `User.create` when creating a new user.

{% highlight javascript %}
// userControllerSpec.js

var expect = require('chai').expect;
var userController = require('../user/userController');
var sinon = require('sinon');
var User = require('../db/db').User;

describe('User Controller', function () {

  // Defining a user object for testing
  var testUser = {
                email: "test@gmail.com",
                facebookId: "118724648484592",
                oauthToken: "someToken",
              };

  // Declare a variable for the stub we would be using 
  // throughout the test
  var userStub;

  // After each test, we restore the stub
  afterEach(function (done) {
    userStub.restore();
    done();
  });

  it('should create a new user', function (done) {
    // Stubbing the sequeqlize model built-in create method
    // to prevent acutally creating a new model instance
    userStub = sinon.stub(User, 'create', function (newUser) {
      return new Promise(function (resolve) {
        resolve(newUser);
      });
    });

    userController.registerNewUser(testUser)
      .then(function (user) {
        expect(user.email).to.equal(testUser.email);
        expect(user.facebookId).to.equal(testUser.facebookId);
        done();
      });
  });

  it('should not create the same user twice', function (done) {
    // Here we are having the stub throw an error because 
    // we are pretending that the model instance for testUser has
    // already been created and inserted to the database 
    // by Sequelize
    userStub = sinon.stub(User, 'create', function (newUser) {
      return new Promise(function (resolve, reject) {
        var e = new Error();
        e.name = 'SequelizeUniqueConstraintError';
        reject(e);
      });
    });

    userController.registerNewUser(testUser)
      .catch(function (err) {
        expect(err.name).to.equal('SequelizeUniqueConstraintError');
        done();
      });
  });

  // More tests ...

});

{% endhighlight %}
