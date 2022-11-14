# Building an API

Now that we have the Node.js environment installed and ready to work, we are going to explore the creation of our REST API application. This kind of application is currently being developed by many projects and companies, because it brings as an advantage the creation of an application focused only on feeding any client-side application with data.
Nowadays, it's quite common to create web and or mobile apps which consumes data from an API or more than one APIs. This makes that several kinds of client applications consult the same server focused on deal with data. Besides, it also allows that each application – client or server – to be worked by different teams.

First, we are going to build an API, however, throughout to the last chapters, we will build a simple web client-side application to consume data from the our API. To start developing the API, we are going to use a very popular web framework, called Express.

## Introduction to Express

Express is a minimalist web framework that was highly inspired in Sinatra framework from Ruby language. With this module, you can create from small applications to large and complex ones. This framework allows you to build APIs and also to create simple web sites.

It's focused to work with `views`, `routes` and `controllers`, only `models` is not handled by this framework, giving you the freedom to use any persistence framework without creating any incompatibility or conflict in the application. This is a big advantage, because there're a lot of ODM *(Object Data Mapper)* and also ORM *(Object Relational Mapping)* available. You can use anyone with Express without problem, you just need to load this kind of module, write some models and use them inside the `controllers`, `routes` or `views`.

Express allows the developers freely arrange the project's code, that is, there're no inflexible conventions in this framework, each convention can be created and applied by yourself. This makes Express flexible to be used on both small and large applications, because not always it's necessary to apply a lot of conventions into a small project.

You can also reapply conventions from other frameworks. Ruby On Rails is an example of a framework full of conventions that are worth reapplying some of their technical features. This independence forces the developer to fully understand how each application structure works.

![expressjs.com](images/site-do-express.jpg)

To sum up, you can see a list of Express main features bellow:

* Robust routing;
* Easily integratable with a lot of template engines;
* Minimalist code;
* Works with middlewares concept;
* A huge list of 3rd-party middlewares to integrate;
* Content negotiation;
* Adopt standards and best practices of REST APIs;

## Getting started the pilot project

What about creating a project in practice? From this chapter on, we are going to explore some concepts on how to create a REST API using some Node.js frameworks.

Our application is going to be a simple task manager, this will be divided into two projects: API and Web app.

Our API will be called **NTask** (Node Task) and it'll have the following features:

* List of tasks;
* Create, delete and update a task;
* Create, delete and update a user data;
* User authentication;
* API documentation page;

> ##### Pilot project source code
> If you are curious to check out the source code of all projects that are going to be explored in this book, just access this link:

> [github.com/caio-ribeiro-pereira/building-apis-with-nodejs](https://github.com/caio-ribeiro-pereira/building-apis-with-nodejs)

In this application, we'll explore the main resources to create an REST API on Node.js and, throughout the chapters; we are going to include new important frameworks to help the developing.

To begin, let's create our first project named `ntask-api`, running these commands:

``` bash
mkdir ntask-api
cd ntask-api
npm init
```

You need to answer the `npm init` command wizard as you like or you can copy this result below:

![Describing the project using npm init](images/npm-init-ntask-api.png)

In the end, the `package.json` will be created with these attributes:

``` json
{
  "name": "ntask-api",
  "version": "1.0.0",
  "description": "Task list API",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Caio Ribeiro Pereira",
  "license": "ISC"
}
```

The currenty Node.js version does not support ES6 perfectly, but we can use a module that emulates some of the resources from ES6/ES7 to make our codes cooler. To do this, we are going to install the `babel`. It's JavaScript transpiler responsible to convert a ES6/ES7 codes to a ES5 code only when the JavaScript runtime doesn't recognize some ES6/ES7 features.

To use all features from ES6/ES7, we are going to install `babel-cli` and `babel-preset-es2015` modules in the project, running this command:


``` bash
npm install babel-cli@6.5.1 babel-preset-es2015@6.5.0 --save
```

Now you need to link the preset `babel-preset-es2015` to be recognized by the `babel-cli`, to do this, just create the file `.babelrc` with this simple code:

``` json
{
  "presets": ["es2015"]
}
```

After that, we'll to turbocharge the `package.json`. First we are going to remove the fields `license` and `scripts.test`. Then, include the field `scripts.start` to enable the alias command `npm start`. This command will be responsible to start our API server via Babel transpiler using the command `babel-node index.js`. See bellow how our `package.json` is going to look like:

``` json
{
  "name": "ntask-api",
  "version": "1.0.0",
  "description": "Task list API",
  "main": "index.js",
  "scripts": {
    "start": "babel-node index.js"
  },
  "author": "Caio Ribeiro Pereira",
  "dependencies": {
    "babel-cli": "^6.5.1",
    "babel-preset-es2015": "^6.5.0"
  }
}
```

Now we have a basic configuration and description about our project, and each information is inside the `package.json`. To start off, let's install the `express` framework:

``` bash
npm install express@4.13.4 --save
```

Installing Express, we'll create our first code. This code is going to load the `express` module, create a simple endpoint using the `GET /` via function `app.get("/")` and start the server in the port `3000` using the function `app.listen()`. To do this, create the `index.js` file using this code:

``` javascript
import express from "express";

const PORT = 3000;
const app = express();

app.get("/", (req, res) => res.json({status: "NTask API"}));

app.listen(PORT, () => console.log(`NTask API - Port ${PORT}`));
```

To test this code and especially check if the API is running, start the server running:

``` bash
npm start
```

Your application must display the following message on the terminal:

![Starting the API](images/start-api.png)

Then, open your browser and go to: [localhost:3000](http://localhost:3000)

If nothing goes wrong, a JSON content will be displayed, similar to the image bellow:

![JSON status message](images/home-api.png)

## Implementing a simple and static resource

RESTful APIs works with the concept of creates and manipulates **resources**. These resources are entities that are used for queries, entries, updating and deleting data, by the way, everything is based on manipulating data from resources.

For example, our application will have as the main resource the entity named `tasks`, which will be accessed by the endpoint `/tasks`. This entity is going to have some data that describes what kind of information can be persisted in this resource. This concept follows a same line as data modeling, the only difference is that the resources abstract the data source. So, a resource can return data from different sources, such as databases, static data or any data from external systems, it can return from another external API too.

An API aims to address and unify the data to, in the end, build and show a resource. Initially, we are going to work with static data, but throughout the book we will make some refactoring to integrate a database.

For while, static data will be implemented only to mold an endpoint. To mold our API, we are going to include a route via `app.get("/tasks")` function, which is going to return only a static JSON via `res.json()` function, which is responsible to render a json content as output. Here is how these modifications are going to look like in the `index.js` file:

``` javascript
import express from "express";

const PORT = 3000;
const app = express();

app.get("/", (req, res) => res.json({status: "NTask API"}));

app.get("/tasks", (req, res) => {
  res.json({
    tasks: [
      {title: "Buy some shoes"},
      {title: "Fix notebook"}
    ]
  });
});

app.listen(PORT, () => console.log(`NTask API - Port ${PORT}`));
```

To test this new endpoint, restart the application typing **CTRL+C** or **Control+C** (if you are a MacOSX user) and then, run `npm start` again.

**Attention:** Always follow this same step above to restart the application correctly.

Now we have a new endpoint available to access via the address: [localhost:3000/tasks](http://localhost:3000/tasks)

Once you open it, you will have the following result:

![Listing tasks](images/listando-tarefas.png)

In case you want that your results to return as a formatted and tabbed JSON output, include in the `index.js` the following configuration: `app.set("json spaces", 4);`. See below where to do it:

``` javascript
import express from "express";

const PORT = 3000;
const app = express();

app.set("json spaces", 4);

app.get("/", (req, res) => res.json({status: "NTask API"}));

app.get("/tasks", (req, res) => {
  res.json({
    tasks: [
      {title: "Buy some shoes"},
      {title: "Fix notebook"}
    ]
  });
});

app.listen(PORT, () => console.log(`NTask API - Port ${PORT}`));
```
Now, restart the server and see a result more elegant:

![Listing tasks with formatted JSON](images/listando-tarefas-json-formatado.png)

## Arranging the loading of modules

Indeed, write all endpoints into `index.js` won’t be a smart move, especially if your application has a lot of them. So, let's arrange the directories and the loading of all codes according to their responsibilities.

We are going to apply the MVR *(Model-View-Router)* pattern to arrange this things. To do it, we'll use the `consign` module, which will allow our project to autoload models, routers, middlewares, configs and more, this module injects dependencies easily. So, let's install!

``` bash
npm install consign@0.1.2 --save
```

With this new module installed, let's migrate the endpoints from `index.js` file creating two new files into the new directory named `routes`. To do this, create the file `routes/index.js`:

``` javascript
module.exports = app => {
  app.get("/", (req, res) => {
    res.json({status: "NTask API"});
  });
};
```

Move the endpoint function `app.get("/tasks")` from `index.js` to his new file `routes/tasks.js`:

``` javascript
module.exports = app => {
  app.get("/tasks", (req, res) => {
    res.json({
      tasks: [
        {title: "Buy some shoes"},
        {title: "Fix notebook"}
      ]
    });
  });
};
```

To finish this step, edit the `index.js` to be able to load those routes via `consign` module and start the server:

``` javascript
import express from "express";
import consign from "consign";

const PORT = 3000;
const app = express();

app.set("json spaces", 4);

consign()
  .include("routes")
  .into(app);

app.listen(PORT, () => console.log(`NTask API - Port ${PORT}`));
```

That’s it, we have just arranged the loading of all routes. Note that, at this point we are only focusing on working with **VR** (view and router) from **MVR** pattern. In our case, the JSON outputs are considered as **views** which are provided by the **routes**. The next step will be arranging the **models**.

Let's create the `models` directory and, back to `index.js` add one more `include()` function inside the `consign()` to allow the loading of the `models` before the `routes`. To make this modification clearer, edit the `index.js`:

``` javascript
import express from "express";
import consign from "consign";

const PORT = 3000;
const app = express();

app.set("json spaces", 4);

consign()
  .include("models")
  .then("routes")
  .into(app);

app.listen(PORT, () => console.log(`NTask API - Port ${PORT}`));
```

In this moment, the `consign()` function won’t load any model, because the directory `models` doesn't exists. To fill this gap, let's temporarily create a model with static data, just to finish this step. To do this, create the file `models/tasks.js` and filling with these codes:

``` javascript
module.exports = app => {
  return {
    findAll: (params, callback) => {
      return callback([
        {title: "Buy some shoes"},
        {title: "Fix notebook"}
      ]);
    }
  };
};
```

Initially, this model is going to only have the function `Tasks.findAll()`, which is going to receive two arguments: `params` and `callback`. The variable `params` will not be used in this moment, but it serves as a base to send some SQL query filters, something that we are going to talk about in details in the next chapters. The second argument is the callback function, which returns asynchronously a static array of tasks.

To call it in the `routes/tasks.js` file, you are going to load this module via `app` variable. After all, the modules added by `consign()` function are injected into a main variable, in our case, injected into `app` via `consign().into(app);`. To see how to use a loaded module, in this case, we'll use the `app.models.tasks` module, edit the `routes/tasks.js` following this code:

``` javascript
module.exports = app => {
  const Tasks = app.models.tasks;
  app.get("/tasks", (req, res) => {
    Tasks.findAll({}, (tasks) => {
      res.json({tasks: tasks});
    });
  });
};
```
Note that the function `Tasks.findAll()` has in the first parameter an empty object. This is the value of the `params` variable, which in this case, there is no need to include parameters to filter the tasks list.

The callback of this function returns the `tasks` variable, which was created to return, for while, some static values from his model. So, at this point, we are sure that `tasks` is going to return an static array with some tasks descriptions.

To finish these refactorings, let's create a file, which is going to load all the middlewares and specific settings of Express. Currently, we only have a simple configuration about JSON's format, which occurs via function `app.set("json spaces", 4)`. But, we are going to include another settings, in this case, it will be the server port calling the `app.set("port", 3000)` function.

Throughout this book, we are going to explore a lot of new middlewares and settings to customize our API server. So, I already recommend to prepare the house for new visitors.

Create the file `libs/middlewares.js` and write theses codes below:

``` javascript
module.exports = app => {
  app.set("port", 3000);
  app.set("json spaces", 4);
};
```

To simplify the server boot, create the file `libs/boot.js` that is going to be responsible for the server initialization via `app.listen()` function, this time, we'll use `app.get("port")` instead the old constant `PORT`, see the code below:

``` javascript
module.exports = app => {
  app.listen(app.get("port"), () => {
    console.log(`NTask API - Port ${app.get("port")}`);
  });
};
```

To finish this chapter, let's load the `libs/boot.js` inside the module's structure provided by `consign` and remove the old `const PORT`, `app.set()` and `app.listen()` declarations to simplify this main file. To do this, just edit the `index.js`:

``` javascript
import express from "express";
import consign from "consign";

const app = express();

consign()
  .include("models")
  .then("libs/middlewares.js")
  .then("routes")
  .then("libs/boot.js")
  .into(app);
```

To test everything, restart the server and access the endpoint again: [localhost:3000/tasks](http://localhost:3000/tasks)

![Listing loaded modules](images/listando-modulos-carregados.png)

To make sure everything is ok, no error should occur and all the tasks data must be displayed normally.

### Conclusion

Mission complete! In the next chapter we'll expand our project implementing new functions to manage tasks dynamically using a database and Sequelize.js framework.
