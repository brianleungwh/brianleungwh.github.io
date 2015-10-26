---
layout: post
title: Increase Test Modularity by Using SinonJS
comments: True
---
I was recently working on an Express server with a `mysql` database and we have decided to use [Sequelize](http://sequelize.readthedocs.org/en/latest/) as our ORM. I wanted to write unit tests for the API server but I don't necessary want to have the test to actually interact with the live database server that is running on my local machine during development. After considering setting up a seperate database that is dedicated for testing, I have decided to use [SinonJS](http://sinonjs.org/docs/#stubs) and  `stub` the `sequelize` methods that inserts data into the database. This way, I don't have to worry about wiping the tables in my database when running tests consecutively. On top of that, having standalone tests is always better than having tests that are tightly coupled with external resources.

Here is the code snippet of my test for the methods in a module called `userController`. Note that `userController` would normally create a new user by invoking `sequeqlize`'s `Model.create()` method to insert a new user instance to the database. To avoid that, I am stubbing the method `User.create()` when creating a new user.

{% highlight javascript %}
// userControllerSpec.js

// Requiring all test dependencies
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
    // Stubbing the User.create() method
    // Since User.create() will return a promise and pass 
    // in the newly created model instances to the resolve 
    // function upon successfully inserting it to the database, 
    // we will have the stub do the same to mock such behavior
    userStub = sinon.stub(User, 'create', function (newUser) {
      return new Promise(function (resolve) {
        resolve(newUser);
      });
    });

    // Actual test for registering a new user
    userController.registerNewUser(testUser)
      .then(function (user) {
        expect(user.email).to.equal(testUser.email);
        expect(user.facebookId).to.equal(testUser.facebookId);
        done();
      });
  });

  it('should not create the same user twice', function (done) {
    // Since users must be unique in our database, we
    // are having the stub throw an error to simulate
    // the effect that the model instance for testUser has
    // already been created and inserted, as this is what
    // sequeqlize would do if we try to create new instance
    // duplicates
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
