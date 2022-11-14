# Authenticating users

Our API already has a tasks and users resources, which thanks to the Sequelize framework is integrated to a SQL database – in our case, SQLite3. We have already implemented their routes via the main routing functions provided by Express.

In this chapter, we'll explore the main concepts and implementations of user's authentication. After all, this is an important step to ensure that our users can manage their tasks safely.

## Introduction to Passport.js and JWT

### About Passport.js

There is a Node.js module very cool and easy to work with user’s authentication, it’s called **Passport**.

Passport is a framework that is extremely flexible and modular. It allows you to work with the main authentication strategies: **Basic & Digest**, **OpenID**, **OAuth**, **OAuth 2.0** and **JWT**. And also allows you to work with external services authentication as **Single Sing-on** by using existing social networking accounts information, such as **Facebook**, **Google+**, **Twitter** and more. By the way, in its official website, **there is a list with +300 authentication strategies**, created and maintained by 3rd-party.

![Passport homepage](images/site-passport.jpg)

Its official website is: [passportjs.org](http://passportjs.org).

### About JWT

JWT *(JSON Web Tokens)* is a very simple and secure authentication's strategy for REST APIs. It is a *open standard* for web authentications and is based in JSON token's requests between client and server. Its authentication's engine works like this:

1. Client makes a request once by sending their login credentials and password;
2. Server validates the credentials and, if everything is right, it returns to the client a JSON with token that encodes data from a user logged into the system, optionally this token could have a expiring date, to enforce the authentication's security;
3. Client, after receiving this token, can store it the way it wants, whether via LocalStorage, Cookie or other client-side storage mechanisms;
4. Every time the client accesses a route that requires authentication, it will only send this token to the API to authenticate and release consumption data;
5. Server always validate this token to allow or not a customer request.

To specific details about JWT, go to [jwt.io](http://jwt.io).

![JWT homepage](images/site-jwt.jpg)

## Installing Passport and JWT

To start the fun, we'll use the following modules:

* **Passport:** as authentication's engine;
* **Passport JWT:** as JWT authentication's strategy for Passport;
* **JWT Simple:** as encoder and decoder JSON tokens;

Now, let's install them running this command:

``` bash
npm install passport@0.3.2 passport-jwt@2.0.0 jwt-simple@0.4.1 --save
```

To start this implementation, first we are going to add two new settings items for JWT (`jwtSecret` and `jwtSession`). Edit the `libs/config.js` file and in the end, add the following attributes:

``` javascript
module.exports = {
  database: "ntask",
  username: "",
  password: "",
  params: {
    dialect: "sqlite",
    storage: "ntask.sqlite",
    define: {
      underscored: true
    }
  },
  jwtSecret: "Nta$K-AP1",
  jwtSession: {session: false}
};
```

The field `jwtSecret` keeps a secret key string that serves as a base to **encode/decode** tokens. It’s highly advisable to use a complex string which uses many different characters.

**Never share or publish this secret key in public**, because, if it leaks, you will let your application vulnerable, allowing a bad intentioned person to access the system and manages the tokens from logged users without informing the correctly credentials in the system.

To finish, the last included field is `jwtSession` that has the value `{session: false}`. This item is going to be used to inform Passport that the API won't manage session.

## Implementing JWT authentication

Now that we have the Passport and JWT settings ready, let's implement the rules on how the client will be authenticated in our API. To start, we are going to implement the authentication rules, that will also have middleware functions provided by Passport to use into the API's routes. This code is going to have a middleware and two functions. The middleware will be executed in the moment it starts the application, and it basically receive in his callback a `payload` that contains a **decoded JSON** which was decoded using the secret key `cfg.jwtSecret`. This `payload` will have the attribute `id` that will be a user `id` to be used as argument for `Users.findById(payload.id)` function. As this middleware is going to be frequently accessed, to avoid some overheads, we are going to send a simple object containing only the `id` and `email` of the authenticated user, using the callback function:

``` javascript
done(null, {id: user.id, email: user.email});
```

This middleware will be injected via `passport.use(strategy)` function. To finish, two functions will be included from Passport to be used on the application. They are the `initialize()` function which starts the Passport and `authenticate()` which is used to authenticate the access for a route.

To understand better this implementation, let's create in the root folder the `auth.js` file, using this code:

``` javascript
import passport from "passport";
import {Strategy, ExtractJwt} from "passport-jwt";

module.exports = app => {
  const Users = app.db.models.Users;
  const cfg = app.libs.config;
  const params = {
    secretOrKey: cfg.jwtSecret,
    jwtFromRequest: ExtractJwt.fromAuthHeader()
  };
  const strategy = new Strategy(params, (payload, done) => {
      Users.findById(payload.id)
        .then(user => {
          if (user) {
            return done(null, {
              id: user.id,
              email: user.email
            });
          }
          return done(null, false);
        })
        .catch(error => done(error, null));
    });
  passport.use(strategy);
  return {
    initialize: () => {
      return passport.initialize();
    },
    authenticate: () => {
      return passport.authenticate("jwt", cfg.jwtSession);
    }
  };
};
```

To load the `auth.js` during the server boot time, edit the `index.js` code like this:

``` javascript
import express from "express";
import consign from "consign";

const app = express();

consign()
  .include("libs/config.js")
  .then("db.js")
  .then("auth.js")
  .then("libs/middlewares.js")
  .then("routes")
  .then("libs/boot.js")
  .into(app);
```

To initiate the Passport middleware, edit the `libs/middlewares.js` and include the middleware `app.use(app.auth.initialize())`. See below were to include it:

``` javascript
import bodyParser from "body-parser";

module.exports = app => {
  app.set("port", 3000);
  app.set("json spaces", 4);
  app.use(bodyParser.json());
  app.use(app.auth.initialize());
  app.use((req, res, next) => {
    delete req.body.id;
    next();
  });
};
```

## Generating tokens for authenticated users

To finish the JWT authentication, we are going to prepare the model `Users` to be able to encrypt the user's password. We also will create a route to generate tokens users who are going to authenticate themselves using their login and password on the system, and we'll do a refactoring in the tasks and users routes so that their access properly use the `id` of a authenticated user. Doing this, we finish this authentication step, making our application reliable and safer.

The encryption of user passwords will be performed by the module `bcrypt`. To do this, install it by running the command:

``` bash
npm install bcrypt@0.8.5 --save
```

Now, let's edit the `Users` model including the function `hooks`, which executes functions before or after a database operation. In our case, will be included a function to be executed before registering a new user, via the function `beforeCreate()` to use the `bcrypt` to encrypt the user's password before save it.

A new function inside the `classMethods` will be included as well. It is going to be used to compare if the given password matches with the user's encrypted one. To encode these rules, edit the `models/users.js` with the following logic:

``` javascript
import bcrypt from "bcrypt";

module.exports = (sequelize, DataType) => {
  const Users = sequelize.define("Users", {
    // Users fields, defined in the chapter 5...
  }, {
    hooks: {
      beforeCreate: user => {
        const salt = bcrypt.genSaltSync();
        user.password = bcrypt.hashSync(user.password, salt);
      }
    },
    classMethods: {
      associate: models => {
        Users.hasMany(models.Tasks);
      },
      isPassword: (encodedPassword, password) => {
        return bcrypt.compareSync(password, encodedPassword);
      }
    }
  });
  return Users;
};
```

With these implemented modifications on the model `Users`, now we can create the endpoint `/token`. This route will be responsible for generating an encoded token with a `payload`, given to the user that sends the right e-mail and password via `req.body.email` e `req.body.password`.

The `payload` is going to have only the user `id`. The token generation occurs via `jwt-simple` module using the function `jwt.encode(payload, cfg.jwtSecret)` that mandatorily, will use the same secret key `jwtSecret` which was created on the `libs/config.js` file. Any error in this route will be treated using the HTTP `401 - Unauthorized` status code using the `res.sendStatus(401)` function.

To include this rule of tokens generation, you need to create the `routes/token.js` file using the following code:

``` javascript
import jwt from "jwt-simple";

module.exports = app => {
  const cfg = app.libs.config;
  const Users = app.db.models.Users;
  app.post("/token", (req, res) => {
    if (req.body.email && req.body.password) {
      const email = req.body.email;
      const password = req.body.password;
      Users.findOne({where: {email: email}})
        .then(user => {
          if (Users.isPassword(user.password, password)) {
            const payload = {id: user.id};
            res.json({
              token: jwt.encode(payload, cfg.jwtSecret)
            });
          } else {
            res.sendStatus(401);
          }
        })
        .catch(error => res.sendStatus(401));
    } else {
      res.sendStatus(401);
    }
  });
};
```

We already have the user's authentication and also the token generation's logic. To finish, let's use the `app.auth.authenticate()` function to validates the tokens sent by the clients and allows (or not) the access in some routes. To do this, edit the `routes/tasks.js` file and add the middleware function `all(app.auth.authenticate())` in the beginning of both endpoints. See below how will be:

``` javascript
module.exports = app => {
  const Tasks = app.db.models.Tasks;

  app.route("/tasks")
    .all(app.auth.authenticate())
    .get((req, res) => {
      // "/tasks": List tasks
    })
    .post((req, res) => {
      // "/tasks": Save new task
    });

  app.route("/tasks/:id")
    .all(app.auth.authenticate())
    .get((req, res) => {
      // "/tasks/1": Find a task
    })
    .put((req, res) => {
      // "/tasks/1": Update a task
    })
    .delete((req, res) => {
      // "/tasks/1": Delete a task
    });
};
```

When a client sends a valid token, their access will be successfully authenticated and consequently, the object `req.user` appears to be used inside the routes. This object is only created when the `auth.js` logic returns an authenticated user, that is, only when the following function returns a valid user:

``` javascript
// Do you remember this function from auth.js?
Users.findById(payload.id)
  .then(user => {
    if (user) {
      return done(null, {
        id: user.id,
        email: user.email
      });
    }
    return done(null, false);
  })
  .catch(error => done(error, null));
```

The `done()` callback send the authenticated user's data to the authenticated routes receive these data via `req.user` object. In our case, this object only has the `id` and `email` attributes.

To ensure the proper access to tasks resources, let's do a refactoring on all Sequelize functions from the routes `/tasks` and `/tasks/:id` to their queries use as parameter the `req.user.id`. To do this, edit the `routes/tasks.js` and, in the routes `app.route("/tasks")`, do the following modifications:

``` javascript
app.route("/tasks")
  .all(app.auth.authenticate())
  .get((req, res) => {
    Tasks.findAll({
      where: { user_id: req.user.id }
    })
    .then(result => res.json(result))
    .catch(error => {
      res.status(412).json({msg: error.message});
    });
  })
  .post((req, res) => {
    req.body.user_id = req.user.id;
    Tasks.create(req.body)
      .then(result => res.json(result))
      .catch(error => {
        res.status(412).json({msg: error.message});
      });
  });
```

In the same file, do the same modification inside the queries of the  `app.route("/tasks/:id")` function:

``` javascript
app.route("/tasks/:id")
  .all(app.auth.authenticate())
  .get((req, res) => {
    Tasks.findOne({ where: {
      id: req.params.id,
      user_id: req.user.id
    }})
    .then(result => {
      if (result) {
        return res.json(result);
      }
      return res.sendStatus(404);
    })
    .catch(error => {
      res.status(412).json({msg: error.message});
    });
  })
  .put((req, res) => {
    Tasks.update(req.body, { where: {
      id: req.params.id,
      user_id: req.user.id
    }})
    .then(result => res.sendStatus(204))
    .catch(error => {
      res.status(412).json({msg: error.message});
    });
  })
  .delete((req, res) => {
    Tasks.destroy({ where: {
      id: req.params.id,
      user_id: req.user.id
    }})
    .then(result => res.sendStatus(204))
    .catch(error => {
      res.status(412).json({msg: error.message});
    });
  });
```

To finish these refactoring, let's modify the users resources! Now we need to adapt some pieces of code inside the user's routes. Basically, we are going to change the way of how to search or delete an user to use the authenticated user's `id`.

In this case, won't be necessary to use an `id` in the route parameter, because now the route `/users/:id` will be just `/user` (in the singular, because we are now dealing with a single logged user). Only to search and delete will have an authentication middleware, so, now both of them can be grouped via `app.route("/user")` function to reuse the middleware `all(app.auth.authenticate())`. Instead of `req.params.id`, we are going to use `req.user.id`, to ensure that an authenticated user `id` will be used.

To understand better how this logic will works, edit the `routes/users.js` file and do the following modifications:

``` javascript
module.exports = app => {
  const Users = app.db.models.Users;

  app.route("/user")
    .all(app.auth.authenticate())
    .get((req, res) => {
      Users.findById(req.user.id, {
        attributes: ["id", "name", "email"]
      })
      .then(result => res.json(result))
      .catch(error => {
        res.status(412).json({msg: error.message});
      });
    })
    .delete((req, res) => {
      Users.destroy({where: {id: req.user.id} })
        .then(result => res.sendStatus(204))
        .catch(error => {
          res.status(412).json({msg: error.message});
        });
    });

  app.post("/users", (req, res) => {
    Users.create(req.body)
      .then(result => res.json(result))
      .catch(error => {
        res.status(412).json({msg: error.message});
      });
  });
};
```

### Conclusion

Congratulations! We have finished an extremely important step of the application. This time, all tasks will be returned only by an authenticated user. Thanks to JWT, now we have a safe mechanism for user's authentication between client and server.

Up to now, it was implemented all the application back-end and we did not create a client application that uses all the power of our API. But chill out, there are a lots of surprises in the next chapters that are going to get you excited, just keep reading!
