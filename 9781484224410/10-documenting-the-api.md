# Documenting the API

If you reached this chapter and your application is working correctly – with routes to the management tasks and users, integrated to a database and with user's authentication via JSON Web Token – congratulations! You have created, following some best practices, an REST API using Node.js. If you intend to use this pilot project as a base to construct your own API, then you already have an application enough to deploy it into a production environment.

## Introduction to ApiDoc.js

In this chapter, we'll learn how to write and generate a pretty API documentation, after all, it is a good practice to provide documentation about how the client applications can connect to consume the data from an API. The coolest thing is that we are going to use a very simple tool and all the documentation of our application will be built using code's comment.

Our project will use the **ApiDoc.js**, which is a Node.js module to generate elegants documentation for APIs.

![ApiDoc.js homepage](images/site-apidocjs.png)

This module is a CLI *(Command Line Interface)* and it is highly advisable to install it as a global module (using the command `npm install –g`). However, in our case, we can use it as local module and we'll create a npm alias command to use every time we start the API server. So, its installation is going to be as a local module, similar to the others. Install it using the command:

``` bash
npm install apidoc@0.15.1 --save-dev
```

First, let's create the command: `npm run apidoc` to execute internally the command `apidoc -i routes/ -o public/apidoc`. Then, modify the attribute `scripts.start` so it can generate the API documentation before start the API server. We also gonna include the attribute `apidoc.name` to set a title and the `apidoc.template.forceLanguage` to set the default language to be used in the API documentation.

Open and edit the `package.json`, doing these changes:

``` json
{
  "name": "ntask-api",
  "version": "1.0.0",
  "description": "Task list API",
  "main": "index.js",
  "scripts": {
    "start": "npm run apidoc && babel-node index.js",
    "apidoc": "apidoc -i routes/ -o public/apidoc",
    "test": "NODE_ENV=test mocha test/**/*.js"
  },
  "apidoc": {
    "name": "Node Task API - Documentation",
    "template": {
      "forceLanguage": "en"
    }
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
    "apidoc": "^0.15.1",
    "babel-register": "^6.5.2",
    "chai": "^3.5.0",
    "mocha": "^2.4.5",
    "supertest": "^1.2.0"
  }
}
```

Now, every time you run the command `npm start`, if you want to generate a new documentation without initiating the server, you just run `npm run apidoc` command. Both of the commands are going to search all the existent comments in the `routes` directory  to generate a new API documentation, which will be published in the `public/apidoc` folder and then, start the server.

To be able to view the documentation page, first we have to enable our API to server static file from the folder `public`. To enable it, you have to import the `express` module to use the middleware `app.use(express.static("public"))` at the end of the `libs/middlewares.js` file and don't forget to create the `public` empty folder. See how it looks:

``` javascript
import bodyParser from "body-parser";
import express from "express";

module.exports = app => {
  app.set("port", 3000);
  app.set("json spaces", 4);
  app.use(bodyParser.json());
  app.use(app.auth.initialize());
  app.use((req, res, next) => {
    delete req.body.id;
    next();
  });
  app.use(express.static("public"));
};
```

To validate if everything is working fine, let's start documenting about the `GET /` endpoint, for this endpoint we'll use the following comments:

* `@api`: informs the type, the address and the title of the endpoint;
* `@apiGroup`: informs the endpoint group name;
* `@apiSuccess`: describes the fields and their data types for a successful's response;
* `@apiSuccessExample`: shows a output's sample of a successful's response.

To document this endpoint, edit the file `routes/index.js` writing this code below:

``` javascript
module.exports = app => {
  /**
   * @api {get} / API Status
   * @apiGroup Status
   * @apiSuccess {String} status API Status' message
   * @apiSuccessExample {json} Success
   *    HTTP/1.1 200 OK
   *    {"status": "NTask API"}
   */
  app.get("/", (req, res) => {
    res.json({status: "NTask API"});
  });
};
```

To test these changes, restart the server, then open the browser and go to: [localhost:3000/apidoc](http://localhost:3000/apidoc).

If no errors occur, you will see a beautiful documentation page about our API.

![API documentation page](images/documentacao-api-inicial.png)

## Documenting token generation

Now, we are going to explore the ApiDoc's features deeper, documenting all API's routes.

To start doing this, let's document the route `POST /token`. This has some extra details to be documented. To do this, we'll use not only the items from the last section but some new ones as well:

* `@apiParam`: describes an input parameter, which may be or not required its submission in a request;
* `@apiParamExample`: shows a sample of a input parameter, in our case, we will display a JSON input format;
* `@apiErrorExample`: shows a sample of some errors which could be generated by the API if something wrong happens.

To understand in practice the usage of these new items, let's edit the `routes/token.js`, following the comments below:

``` javascript
import jwt from "jwt-simple";

module.exports = app => {
  const cfg = app.libs.config;
  const Users = app.db.models.Users;

  /**
   * @api {post} /token Authentication Token
   * @apiGroup Credentials
   * @apiParam {String} email User email
   * @apiParam {String} password User password
   * @apiParamExample {json} Input
   *    {
   *      "email": "john@connor.net",
   *      "password": "123456"
   *    }
   * @apiSuccess {String} token Token of authenticated user
   * @apiSuccessExample {json} Success
   *    HTTP/1.1 200 OK
   *    {"token": "xyz.abc.123.hgf"}
   * @apiErrorExample {json} Authentication error
   *    HTTP/1.1 401 Unauthorized
   */
  app.post("/token", (req, res) => {
    // The code here was explained in chapter 7.
  });
};
```

## Documenting user's resource

In this section and in the next one, we'll focus to document the main resources of our API. As the majority of the routes of these resources need an authenticated **token** - which is sent in the request's header - we are going to use the following items to describe more about this header:

* `@apiHeader`: describes the name and data type of a header;
* `@apiHeaderExample`: shows a sample of header to be used in the request.

Open the `routes/users.js` and let's doc it! First, we'll doc the `GET /user` endpoint:

``` javascript
module.exports = app => {
  const Users = app.db.models.Users;

  app.route("/user")
    .all(app.auth.authenticate())
    /**
     * @api {get} /user Return the authenticated user's data
     * @apiGroup User
     * @apiHeader {String} Authorization Token of authenticated user
     * @apiHeaderExample {json} Header
     *    {"Authorization": "JWT xyz.abc.123.hgf"}
     * @apiSuccess {Number} id User id
     * @apiSuccess {String} name User name
     * @apiSuccess {String} email User email
     * @apiSuccessExample {json} Success
     *    HTTP/1.1 200 OK
     *    {
     *      "id": 1,
     *      "name": "John Connor",
     *      "email": "john@connor.net"
     *    }
     * @apiErrorExample {json} Find error
     *    HTTP/1.1 412 Precondition Failed
     */
    .get((req, res) => {
      // GET /user logic...
    })
```

Then, let's doc the route `DELETE /user`:

``` javascript
    /**
     * @api {delete} /user Deletes an authenticated user
     * @apiGroup User
     * @apiHeader {String} Authorization Token of authenticated user
     * @apiHeaderExample {json} Header
     *    {"Authorization": "JWT xyz.abc.123.hgf"}
     * @apiSuccessExample {json} Success
     *    HTTP/1.1 204 No Content
     * @apiErrorExample {json} Delete error
     *    HTTP/1.1 412 Precondition Failed
     */
    .delete((req, res) => {
    // DELETE /user logic...
    })
```

To finish this endpoint, we need to doc its last route, the `POST /user`, now we'll use several items to describe the input and output fields of this route:

``` javascript
    /**
     * @api {post} /users Register a new user
     * @apiGroup User
     * @apiParam {String} name User name
     * @apiParam {String} email User email
     * @apiParam {String} password User password
     * @apiParamExample {json} Input
     *    {
     *      "name": "John Connor",
     *      "email": "john@connor.net",
     *      "password": "123456"
     *    }
     * @apiSuccess {Number} id User id
     * @apiSuccess {String} name User name
     * @apiSuccess {String} email User email
     * @apiSuccess {String} password User encrypted password
     * @apiSuccess {Date} updated_at Update's date
     * @apiSuccess {Date} created_at Register's date
     * @apiSuccessExample {json} Success
     *    HTTP/1.1 200 OK
     *    {
     *      "id": 1,
     *      "name": "John Connor",
     *      "email": "john@connor.net",
     *      "password": "$2a$10$SK1B1",
     *      "updated_at": "2016-02-10T15:20:11.700Z",
     *      "created_at": "2016-02-10T15:29:11.700Z",
     *    }
     * @apiErrorExample {json} Register error
     *    HTTP/1.1 412 Precondition Failed
     */
    app.post("/users", (req, res) => {
      // POST /users logic...
    });
};
```

## Documenting task's resource

Continuing our API documentation, now we need to finish the task's routes documentation, let's edit the `routes/tasks.js` file, initially describing the `GET /tasks` route:

``` javascript
module.exports = app => {
  const Tasks = app.db.models.Tasks;

  app.route("/tasks")
    .all(app.auth.authenticate())
    /**
     * @api {get} /tasks List the user's tasks
     * @apiGroup Tasks
     * @apiHeader {String} Authorization Token of authenticated user
     * @apiHeaderExample {json} Header
     *    {"Authorization": "JWT xyz.abc.123.hgf"}
     * @apiSuccess {Object[]} tasks Task's list
     * @apiSuccess {Number} tasks.id Task id
     * @apiSuccess {String} tasks.title Task title
     * @apiSuccess {Boolean} tasks.done Task is done?
     * @apiSuccess {Date} tasks.updated_at Update's date
     * @apiSuccess {Date} tasks.created_at Register's date
     * @apiSuccess {Number} tasks.user_id Id do usuário
     * @apiSuccessExample {json} Success
     *    HTTP/1.1 200 OK
     *    [{
     *      "id": 1,
     *      "title": "Study",
     *      "done": false
     *      "updated_at": "2016-02-10T15:46:51.778Z",
     *      "created_at": "2016-02-10T15:46:51.778Z",
     *      "user_id": 1
     *    }]
     * @apiErrorExample {json} List error
     *    HTTP/1.1 412 Precondition Failed
     */
     .get((req, res) => {
      // GET /tasks logic...
     })
```

Then, let's doc the route `POST /tasks`:

``` javascript
    /**
     * @api {post} /tasks Register a new task
     * @apiGroup Tasks
     * @apiHeader {String} Authorization Token of authenticated user
     * @apiHeaderExample {json} Header
     *    {"Authorization": "JWT xyz.abc.123.hgf"}
     * @apiParam {String} title Task title
     * @apiParamExample {json} Input
     *    {"title": "Study"}
     * @apiSuccess {Number} id Task id
     * @apiSuccess {String} title Task title
     * @apiSuccess {Boolean} done=false Task is done?
     * @apiSuccess {Date} updated_at Update's date
     * @apiSuccess {Date} created_at Register's date
     * @apiSuccess {Number} user_id User id
     * @apiSuccessExample {json} Success
     *    HTTP/1.1 200 OK
     *    {
     *      "id": 1,
     *      "title": "Study",
     *      "done": false,
     *      "updated_at": "2016-02-10T15:46:51.778Z",
     *      "created_at": "2016-02-10T15:46:51.778Z",
     *      "user_id": 1
     *    }
     * @apiErrorExample {json} Register error
     *    HTTP/1.1 412 Precondition Failed
     */
     .post((req, res) => {
     // POST /tasks logic...
     })
```

Then, we are going to doc the route `GET /tasks/:id` with the following comments:

``` javascript
    /**
     * @api {get} /tasks/:id Get a task
     * @apiGroup Tasks
     * @apiHeader {String} Authorization Token of authenticated user
     * @apiHeaderExample {json} Header
     *    {"Authorization": "JWT xyz.abc.123.hgf"}
     * @apiParam {id} id Task id
     * @apiSuccess {Number} id Task id
     * @apiSuccess {String} title Task title
     * @apiSuccess {Boolean} done Task is done?
     * @apiSuccess {Date} updated_at Update's date
     * @apiSuccess {Date} created_at Register's date
     * @apiSuccess {Number} user_id User id
     * @apiSuccessExample {json} Success
     *    HTTP/1.1 200 OK
     *    {
     *      "id": 1,
     *      "title": "Study",
     *      "done": false
     *      "updated_at": "2016-02-10T15:46:51.778Z",
     *      "created_at": "2016-02-10T15:46:51.778Z",
     *      "user_id": 1
     *    }
     * @apiErrorExample {json} Task not found error
     *    HTTP/1.1 404 Not Found
     * @apiErrorExample {json} Find error
     *    HTTP/1.1 412 Precondition Failed
     */
     .get((req, res) => {
      // GET /tasks/:id logic...
     })
```

Now, the `PUT /tasks/:id`:

``` javascript
    /**
     * @api {put} /tasks/:id Update a task
     * @apiGroup Tasks
     * @apiHeader {String} Authorization Token of authenticated user
     * @apiHeaderExample {json} Header
     *    {"Authorization": "JWT xyz.abc.123.hgf"}
     * @apiParam {id} id Task id
     * @apiParam {String} title Task title
     * @apiParam {Boolean} done Task is done?
     * @apiParamExample {json} Input
     *    {
     *      "title": "Work",
     *      "done": true
     *    }
     * @apiSuccessExample {json} Success
     *    HTTP/1.1 204 No Content
     * @apiErrorExample {json} Update error
     *    HTTP/1.1 412 Precondition Failed
     */
     .put((req, res) => {
      // PUT /tasks/:id logic...
     })
```

At last, let's finish this chapter documenting the route `DELETE /tasks/:id`:

``` javascript
    /**
     * @api {delete} /tasks/:id Remove a task
     * @apiGroup Tasks
     * @apiHeader {String} Authorization Token of authenticated user
     * @apiHeaderExample {json} Header
     *    {"Authorization": "JWT xyz.abc.123.hgf"}
     * @apiParam {id} id Task id
     * @apiSuccessExample {json} Success
     *    HTTP/1.1 204 No Content
     * @apiErrorExample {json} Delete error
     *    HTTP/1.1 412 Precondition Failed
     */
     .delete((req, res) => {
      // DELETE /tasks/:id logic...
     });
};
```

Let’s test? Just restart the server and then go to: [localhost:3000/apidoc](http://localhost:3000/apidoc).

This time, we have a complete documentation page that describes step-by-step for a new developer how to create a client application to consume data from our API.

![Now our API is well documented](images/documentacao-api-completa.png)

### Conclusion

Congratulations! We have finished another excellent chapter. Now we not only have a functional API as well as a complete documentation to allow other developers to create client-side applications using our API.

Keep reading because, in the next episode, we'll include some frameworks and best practices for our API works fine in a production environment.
