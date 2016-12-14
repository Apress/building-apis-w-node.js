# Testing the application - Part 1

## Introduction to Mocha

Creating automated tests is something highly recommended to use. There are several types of tests: **unitary**, **functional**, **acceptance** and more. In this chapter, let's focus only in the **acceptance's test**, in our case aims to test the outputs and behaviors of our API's routes.

To create and execute the tests, it's necessary to use a test runner. We'll use the Mocha, which is a very popular in Node.js community.

![Mocha test runner](images/mocha-logotipo.png)

Mocha has the following features:

* TDD style;
* BDD style;
* Code coverage HTML report;
* Customized test reports;
* Asynchronous test support;
* Easily integrated with the modules: `should`, `assert` and `chai`.

It is a complete environment to write tests and you learn more accessing his website: [mochajs.org](https://mochajs.org).

## Setting up the test environment

To setup our test's environment, first we are going to setup a new database that is going to be used for us to play with **some fake data**. This practice is largely used to make sure that an application can work easily in multiple environments. For now, our API has just single environment, because all the examples were developed in the `development` environment.

To enable the support for multiple environments, let's rename the current `libs/config.js` file to `libs/config.development.js` and then, create the `libs/config.test.js` file. The only new parameter in this new file is the `logging: false` which disables the SQL logs outputs. This will be necessary to disable these logs to not mess the tests report in the terminal. See how this file (`libs/config.test.js`) must look like:

``` javascript
module.exports = {
  database: "ntask_test",
  username: "",
  password: "",
  params: {
    dialect: "sqlite",
    storage: "ntask.sqlite",
    logging: false,
    define: {
      underscored: true
    }
  },
  jwtSecret: "NTASK_TEST",
  jwtSession: {session: false}
};
```

Now, we have to setup some files, each one has specific data to his correspondent environments. To load the settings according to the current environment, we must create a code to identifies what environment it is. In this case, we can use the `process.env` object which basically returns a lot of several environment variables from the OS.

A good practice in Node.js projects is to work with the variable `process.env.NODE_ENV` and use his value as a base, our application will have to load the settings from `test` or `development` environment. By default, the `development` will be defined when `process.env.NODE_ENV` returns null or an empty string.

Based on this brief explanation, let's recreate the `libs/config.js` file to load the settings according to the right system environment:

``` javascript
module.exports = app => {
  const env = process.env.NODE_ENV;
  if (env) {
    return require(`./config.${env}.js`);
  }
  return require("./config.development.js");
};
```

In our project, we are going to explore the **acceptance tests** only. To create them, we need to use these modules:

* `babel-register`: to run ES6 codes;
* `mocha`: to run the tests;
* `chai` to write BDD tests;
* `supertest` to do some requests in the API;

All these modules will be installed as a `devDependencies` in the `package.json` to use them only as test dependency. To do this, you need to use the `--save-dev` flag in the `npm install` command. See the command below:

``` bash
npm install babel-register@6.5.2 mocha@2.4.5 chai@3.5.0 supertest@1.2.0 --save-dev
```

Now, let's encapsulate the `mocha` test runner into the `npm test` alias command, to internally run the command: `NODE_ENV=test mocha test/**/*.js`. To implement this new command, edit the `package.json` and include the `scripts.test` attribute:

``` json
{
  "name": "ntask-api",
  "version": "1.0.0",
  "description": "Task list API",
  "main": "index.js",
  "scripts": {
    "start": "babel-node index.js",
    "test": "NODE_ENV=test mocha test/**/*.js"
  },
  "author": "Caio Ribeiro Pereira",
  "dependencies": {
    "babel-cli": "^6.5.1",
    "babel-preset-es2015": "^6.5.0",
    "bcrypt": "^0.8.5",
    "body-parser": "^1.15.0",
    "consign": "^0.1.2",
    "express": "^4.13.4",
    "jwt-simple": "^0.4.1",
    "passport": "^0.3.2",
    "passport-jwt": "^2.0.0",
    "sequelize": "^3.19.2",
    "sqlite3": "^3.1.1"
  },
  "devDependencies": {
    "babel-register": "^6.5.2",
    "chai": "^3.5.0",
    "mocha": "^2.4.5",
    "supertest": "^1.2.0"
  }
}
```

Then, we are going to export our main API module, the `index.js`, to allow the API can be started during the tests. To do it, you must include the `module.exports = app` in the end of the `index.js` file and we'll also disable the logs created by the `consign` module via `consign({verbose: false})` settings to not pollute the tests report.

``` javascript
import express from "express";
import consign from "consign";

const app = express();

consign({verbose: false})
  .include("libs/config.js")
  .then("db.js")
  .then("auth.js")
  .then("libs/middlewares.js")
  .then("routes")
  .then("libs/boot.js")
  .into(app);

module.exports = app;
```

Now, the application can be internally started by the `supertest` module during the tests. To avoid the server runs twice in the tests environment, you need to modify the `libs/boot.js` to run the database sync and server listen only when `process.env.NODE_ENV` has not the `test` value.

To change this, open and edit the `libs/boot.js` using the simple code below:

``` javascript
module.exports = app => {
  if (process.env.NODE_ENV !== "test") {
    app.db.sequelize.sync().done(() => {
      app.listen(app.get("port"), () => {
        console.log(`NTask API - Port ${app.get("port")}`);
      });
    });
  }
};
```

To finish our test's environment setup, let's prepare some Mocha specific settings, to load the API server and the modules `chai` and `supertest` as global variables. This will accelerate the execution of tests, after all, each one will load these modules again and again, and if we put all main things to load once, will save some milliseconds of the tests execution. To implement this simple practice, create the file `test/helpers.js`:

``` javascript
import supertest from "supertest";
import chai from "chai";
import app from "../index.js";

global.app = app;
global.request = supertest(app);
global.expect = chai.expect;
```

Then, let's create a simple file which allows to include some settings as parameters to the `mocha` command. This will be responsible to load the `test/helpers.js` and also use the `--reporter spec` flag to show a detailed report about the tests. After that, we'll include the `--compilers js:babel-register` flag, to Mocha be able to run the tests in **ECMAScript 6** standard via `babel-register` module.

The last flag is `--slow 5000`, this will wait five seconds before start all tests (time enough to start the API server and database connection safely). Create the `test/mocha.opts` file using the following parameters:

``` bash
--require test/helpers
--reporter spec
--compilers js:babel-register
--slow 5000
```

### Writing the firsts tests

We finished the setup about the test environment. What about test something? What about write some tests code for `routes/index.js`? It's very simple to test: basically we will make sure the API is returning the JSON correctly, comparing the results with the static JSON `const expected = {status: "NTask API"}` to see if both are matching.

To create our first test, let's use the `request.get("/")` function, to validate if this request is returning the status `200`. To finish this test, we check if the `req.body` and `expected` are the same using `expect(res.body).to.eql(expected)` function.

To implement this test, create the `test/routes/index.js` file using the following codes:

``` javascript
describe("Routes: Index", () => {
  describe("GET /", () => {
    it("returns the API status", done => {
      request.get("/")
        .expect(200)
        .end((err, res) => {
          const expected = {status: "NTask API"};
          expect(res.body).to.eql(expected);
          done(err);
        });
    });
  });
});
```

To execute this test, run the command:

``` bash
npm test
```

After the execution, you will have a similar output like this image:

![Running the first test](images/primeiro-teste-com-mocha.png)

## Testing the authentication endpoint

In this section, we'll implement several tests. To start off, let's test the endpoint from `routes/token.js`, which is responsible to generates JSON web tokens for authenticated users.

Basically, this endpoint will have four tests to validate:

* Request authenticated by a valid user;
* Request with a valid e-mail but with wrong password;
* Request with an unregistered e-mail;
* Request without email and password;

Create the test `test/routes/token.js` with the following structure:

``` javascript
describe("Routes: Token", () => {
  const Users = app.db.models.Users;
  describe("POST /token", () => {
    beforeEach(done => {
      // Runs before each test...
    });
    describe("status 200", () => {
      it("returns authenticated user token", done => {
        // Test's logic...
      });
    });
    describe("status 401", () => {
      it("throws error when password is incorrect", done => {
        // Test's logic...
      });
      it("throws error when email not exist", done => {
        // Test's logic...
      });
      it("throws error when email and password are blank", done => {
        // Test's logic...
      });
    });
  });
});
```

To start write these tests, first, we're gonna code some queries to clear the user's table and create one valid user inside `beforeEach()` callback. This function will be executed before each test. To do this, we'll use the model `app.db.models.Users` and his functions: `Users.destroy({where: {}})` to clean the user's table and `Users.create()` to save a single valid user for each test execution. This will allow to test the main flows of this route:

``` javascript
beforeEach(done => {
  Users
    .destroy({where: {}})
    .then(() => Users.create({
      name: "John",
      email: "john@mail.net",
      password: "12345"
    }))
    .then(() => done());
});
```

Now, we are going to implement test by test. The first test is a successful case. To test it, let's use the function `request.post("/token")` to request a token by sending the email and the password of a valid user via `send()` function.

To finish the test, the `end(err, res)` callback must return `res.body` with the `token` key to be checked via `expect(res.body).to.include.keys("token")` function. To conclude a test, it's required to execute the callback `done()` in the end of a test.

Always send the variable `err` as a parameter into `done(err)` function, because if something wrong happens during the tests, this function will show the details of the error. See below the complete code of this first test:

``` javascript
it("returns authenticated user token", done => {
  request.post("/token")
    .send({
      email: "john@mail.net",
      password: "12345"
    })
    .expect(200)
    .end((err, res) => {
      expect(res.body).to.include.keys("token");
      done(err);
    });
});
```

After this first test, we'll write some tests to verify if the errors is happening well. Now, let's test the request of an invalid password, expecting a `401 unauthorized access` status code. This tests is simpler, because basically we'll test if the request returns the `status 401` error, via the function `expect(401)`:

``` javascript
it("throws error when password is incorrect", done => {
  request.post("/token")
    .send({
      email: "john@mail.net",
      password: "WRONG_PASSWORD"
    })
    .expect(401)
    .end((err, res) => {
      done(err);
    });
});
```

The next test is very similar to the last one, but now, it'll be tested the invalid user's email behavior, expecting the request returns `401` status code again:

``` javascript
it("throws error when email not exist", done => {
  request.post("/token")
    .send({
      email: "wrong@email.com",
      password: "12345"
    })
    .expect(401)
    .end((err, res) => {
      done(err);
    });
});
```

And, to finish this test case, let's check if we get the same `401` status code when no email and no password are sent. This one is even simpler, because we don't need to send parameters in this request.

``` javascript
it("throws error when email and password are blank", done => {
  request.post("/token")
    .expect(401)
    .end((err, res) => {
      done(err);
    });
});
```

### Conclusion

Congrats! To run all tests, just type the `npm test` command again, now, probably you'll get a similar result as this image is:

![Test's result](images/testes-finais-parte1.png)

Keep reading because the tests subject is long and we'll keep talking about it in the next chapter to write more tests for our API's routes.
