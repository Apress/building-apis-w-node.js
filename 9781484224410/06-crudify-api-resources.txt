# CRUDify API resources

In this chapter, we'll explore more the usage of the new functions from Sequelize and also organize the API's routes and some middlewares of Express. The CRUD *(Create, Read, Update, Delete)* will be built using the models: `Tasks` and `Users`.

## Organizing tasks routes

To start this refactoring, let's explore the main HTTP methods to CRUDify our application. In this case, we are going to use the functions `app.route("/tasks")` and `app.route("/tasks/:id")` to define the path: `"/tasks"` and `"/tasks/(task_id)"`.

These functions allow to use nested functions, reusing the routes through all HTTP methods provided by these Express functions below:

* `app.all()`: is a middleware that is executed via any HTTP method call;
* `app.get()`: executes the method `GET` from HTTP, it is used to search and get some set of data from one or multiple resources;
* `app.post()`: executes the method `POST` from HTTP, it is semantically used to send and persist new data into a resource;
* `app.put()`: executes the method `PUT` from HTTP, very used to update data of a resource;
* `app.patch()`: executes the method `PATCH` do HTTP, it has similar semantics to `PUT` method, but is recommended only to update some attributes of a resource, and not the entire data of a resource;
* `app.delete()`: executes the method `DELETE` from HTTP, as his name implies to delete a particular data of a resource.

To better understand the use of these routes, we are going to edit the `routes/tasks.js`, applying the needed functions to CRUDify the tasks resource.

``` javascript
module.exports = app => {
  const Tasks = app.db.models.Tasks;

  app.route("/tasks")
    .all((req, res) => {
      // Middleware for pre-execution of routes
    })
    .get((req, res) => {
      // "/tasks": List tasks
    })
    .post((req, res) => {
      // "/tasks": Save new task
    });

  app.route("/tasks/:id")
    .all((req, res) => {
      // Middleware for pre-execution of routes
    })
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

With this basic structure, we already have an outline of the needed routes to manage tasks. Now, we can implement their logics to correctly deal with each action of our tasks CRUD.

On both endpoints (`"/tasks"` e `"/tasks/:id"`) we are going to deal with some rules inside their middlewares via `app.all()` function to avoid some invalid access when if somebody sends the task `id` inside request's body. This is going to be a simple modification, see below how must look like:

``` javascript
app.route("/tasks")
  .all((req, res, next) => {
    delete req.body.id;
    next();
  })
  // Continuation of routes...

app.route("/tasks/:id")
  .all((req, res, next) => {
    delete req.body.id;
    next();
  })
  // Continuation of routes...
```

We are practically ensuring the exclusion of the `id` attribute within the request's body, via `delete req.body.id`. This happens because, on each request functions, we are going to use `req.body` as a parameter for Sequelize.js functions, and the attribute `req.body.id` could overwrite the `id` of a task, for example, on `update` or `create` a task.

To finish the middleware, warning that it should perform a corresponding function to an HTTP method, you just need to include in the end of callback the `next()` function to warn the Express router that now it can execute the next function on the route or the next middleware below.

## Listing tasks via GET

We already have a basic middleware for treating all the task's resources, now let's implement some CRUD functions, step-by-step. To start off, we'll the `app.get()` function, which is going to list all data from `tasks` models, running the `Tasks.findAll()` function:

``` javascript
app.route("/tasks")
  .all((req, res, next) => {
    delete req.body.id;
    next();
  })
  .get((req, res) => {
    Tasks.findAll({})
      .then(result => res.json(result))
      .catch(error => {
        res.status(412).json({msg: error.message});
      });
  })
```

In this first implementation, we are listing all the tasks running: `Tasks.findAll({})`. Although it is a bad practice to list all data, we are only using it for didactic matters. But chill out that throughout the chapters some arguments will be implemented to make this list of tasks more specific. The search results occurs via `then()` function and if any problem happens, you can handle them via `catch()` function. An important detail about API's development is to treat all or the most important results using the correct HTTP status code, in this case a successful result will return by default the `200 - OK` status. On our API, to simplify things a lot of common errors will use the `412 - Precondition Failed` status code.

> ##### About HTTP status
> There is no rules to follow in the HTTP status definition; however, it's advisable to understand the meaning of the each status to make your server's response more semantic. To learn more about some HTTP status code, take a look this website:

> [www.w3.org/Protocols/rfc2616/rfc2616-sec10.html](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)

## Creating tasks via POST

There is no secret in the implementation of POST method. Just use the `Tasks.create(req.body)` function and then treat their promises callbacks: `then()` and `catch()`.

One of the most interesting things on Sequelize is the sanitization of parameters you send to a model. This is very good, because if the `req.body` have some attributes not defined in the model, they will be removed before inserts or updates a model. The only problem is the `req.body.id` that could modify the `id` of a model. However, this was already handled by us when we wrote a simple rules into `app.all()` middleware. The implementation of this route will be like this piece of code:

``` javascript
app.route("/tasks")
  .all((req, res, next) => {
    delete req.body.id;
    next();
  })
  .get((req, res) => {
    Tasks.findAll({})
      .then(result => res.json(result))
      .catch(error => {
        res.status(412).json({msg: error.message});
      });
  })
  .post((req, res) => {
    Tasks.create(req.body)
      .then(result => res.json(result))
      .catch(error => {
        res.status(412).json({msg: error.message});
      });
  });
```

## Finding a task via GET

Now we are gonna to treat the `/tasks/:id` endpoints. To do this, let's start by the implementation of the `app.route("/tasks/:id")` function, which have the same middleware logic as the last `app.all()` function.

To finish, we'll use the function `Tasks.findOne({where: req.params})`, which executes, for example, `Tasks.findOne({where: {id: "1"}})`. It does an single search of tasks based on the task `id`, in case there isn't a task, the API will respond the `404 - Not Found` status code via `res.sendStatus(404)` function. See how it must look like:

``` javascript
app.route("/tasks/:id")
  .all((req, res, next) => {
    delete req.body.id;
    next();
  })
  .get((req, res) => {
    Tasks.findOne({where: req.params})
      .then(result => {
        if (result) {
          res.json(result);
        } else {
          res.sendStatus(404);
        }
      })
      .catch(error => {
        res.status(412).json({msg: error.message});
      });
  })
```

## Updating a task via PUT

Now we are going to implement a function to update a task in the database. To do this, there is no secret, just use the `Task.update()` function which the first parameter you have included an updated object and, in the second one, an object with parameters to find the current task to be updated. These functions will return a simple array with a number of changes made. But this information won't be very useful as response, so to simplify things, let's force the `204 - No Content` status code using the `res.sendStatus(204)` function, which means that the request was successful done and no content will be returned in the response. See below how this implementation looks like:

``` javascript
app.route("/tasks/:id")
  .all((req, res, next) => {
    delete req.body.id;
    next();
  })
  .get((req, res) => {
    Tasks.findOne({where: req.params})
      .then(result => {
        if (result) {
          res.json(result);
        } else {
          res.sendStatus(404);
        }
      })
      .catch(error => {
        res.status(412).json({msg: error.message});
      });
  })
  .put((req, res) => {
    Tasks.update(req.body, {where: req.params})
      .then(result => res.sendStatus(204))
      .catch(error => {
        res.status(412).json({msg: error.message});
      });
  })
```

Just like the `Tasks.create()` function, the `Tasks.update` cleans the fields that are not on its own model, so there is no problem to send `req.body` directly as parameter.

## Deleting a task via DELETE

To finish, we have to implement a route to delete a task, and once more, there is no secret here. You just have to use the `Tasks.destroy()` function which uses as arguments an object containing data to find and delete a task. In our case, we'll use `req.params.id` to remove a single task and, as a successful response, uses `204 - No Content` as status code via `res.sendStatus(204)` function:

``` javascript
app.route("/tasks/:id")
  .all((req, res, next) => {
    delete req.body.id;
    next();
  })
  .get((req, res) => {
    Tasks.findOne({where: req.params})
      .then(result => {
        if (result) {
          res.json(result);
        } else {
          res.sendStatus(404);
        }
      })
      .catch(error => {
        res.status(412).json({msg: error.message});
      });
  })
  .put((req, res) => {
    Tasks.update(req.body, {where: req.params})
      .then(result => res.sendStatus(204))
      .catch(error => {
        res.status(412).json({msg: error.message});
      });
  })
  .delete((req, res) => {
    Tasks.destroy({where: req.params})
      .then(result => res.sendStatus(204))
      .catch(error => {
        res.status(412).json({msg: error.message});
      });
  });
```

So, we finish to *CRUDify* the task's resources of our API!

## Refactoring some middlewares

To avoid code duplication, we'll apply a simple refactoring migrating the business logic used inside `app.all()` function to an Express middleware, using the `app.use()` function, into `libs/middlewares.js` file. To apply this refactoring, first, remove the functions `app.all()` from `routes/tasks.js` file, leaving the code with the following structure:

``` javascript
module.exports = app => {
  const Tasks = app.db.models.Tasks;

  app.route("/tasks")
    .get((req, res) => {
      // GET /tasks callback...
    })
    .post((req, res) => {
      // POST /tasks callback...
    });

  app.route("/tasks/:id")
    .get((req, res) => {
      // GET /tasks/id callback...
    })
    .put((req, res) => {
      // PUT /tasks/id callback...
    })
    .delete((req, res) => {
      // DELETE /tasks/id callback...
    });
};
```

To enable a JSON parse inside all API's routes, we must install the `body-parser` module, you do it running:

``` bash
npm install body-parser@1.15.0 --save
```

After this installation, open and edit `libs/middlewares.js` file to insert the `bodyParser.json()` function as middleware, and in the end, include the middleware exclusion logic using the `app.use()` callback, following the code below:

``` javascript
import bodyParser from "body-parser";

module.exports = app => {
  app.set("port", 3000);
  app.set("json spaces", 4);
  app.use(bodyParser.json());
  app.use((req, res, next) => {
    delete req.body.id;
    next();
  });
};
```

## Creating user's endpoints

We also have to create routes to manage users in the application, after all, without them, it becomes impossible to manage tasks, am I right?

Our CRUD of users is not going to have anything new. In fact, won't be a complete CRUD, because we just need to create, find and delete an user, it won't be necessary to include the `app.route()` function too. Each route will be directly called by his correspondent HTTP method. The code will follow a similar treatment as task's routes has. The only new thing that will be used is the `attributes: ["id", "name", "email"]` inside the `Users.findById()` function. This parameter allows to return some selected fields from a model's result instead the full data of a model, it's similar to run the SQL query:

``` sql
SELECT id, name, email FROM users;
```

To code the user's routes, creates the `routes/users.js` file, using the code below:

``` javascript
module.exports = app => {
  const Users = app.db.models.Users;

  app.get("/users/:id", (req, res) => {
    Users.findById(req.params.id, {
      attributes: ["id", "name", "email"]
    })
    .then(result => res.json(result))
    .catch(error => {
      res.status(412).json({msg: error.message});
    });
  });

  app.delete("/users/:id", (req, res) => {
    Users.destroy({where: {id: req.params.id} })
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

**Attention:** The reason why we're not using the function `app.route()` here is because in the next chapter, we are going to modify some specific points on each user's route, to find or delete only if the user is logged in the API.

## Testing endpoint's access using Postman

To test these modifications, restart the application, open the browser and use some REST client application, because it will be necessary to test the `POST`, `PUT` and `DELETE` HTTP's methods. To simplify, I recommend you to use Postman, which is a useful Google Chrome extension and it's easy to use.

To install it go to: [getpostman.com](https://www.getpostman.com)

Then, after the installation, open the **Chrome Apps page** and click on the **Postman icon**.

![Postman Rest Client app](images/aplicativo-postman.png)

A page to log in is going to be displayed, but you do not need to log in to use it. To skip this step and go straight to the main page, click on **Go to the app**:

![Opening Postman](images/acessando-postman.png)

To test the endpoints, let's perform the following tests:

* Choose the method `POST` via address [http://localhost:3000/tasks](http://localhost:3000/tasks);
* Click on the menu **Body**, choose the option **raw**, and change the format from **Text** to **JSON (application/json)**;
* Create the JSON: `{"title": "Sleep"}` and click on the button **Send**;
* Modify the same JSON to `{"title": "Study"}` and click again on **Send**;

![Registering task: "Sleep"](images/postman-post-tarefa-1.png)

![Registering task: "Study"](images/postman-post-tarefa-2.png)

With this procedure, you created two tasks, and now, you can explore the other task's routes. To test them, you can follow this simple list:

* Method `GET`, route [http://localhost:3000/tasks](http://localhost:3000/tasks);
* Method `GET`, route [http://localhost:3000/tasks/1](http://localhost:3000/tasks/1);
* Method `GET`, route [http://localhost:3000/tasks/2](http://localhost:3000/tasks/2);
* Method `PUT`, route [http://localhost:3000/tasks/1](http://localhost:3000/tasks/1) using the body: `{"title": "Work"}`;
* Method `DELETE`, route [http://localhost:3000/tasks/2](http://localhost:3000/tasks/2);

You can also test the user's endpoints, if you like.

### Conclusion

Congratulations! If you reached at this point and alive, you have an outline of a RESTful API, which means, that now it's possible to build a client-side app to consume the tasks or users resources. Don't stop reading! Because in the next chapter we'll write some important things to make sure that our users will be authenticated by the API. See ya!
