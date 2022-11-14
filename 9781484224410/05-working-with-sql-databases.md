# Working with SQL databases

## Introduction to SQLite3 and Sequelize

In the last chapter, we created simples routes for the application to list tasks via static data model. That was enough to start and explore some basic concepts about API resources.

Now, we are going to work deeper on using a relational database, which is going to be used to create a dynamic management of tasks and users. To simplify our examples, we are going to use **SQLite3**. It is preinstalled database on Linux, Unix and MacOSX, so there’s no need to set it up. However, in case you are using Windows, you can easily install it following the instructions on the link bellow:

https://www.sqlite.org/download.htm

![SQLite3 logo](images/sqlite-logo.png)

SQLite3 is a database that keeps all the data in a `.sqlite` extension's file. It has an interface based on SQL language very similar to other databases and is present not only on desktop systems but on mobile applications as well.

On Node.js, there are several frameworks that works with **SQLite3**. On our application, we'll use the **Sequelize.js** module, which is a complete module and has a nice interface and is very easy to work. On it, it is possible to manipulate data using SQL commands (or not) and it also supports the main SQL databases, such as: **Postgres, MariaDB, MySQL, SQL Server and SQLite3.**

![Sequelize.js logo](images/sequelize-logo.jpg)

Sequelize is an ORM *(Object Relational Mapper)* for Node.js. It's **Promises-based** framework. Currently, it is on **3.19.0 version** and has a lot of features like transactions, modeling tables, tables relationship, database replication for reading mode and more.

![Sequelize.js homepage](images/site-sequelize.jpg)

The read the full documentation take a look this link:

http://sequelizejs.com

## Setting up Sequelize

To install Sequelize and SQLite3 wrapper, execute on the terminal this command:

``` bash
npm install sequelize@3.19.2 sqlite3@3.1.1 --save
```

With these two modules installed, we already have the dependencies that we need for the database connection with the API. Now, let's create a connection settings file between Sequelize and SQLite3. To do it, create the file `libs/config.js` using these parameters:

* `database`: defines the database name;
* `username`: informs the username of access;
* `password`: informs the username’s password;
* `params.dialect`: informs which database will be used (`sqlite`, `mysql`, `postgres`, `mariadb`, `mssql`);
* `params.storage`: is a specific attribute for only SQLite3, it defines the directory where the database files will be recorded;
* `params.define.underscored`: standardize the tables fields' name in lowercase letters with underscore;

See bellow how the file is going to look like:

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
  }
};
```

After create this settings file, we're going create a code, which will be responsible to connects with the database. This connection code will adopt the **Singleton pattern** to ensure that Sequelize connection will be instantiated only once. This is going to allow us to load this module innumerous time via a single database's connection.

To do this, create the `db.js` code following the step below:

``` javascript
import Sequelize from "sequelize";
const config = require("./libs/config.js");
let sequelize = null;

module.exports = () => {
  if (!sequelize) {
    sequelize = new Sequelize(
      config.database,
      config.username,
      config.password,
      config.params
    );
  }
  return sequelize;
};
```

Done! To start this connection module, let's include it inside `consign()` function. Now the `db.js` will be the first module to be loaded. To do this, edit `index.js`:

``` javascript
import express from "express";
import consign from "consign";

const app = express();

consign()
  .include("db.js")
  .then("models")
  .then("libs/middlewares.js")
  .then("routes")
  .then("libs/boot.js")
  .into(app);
```

To finish the Sequelize.js setup, let's implement a sync simple function between Sequelize and database. This sync function performs, if necessary, alterations on database tables, according to what is going to be setup on the models. Let's include the `app.db.sync()` function, to ensure this action will be executed before the server starts. This refactoring needs to be written into `libs/boot.js` file:

``` javascript
module.exports = app => {
  app.db.sync().done(() => {
    app.listen(app.get("port"), () => {
      console.log(`NTask API - Port ${app.get("port")}`);
    });
  });
};
```

To test those modifications, restart the server. If everything is ok, your application must work as it was working before, after all, no visible modification was made, only some adjustments were made to establish a connection between the API and SQLite3. In the next section, we'll create all necessary models for our application, and this is gonna be a huge impact for our project.

## Creating models

Up to this point, the only model in the API is the `model/tasks.js`, which is static model and is not connected with any database. In this section, we are going to explore the main Sequelize features to create models that performs as tables in the database and as resources in the API.

Our application will have two simple models: `Users` and `Tasks`. The relationship between them will be **Users 1-N Tasks**, similar to the image below:

![NTask models relationship](images/modelagem-users-tasks.png)

To work with this kind of relationship, will be used the Sequelize's functions: `Users.hasMany(Tasks)` (on `models/users.js`) and `Tasks.belongsTo(Users)` (on `models/tasks.js`). These associations will be encapsulated within a model's attribute called `classMethods`, which allows us to include static functions to the model. In our case, we are going to create the `associate` function within the `classMethods` of each model.

### Model: Tasks

To start off, let's modify the `models/tasks.js` file, applying the following code:

``` javascript
module.exports = (sequelize, DataType) => {
  const Tasks = sequelize.define("Tasks", {
    id: {
      type: DataType.INTEGER,
      primaryKey: true,
      autoIncrement: true
    },
    title: {
      type: DataType.STRING,
      allowNull: false,
      validate: {
        notEmpty: true
      }
    },
    done: {
      type: DataType.BOOLEAN,
      allowNull: false,
      defaultValue: false
    }
  }, {
    classMethods: {
      associate: (models) => {
        Tasks.belongsTo(models.Users);
      }
    }
  });
  return Tasks;
};
```

The function `sequelize.define("Tasks")` is responsible to create or change a table. This happens only when Sequelize syncs with the application during the boot time (in our case, this happens on `libs/boot.js` file via `app.db.sync()` function). The second parameter is an object, their attributes respectively represent the fields of a table, and their values are sub attributes whose describes the data type their fields

On this model, the field `id` is an integer via `DataType.INTEGER. It represents a primary key (`primaryKey: true`) and its value is auto incremented (`autoIncrement: true`) on each new record.

The field `title` is a string (`DataType.STRING`). This field has the attribute `allowNull: false` to block null values and is a validator field as well, which verifies if the string is not empty (`validate.notEmpty: true`). The field `done` is a boolean (`DataType.BOOLEAN`) which doesn't allow null values (`allowNull: false`). By the way, if a value is not informed in this field, it will be registered as `false` via `defaultValue: false` attribute.

At last, we have a third parameter, which allows including static functions within attribute `classMethods`. The `associate(models)` function was created to allow models' relationships. In this case, the relationship was established via `Tasks.belongsTo(models.Users)` function, cause this is a **Tasks 1-N Users** relationship.

**WARNING:** The model `Users` wasn't created yet. So, if you restart the server right now an error will happen. Stay calm, keep reading and coding until the end of this chapter to see the result in the end.

### Model: Users

To complete our database modeling, let's create a model that will represent the users of our application. You need to create the `models/users.js` file with the following model definitions:

``` javascript
module.exports = (sequelize, DataType) => {
  const Users = sequelize.define("Users", {
    id: {
      type: DataType.INTEGER,
      primaryKey: true,
      autoIncrement: true
    },
    name: {
      type: DataType.STRING,
      allowNull: false,
      validate: {
        notEmpty: true
      }
    },
    password: {
      type: DataType.STRING,
      allowNull: false,
      validate: {
        notEmpty: true
      }
    },
    email: {
      type: DataType.STRING,
      unique: true,
      allowNull: false,
      validate: {
        notEmpty: true
      }
    }
  }, {
    classMethods: {
      associate: (models) => {
        Users.hasMany(models.Tasks);
      }
    }
  });
  return Users;
};
```

This time, the modeling of the table `Users` was very similar to `Tasks`. The only difference was the inclusion of the `unique: true` attribute, inside the field `email`, to ensure that repeated emails won't be allowed.

After finishing this step, we are going to change some codes in the project, so it can load these models correctly and run their respective relationship functions in the database. To start off, let's modify some of the modules loading on `index.js`. First, we are going to order the loading of the modules to the `libs/config.js` be loaded first, then the `db.js` and also remove the loading of the `models` directory from `consign`, because now all models are loaded from `db.js`. See how the code must look like:

``` javascript
import express from "express";
import consign from "consign";

const app = express();

consign()
  .include("libs/config.js")
  .then("db.js")
  .then("libs/middlewares.js")
  .then("routes")
  .then("libs/boot.js")
  .into(app);
```

The reason why the directory `models` was deleted from `consign()` is that now, all models will be loaded directly by `db.js` file, via `sequelize.import()` function. After all, if you go back to the models codes you will notice that two new attributes within `module.exports = (sequelize, DataType)` appeared. Those are going to be magically injected via `sequelize.import()`, which is responsible for loading and defining the models. So, let's refactory the `db.js` file, to import and load all models correctly:

``` javascript
import fs from "fs";
import path from "path";
import Sequelize from "sequelize";

let db = null;

module.exports = app => {
  if (!db) {
    const config = app.libs.config;
    const sequelize = new Sequelize(
      config.database,
      config.username,
      config.password,
      config.params
    );
    db = {
      sequelize,
      Sequelize,
      models: {}
    };
    const dir = path.join(__dirname, "models");
    fs.readdirSync(dir).forEach(file => {
      const modelDir = path.join(dir, file);
      const model = sequelize.import(modelDir);
      db.models[model.name] = model;
    });
    Object.keys(db.models).forEach(key => {
      db.models[key].associate(db.models);
    });
  }
  return db;
};
```

This time, the code got a little complex, didn’t it? But, its features are pretty cool! Now we can use the database settings via `app.libs.config` object. Another detail is in the execution of the nested function `fs.readdirSync(dir).forEach(file)` which basically returns a array of strings referring to the files names from the directory `models`. Then, this array is iterated to import and load all models using the `sequelize.import(modelDir)` function and, then, inserted in this module inside the structure `db.models` via `db.models[model.name] = model`.

After loading all models, a new iteration happens through `Object.keys(db.models).forEach(key)` function, which basically executes the function `db.models[key].associate(db.models)` to ensure the models' relationship.

To finish the adjustments, we need to write a simple modification in the `libs/boot.js` file, changing from the `app.db.sync()` function to `app.db.sequelize.sync()`:

``` javascript
module.exports = app => {
  app.db.sequelize.sync().done(() => {
    app.listen(app.get("port"), () => {
      console.log(`NTask API - Port ${app.get("port")}`);
    });
  });
};
```

Then, let's modify the `routes/tasks.js` to load the `Tasks` model correctly using `app.db.models.Tasks` and modify the function `Tasks.findAll()` to his promises-based function provided by Sequelize. See below how it must look like:

``` javascript
module.exports = app => {
  const Tasks = app.db.models.Tasks;
  app.get("/tasks", (req, res) => {
    Tasks.findAll({}).then(tasks => {
      res.json({tasks: tasks});
    });
  });
};
```

### Conclusion

Finally we have finished the Sequelize.js adaptation into our API. To test it, just restart the server and go to the address:

http://localhost:3000/tasks

This time it is going to display a JSON object with empty tasks.

![Now, the tasks list is empty!](images/lista-tarefas-vazia.png)

But don’t worry, in the next chapter we'll implement the main endpoints to perform a complete CRUD in the API. So just keep reading!
