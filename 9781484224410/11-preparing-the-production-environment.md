# Preparing the production environment

## Introduction to CORS

In case you do not know, CORS *(Cross-origin resource sharing)* is an important HTTP mechanism. It is responsible for allowing or not asynchronous requests from other domains.

CORS, in practice, are only the HTTP's headers that are included on server-side. Those headers can inform which domain can consume the API, which HTTP methods are allowed and, mainly, which endpoints can be shared publicly to applications from other domains consume. Generally when implementing CORS in a browser, each *cross-origin* request will be preceded by an **OPTIONS** request that checks whether the GET or POST will be OK.

## Enabling CORS in the API

As we are developing an API that will serve data for any kind of client-side applications, we need to enable the CORS's middleware for the endpoints become public. Meaning that any client can make requests on our API. So for security reason, let's install and use the module `cors`, to limit which resources a client can gain access to and from another domain:

``` bash
npm install cors@2.7.1 --save
```

Then, to initiate it, just use the middleware `app.use(cors())` in the `libs/middlewares.js`:

``` javascript
import bodyParser from "body-parser";
import express from "express";
import cors from "cors";

module.exports = app => {
  app.set("port", 3000);
  app.set("json spaces", 4);
  app.use(cors());
  app.use(bodyParser.json());
  app.use(app.auth.initialize());
  app.use((req, res, next) => {
    delete req.body.id;
    next();
  });
  app.use(express.static("public"));
};
```

When using only the function `cors()` the middleware will release full access of our API. However, it is advisable to control which client domains can have access and which methods they can use, and, mainly, which headers must be required to the clients inform in the request. In our case, let's setup only three attributes: `origin` (allowed domains), `methods` (allowed methods) e `allowedHeaders` (requested headers).

In the `libs/middlewares.js`, let's add these parameters in `app.use(cors())` function:

``` javascript
import bodyParser from "body-parser";
import express from "express";
import cors from "cors";

module.exports = app => {
  app.set("port", 3000);
  app.set("json spaces", 4);
  app.use(cors({
    origin: ["http://localhost:3001"],
    methods: ["GET", "POST", "PUT", "DELETE"],
    allowedHeaders: ["Content-Type", "Authorization"]
  }));
  app.use(bodyParser.json());
  app.use(app.auth.initialize());
  app.use((req, res, next) => {
    delete req.body.id;
    next();
  });
  app.use(express.static("public"));
};
```

Now we have an API that is only going to allow client apps from the address: [localhost:3001](http://localhost:3001).

Keep calm, because this will be the domain of our client application that we'll build in the next chapters.

> ##### A bit more about CORS
> For study purposes about CORS, to understand its headers and, most important, to learn how to customize the rule for your API, I recommend you to read this full documentation:
> [https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)

## Generating logs

In this section, we are going to setup our application to report and generate logs files about the user's requests. To do this, let's use the module `winston` that is useful in treat several kinds of logs.

In our case, the logs requests will be treated by the module `morgan` which is a middleware for generating request's logs in the server. We also will get logs for any SQL command from database to generate a full log in the API. First, install the modules `winston` and `morgan`.

``` bash
npm install winston@2.1.1 morgan@1.6.1 --save
```

After that, let's write the code responsible to setup and load the `winston` module. We are going to verify if the logs' folder exist, using the native module `fs` *(File System)* to treat files.

Then, will be implemented a simple conditional via function `fs.existsSync("logs")`, to check if there is or not a `logs` folder. If it doesn't exist, it will be created by `fs.mkdirSync("logs")` function.

After this verification, we'll load and export the `module.exports = new winston.Logger` to be able to generate logs files. The logs object will use as `transports` the object `new winston.transports.File`, which is responsible for creating and maintaining several recent logs files. Create the file `libs/logger.js` like this:

``` javascript
import fs from "fs";
import winston from "winston";

if (!fs.existsSync("logs")) {
  fs.mkdirSync("logs");
}

module.exports = new winston.Logger({
  transports: [
    new winston.transports.File({
      level: "info",
      filename: "logs/app.log",
      maxsize: 1048576,
      maxFiles: 10,
      colorize: false
    })
  ]
});
```

Now, we are going to use our `libs/logger.js` for two important aspect of our application. First, to generate SQL logs commands modifying the file `libs/config.development.js` to load our module `logger`. We'll use its function `logger.info()` as a callback of the attribute `logging` from Sequelize to capture the SQL command via `logger.info()` function. To do this, just edit the `libs/config.development.js` as below:

``` javascript
import logger from "./logger.js";

module.exports = {
  database: "ntask",
  username: "",
  password: "",
  params: {
    dialect: "sqlite",
    storage: "ntask.sqlite",
    logging: (sql) => {
      logger.info(`[${new Date()}] ${sql}`);
    },
    define: {
      underscored: true
    }
  },
  jwtSecret: "Nta$K-AP1",
  jwtSession: {session: false}
};
```

To finish, we are going to use the module `logger` to generate the request's logs. To do this, let's use the module `morgan` and include in the top of the middlewares the function `app.use(morgan("common"))` to log all requests.

To send these logs to our module `logger`, add the attribute `stream` as a callback function called `write(message)` and, then, send the variable `message` to our log function, the `logger.info(message)`. To understand this implementation better, edit the file `libs/middlewares.js` as the following code below:

``` javascript
import bodyParser from "body-parser";
import express from "express";
import morgan from "morgan";
import cors from "cors";
import logger from "./logger.js";

module.exports = app => {
  app.set("port", 3000);
  app.set("json spaces", 4);
  app.use(morgan("common", {
    stream: {
      write: (message) => {
        logger.info(message);
      }
    }
  }));
  app.use(cors({
    origin: ["http://localhost:3001"],
    methods: ["GET", "POST", "PUT", "DELETE"],
    allowedHeaders: ["Content-Type", "Authorization"]
  }));
  app.use(bodyParser.json());
  app.use(app.auth.initialize());
  app.use((req, res, next) => {
    delete req.body.id;
    next();
  });
  app.use(express.static("public"));
};
```

To test the log generation, restart the server and access many times any API address, such as [localhost:3000](http://localhost:3000).

After making some requests, access the `logs` folder. There will be a file containing the logs of the requests similar to this one:

![Log of the requests](images/logs-das-requisicoes.png)

## Configuring parallel processing using cluster module

Unfortunately, Node.js does not work with multi-threads. This is something that in the opinion of some developers is considered a negative aspect and that causes disinterest in learning and in taking it seriously. However, despite being single-thread, it's possible to prepare it to work at least with parallel processing. To do this, you can use the native module called `cluster`.

It basically instantiates new processes of an application working in a distributed way and this module takes care to share the same port network between the actives clusters. The number of processes to be created is you who determines, but a good practice is to instantiate a number of processes based in the amount of server processor cores, or also a relative amount to **core x processors**.

For example, if I have a single processor of eight cores, you can instantiate eight processes, creating a **network of eight clusters**. But, if there is four processors of each cores each, you can create a **network of thirty-two clusters** in action.

To make sure the clusters work in a distributed and organized way, it is necessary that a **parent process** exists (a.k.a **cluster master**). Because it is responsible for balancing the parallel processing among the others clusters, distributing this load to the other processes, called **child process** (a.k.a **cluster slave**). It is very easy to implement this technique on Node.js, since all processing distribution is performed abstractedly to the developer.

Another advantage is that the clusters are independent. That is, in case a cluster goes down, the others will continue working. However, it is necessary to manage the instances and the shutdown of clusters manually to ensure the return of the cluster that went down.

Basing ourselves on these concepts, we are going to apply in practice the implementation of clusters. Create in the root directory the file `clusters.js`. See the code bellow:

``` javascript
import cluster from "cluster";
import os from "os";

const CPUS = os.cpus();
if (cluster.isMaster) {
  CPUS.forEach(() => cluster.fork());
  cluster.on("listening", worker => {
    console.log("Cluster %d connected", worker.process.pid);
  });
  cluster.on("disconnect", worker => {
    console.log("Cluster %d disconnected", worker.process.pid);
  });
  cluster.on("exit", worker => {
    console.log("Cluster %d is dead", worker.process.pid);
    cluster.fork();
    // Ensure to starts a new cluster if an old one dies
  });
} else {
  require("./index.js");
}
```

This time, to setup the server, first edit the `package.json` inside the attribute `scripts` and create the command `npm run clusters`, as the code below:

``` json
"scripts": {
  "start": "npm run apidoc && babel-node index.js",
  "clusters": "babel-node clusters.js",
  "test": "NODE_ENV=test mocha test/**/*.js",
  "apidoc": "apidoc -i routes/ -o public/apidoc"
}
```

Now, if you execute the command `npm run clusters`. This time, the application will run distributed into the clusters and to make sure it worked, you will see the message `"NTask API - Port 3000"` more than once in the terminal.

![Running Node.js in cluster mode](images/node-em-clusters.png)

Basically, we have to load the module `cluster` and first, verify it is the **master cluster** via `cluster.isMaster` variable. If the cluster is master, a loop will be iterated based on the total of processing cores **(CPUs)** forking new slave clusters inside the `CPUS.forEach(() => cluster.fork())` function.

When a new process is created (in this case a child process), consequently it does not fit in the conditional `if(cluster.isMaster)`. So, the application server is started via `require("./index.js")` for this child process.

Also, some events created by the **cluster master** are included. In the last code, we only used the main events, listed below:

* `listening`: happens when a cluster is listening a port. In this case, our application is listening the port **3000**;
* `disconnect`: happens when a cluster is disconnected from the cluster's network;
* `exit`: occurs when a cluster is closed in the OS.

> ##### Developing clusters
> A lot of things can be explored about developing clusters on Node.js. Here, we only applied a little bit which was enough to run parallel processing. But, in case you have to implement a more detailed clusters, I recommend you to read the documentation: [nodejs.org/api/cluster.html](https://nodejs.org/api/cluster.html).

To finish and automate the command `npm start` to starts the server in cluster mode, let's update the `package.json` file, following the code below:

``` javascript
"scripts": {
  "start": "npm run apidoc && npm run clusters",
  "clusters": "babel-node clusters.js",
  "test": "NODE_ENV=test mocha test/**/*.js",
  "apidoc": "apidoc -i routes/ -o public/apidoc"
}
```

Done! Now you can start the clusters network using the command `npm start`.

## Compacting requests using GZIP middleware

To make the requests lighter and load faster, let's enable another middleware which is going to be responsible to compact the JSON responses and also the entire API documentation static files to GZIP format – a compatible format to several browsers. We'll do this simple, but important refactoring just using the module `compression`. So let's install it:

``` bash
npm install compression@1.6.1 --save
```

After installing it, will be necessary to include its middleware into `libs/middlewares.js` file:

``` javascript
import bodyParser from "body-parser";
import express from "express";
import morgan from "morgan";
import cors from "cors";
import compression from "compression";
import logger from "./logger.js";

module.exports = app => {
  app.set("port", 3000);
  app.set("json spaces", 4);
  app.use(morgan("common", {
    stream: {
      write: (message) => {
        logger.info(message);
      }
    }
  }));
  app.use(cors({
    origin: ["http://localhost:3001"],
    methods: ["GET", "POST", "PUT", "DELETE"],
    allowedHeaders: ["Content-Type", "Authorization"]
  }));
  app.use(compression());
  app.use(bodyParser.json());
  app.use(app.auth.initialize());
  app.use((req, res, next) => {
    delete req.body.id;
    next();
  });
  app.use(express.static("public"));
};
```

To test this implementation, restart the server and then go to the API documentation, after all, there are a lot of static file there, in the address: [localhost:3000/apidoc](http://localhost:3000/apidoc)

To see in details, open the browser console (Firefox and Google Chrome have greats client-side consoles) and go to the **Networks** menu. There, you can see the **transferred size versus file size**, it's something similar to this image:

![Applying GZIP in some files](images/requisicoes-em-gzip.png)

## Installing SSL support to use HTTPS

Nowadays, it is required to build a safe application that has a safe connection between the server and the client. To do this, many applications buy and use security certificates to ensure a SSL **(Secure Sockets Layer)** connection via the HTTPS protocol.

To implement a HTTPS protocol connection, it is necessary to buy a digital certificate for production's environment usage. But, in our case, we are gonna work with a fake certificate, not valid for production usage, but valid enough for didactic ends. To create a simple certificate, you can go to: [www.selfsignedcertificate.com](http://www.selfsignedcertificate.com)

Informs `ntask` domain and click on **Generate** button. A new screen will be displayed with two extension files: `.key` and `.cert`. Download these files and put them into the root folder of our project.

Now, let's use the native `https` module to allow our server to start using HTTPS protocol and the `fs` module to read the downloaded files: `ntask.key` and `ntask.cert` to be used as credential parameters to start our server in HTTPS mode. To do this, we are going to replace the function `app.listen()` to `https.createServer(credentials, app).listen()` function.. To implement this change, edit the `libs/boot.js`:

``` javascript
import https from "https";
import fs from "fs";

module.exports = app => {
  if (process.env.NODE_ENV !== "test") {
    const credentials = {
      key: fs.readFileSync("ntask.key", "utf8"),
      cert: fs.readFileSync("ntask.cert", "utf8")
    }
    app.db.sequelize.sync().done(() => {
      https.createServer(credentials, app)
        .listen(app.get("port"), () => {
          console.log(`NTask API - Port ${app.get("port")}`);
        });
    });
  }
};
```

Congratulations! Now your application is running in a safe protocol, ensuring that the data won't be intercepted. Note that in a real project this kind of implementation requires a valid digital certificate, so don't forget to buy one if you put a serious API in a production's environment.

To test this changes, just restart and go to: [https://localhost:3000](https://localhost:3000)

## Armoring the API with Helmet

Finishing the development of our API, let's include a very important module, which is a security middleware that handles several kinds of attacks in the HTTP/HTTPS protocols. This module is called `helmet` which is a set of nine internal middlewares, responsible to treat the following HTTP settings:

* Configures the **Content Security Policy**;
* Remove the header `X-Powered-By` that informs the name and the version of a server;
* Configures rules for **HTTP Public Key Pinning**;
* Configures rules for **HTTP Strict Transport Security**;
* Treats the header `X-Download-Options` for Internet Explorer 8+;
* Disable the **client-side caching**;
* Prevents **sniffing** attacks on the client **Mime Type**;
* Prevents **ClickJacking** attacks;
* Protects against XSS **(Cross-Site Scripting)** attacks.

To sum up, even if you do not understand a lot about HTTP security, use `helmet` modules, because in addition to have a simple interface, it will armor your web application against many kinds of attacks. To install it, run the command:

``` bash
npm install helmet@1.1.0 --save
```

To ensure maximum security on our API, we are going to use all middlewares provided by `helmet` module which can easily be included via function `app.use(helmet())`. So, edit the `libs/middlewares.js` file:

``` javascript
import bodyParser from "body-parser";
import express from "express";
import morgan from "morgan";
import cors from "cors";
import compression from "compression";
import helmet from "helmet";
import logger from "./logger.js";

module.exports = app => {
  app.set("port", 3000);
  app.set("json spaces", 4);
  app.use(morgan("common", {
    stream: {
      write: (message) => {
        logger.info(message);
      }
    }
  }));
  app.use(helmet());
  app.use(cors({
    origin: ["http://localhost:3001"],
    methods: ["GET", "POST", "PUT", "DELETE"],
    allowedHeaders: ["Content-Type", "Authorization"]
  }));
  app.use(compression());
  app.use(bodyParser.json());
  app.use(app.auth.initialize());
  app.use((req, res, next) => {
    delete req.body.id;
    next();
  });
  app.use(express.static("public"));
};
```

Now, restart your application and go to: [https://localhost:3000](https://localhost:3000)

Open the browser console and, in the **Networks** menu, you can view in details the `GET /` request's data. There, you'll see new items included in the header, something similar to this image:

![Security headers](images/headers-de-seguranca.png)

### Conclusion

Congrats! We have just finished the complete development of our API! You can use this project as a base for your future APIs, because we have developed a documented API which applies the main REST standards, has endpoints well tested, persist data in a SQL database via Sequelize.js module and, mostly important, follows some best practices about performance and security for production's environments.

But, the book is not over yet! In the next chapter, we'll create a web application, which is going to consume our API. It will be a simple SPA **(Single Page Application)** and will be developed using only JavaScript ES6 code too.
