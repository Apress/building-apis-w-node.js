# Building the client-side app - Part 1

After a long reading about how to build a REST API using Node.js using JavaScript ES6 language, now, from this chapter until the end of this book, we'll create a new project. This time, a front-end project using the best of JavaScript!

This book presented 80% of the content about back-end development, but now, we are gonna focus on the 20% part, which is about front-end development. After all, we have an API, but we need a client-side app, because it's the only way for users interact with our project.

For this reason, in these last chapters, we are going to build a SPA application, using only pure JavaScript. That’s right! Only Vanilla JavaScript!

No front-end framework like Angular, Backbone, Ember, React, jQuery or any others will be used. Only the best of Vanilla JavaScript, or should I say Vanilla ES6?

## Setting up the app environment

Our client-side app will be built under OOP *(Object-Oriented Programming)* paradigm using ES6 classes and Browserify to be able to use some front-end modules from NPM. Also, we are going to automate some tasks to build our application using only npm alias command, like we used in the API project.

To kick off, let's open the terminal to create a new project. This new project cannot be in the same API directory. This new project will be named `ntask-web`, so, let's go:

``` bash
mkdir ntask-web
cd ntask-web
npm init
```

With the `npm init` command, we are gonna answer these questions to describe this new project:

![NTask Web generating package.json file](images/npm-init-ntask-web.png)

After executing the `npm init`, the file `package.json` will be generated. To organize this new project, we must to create some directories in the project's root, these folders are:

* `public`: for static files like css, fonts, javascript and images;
* `public/css`: CSS files only (we are gonna use Ionic CSS);
* `public/fonts`: for font files (we are gonna use Ionic FontIcons);
* `public/js`: For JS files, here we'll put a compiled and minified JS file, generated from the `src` folder;
* `src/components`: for components business logic;
* `src/templates`: for components views;

To create these folders, just run this command:

``` bash
mkdir public/{css,fonts,js}
mkdir src/{components,templates}
```

Below, there is a list with all modules which will be used in this project:

* `http-server`: CLI for start a simple HTTP server for static files only.
* `browserify`: a JavaScript compiler which allows to use some NPM modules, we'll use some modules which have JavaScript Universal codes (codes who works in back-end and front-end layers) and also allows to load using CommonJS pattern, which is the same standard from Node.js.
* `babelify`: a plugin for `browserify` module to enable read and compile ES6 code in the front-end.
* `babel-preset-es2015`: the presets for babel recognizes ES6 code via `babelify`
* `tiny-emitter`: a small module to create and handle events.
* `browser-request`: is a version of `request` module focused for browsers, it is a *cross-browser* (compatible with all major browsers) and abstracts the whole complexity of an AJAX's request.

Basically, we are going to build a web client using these modules. So, let's install them running these commands:

``` bash
npm install http-server@0.9.0 browserify@13.0.0 --save
npm install babelify@7.2.0 babel-preset-es2015@6.5.0 --save
npm install tiny-emitter@1.0.2 browser-request@0.3.3 --save
```

After the installation, let's modify `package.json`, removing the attributes `main`, `script.test` and `license`, and add all the alias commands which will be necessary to automate the build process of this project.

Basically, we'll do this tasks:

* Compile and concatenate all ES6 codes from `src` folder via `npm run build` command;
* Initiate the server in the port `3001` via `npm run server` command;
* Create the command `npm start`, which will run: `npm run build` and `npm run server` commands;

To apply these alterations, edit the `package.json`:

``` json
{
  "name": "ntask-web",
  "version": "1.0.0",
  "description": "NTask web client application",
  "scripts": {
    "start": "npm run build && npm run server",
    "server": "http-server public -p 3001",
    "build": "browserify src -t babelify -o public/js/app.js"
  },
  "author": "Caio Ribeiro Pereira",
  "dependencies": {
    "babel-preset-es2015": "^6.5.0",
    "babelify": "^7.2.0",
    "browser-request": "^0.3.3",
    "browserify": "^13.0.0",
    "http-server": "^0.9.0",
    "tiny-emitter": "^1.0.2"
  }
}
```

Now, we need to create again, the `.babelrc` file, to setup the `babel-preset-es2015` module, to do this simple task, create the `.babelrc` in root folder using this code:

``` json
{
  "presets": ["es2015"]
}
```

After this setup, let's include some static files that will be responsible for the layout of our project. To not waist time, we are gonna use a ready CSS stylization, from the Ionic framework, a very cool framework with several mobile components ready to build responsive apps.

We won't use the full Ionic framework, because it has a strong dependency of Angular framework. We are only gonna use the CSS file and font icon package. To do this, I recommend you to download the files from the link listed below:

* Ionic CSS: [code.ionicframework.com/1.0.0/css/ionic.min.css](http://code.ionicframework.com/1.0.0/css/ionic.min.css)
* Ionic CSS icons: [code.ionicframework.com/ionicons/2.0.0/css/ionicons.min.css](http://code.ionicframework.com/ionicons/2.0.0/css/ionicons.min.css)
* Ionic Font icons: [code.ionicframework.com/1.0.0/fonts/ionicons.eot](http://code.ionicframework.com/1.0.0/fonts/ionicons.eot)
* Ionic Font icons: [code.ionicframework.com/1.0.0/fonts/ionicons.svg](http://code.ionicframework.com/1.0.0/fonts/ionicons.svg)
* Ionic Font icons: [code.ionicframework.com/1.0.0/fonts/ionicons.ttf](http://code.ionicframework.com/1.0.0/fonts/ionicons.ttf)
* Ionic Font icons: [code.ionicframework.com/1.0.0/fonts/ionicons.woff](http://code.ionicframework.com/1.0.0/fonts/ionicons.woff)

And put their CSS files into `public/css` folder and put the fonts files into `public/fonts`.

To start coding, let's create the homepage and the JavaScript code responsible to load the main page, let's do it, just to test if everything will run fine. The main HTML is going to be responsible to load the Ionic CSS stylization, the main JavaScript files and also few HTML tags enough to build the layout structure. To understand this implementation, create the file `public/index.html` using the code below:

``` html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>NTask - Task manager</title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <link rel="stylesheet" href="css/ionic.min.css">
    <link rel="stylesheet" href="css/ionicons.min.css">
    <script src="js/app.js" async defer></script>
  </head>
  <body>
    <header class="bar bar-header bar-calm">
      <h1 class="title">NTask</h1>
    </header>
    <div class="scroll-content ionic-scroll">
      <div class="content overflow-scroll has-header">
        <main></main>
        <footer></footer>
      </div>
    </div>
  </body>
</html>
```

Note: there are two empty tags: `<main></main>` and `<footer></footer>`. All the page interaction will manipulate these tags dynamically using JavaScript codes from the `src/components` files that we will write soon.

To finish this section, create the `src/index.js` file, for while, this will display a simple `Welcome!` message in the browser when the page is loaded. This will be modified soon, after all, we are going to create it, just to test if our project will work fine.

``` javascript
window.onload = () => {
  alert("Welcome!");
};
```

Now we already have a simple, but functional environment to run our NTask client-side app. To execute it, run the command `npm start` and go to: [localhost:3001](http://localhost:3001)

If everything works fine, you'll see this result:

![First NTask screen](images/ntask-web-primeira-tela.png)

## Creating signin and signup views

In this section, we are gonna create all the templates to be used in the application. The templates are basically HTML pieces of strings, manipulated via JavaScript, and are largely used on SPA projects. After all, one of the SPA concepts is to load once all static files to only transfer data frequently between client and server.

All screens transition (template's transition) become a task for client-side apps, causing the server only to work with data and the client to pick up these data to put them into an appropriate screens for users interacts in the application.

Our application is a simple task manager, which has a REST API with endpoints to create, update, delete and list tasks, and register, find and delete an user. All templates will be based on these API's features.

To start, let's build the template that is going to be the **sign in** and **sign up** screens. Thanks to the **Template String** feature provided by ES6, it has become possible to create strings with data concatenation in an elegant way using the syntax:

``` javascript
console.log(`Olá ${nome}!`);
```

With this, it's no longer be necessary to use any template engine framework, because we can easily create the templates using a simple function to return a HTML string concatenated with some data.

To understand this implementation better, let's write the sign in homepage templates using the function `render()` to return a HTML string. Create the file `src/templates/signin.js`:

``` javascript
exports.render = () => {
  return `<form>
    <div class="list">
      <label class="item item-input item-stacked-label">
        <span class="input-label">Email</span>
        <input type="text" data-email>
      </label>
      <label class="item item-input item-stacked-label">
        <span class="input-label">Password</span>
        <input type="password" data-password>
      </label>
    </div>
    <div class="padding">
      <button class="button button-positive button-block">
        <i class="ion-home"></i> Login
      </button>
    </div>
  </form>
  <div class="padding">
    <button class="button button-block" data-signup>
      <i class="ion-person-add"></i> Sign up
    </button>
  </div>`;
};
```

Now, to complete the flow, we are also gonna create the sign up screen template. Create the file `src/templates/signup.js`, like this code below:

``` javascript
exports.render = () => {
  return `<form>
    <div class="list">
      <label class="item item-input item-stacked-label">
        <span class="input-label">Name</span>
        <input type="text" data-name>
      </label>
      <label class="item item-input item-stacked-label">
        <span class="input-label">Email</span>
        <input type="text" data-email>
      </label>
      <label class="item item-input item-stacked-label">
        <span class="input-label">Password</span>
        <input type="password" data-password>
      </label>
    </div>
    <div class="padding">
      <button class="button button-positive button-block">
        <i class="ion-thumbsup"></i> Register
      </button>
    </div>
  </form>`;
};
```

Done! We already have two important screens of the application. Now, we have to create the interaction code for these screens which they will be responsible to render these templates and, especially, interact with the users and communicate with the API.

## Writing signin and signup components

The codes for template's interaction are going to be put into `src/components` folder, but before creating them, let's explore two new JavaScript ES6 features: **classes** and **inheritance**. To write our front-end code more semantic and well organized, we'll create a parent class, which is going to have only two important attributes as inheritance for all components which extend it: `this.URL` (contains the API URL address) and `this.request` (contains the `browser-request` module loaded).

Another detail of this parent class is that it will inherit all functionalities from the `tiny-emitter` module (via `class NTask extends TinyEmitter` declaration), and these functionalities will be passed for their child classes as well, allowing our components to be able to listen and trigger events among the others components. To understand this class better, create the file `src/ntask.js` following the code below:

``` javascript
import TinyEmitter from "tiny-emitter";
import request from "browser-request";

class NTask extends TinyEmitter {
  constructor() {
    super();
    this.request = request;
    this.URL = "https://localhost:3000";
  }
}

module.exports = NTask;
```

Now that we have the parent class `NTask`, will be possible to create the component's classes, not only are going to have specific functionalities but also attributes and generic functions inherited by the parent class. That is the components, instead of double or triple the codes; they will reuse the codes.

Let's create our first component for the sign in screen. All component's classes of our application will receive from the constructor the `body` object. This `body` is basically, the DOM object from the tag `<main>` or `<footer>`. All components will inherit from the parent class `NTask`, so, the execution of the function `super()` is required on child class's constructor. We'll use to organize our components the methods: `render()` (to render a template) and `addEventListener()` (to listen and treat any DOM components from the templates).

In this case, we are gonna have two encapsulated events in the methods: `formSubmit()` (responsible to make an user authentication on the API) and `signupClick()` (responsible to render the sign up screen in the browser).

Each component must emit an event via the function `this.emit("event-name")`, because we will create an observer class to listen and delegate tasks among the application's components. A good example of task that is going to be largely used is the template's transitions, which occurs when an user clicks on a template's button, and, the click event is triggered to the observer class listen and delegate a task for a new template be rendered in the browser.

To better understand these rules, create the file `src/components/signin.js` following this large code below:

``` javascript
import NTask from "../ntask.js";
import Template from "../templates/signin.js";

class Signin extends NTask {
  constructor(body) {
    super();
    this.body = body;
  }
  render() {
    this.body.innerHTML = Template.render();
    this.body.querySelector("[data-email]").focus();
    this.addEventListener();
  }
  addEventListener() {
    this.formSubmit();
    this.signupClick();
  }
  formSubmit() {
    const form = this.body.querySelector("form");
    form.addEventListener("submit", (e) => {
      e.preventDefault();
      const email = e.target.querySelector("[data-email]");
      const password = e.target.querySelector("[data-password]");
      const opts = {
        method: "POST",
        url: `${this.URL}/token`,
        json: true,
        body: {
          email: email.value,
          password: password.value
        }
      };
      this.request(opts, (err, resp, data) => {
        if (err || resp.status === 401) {
          this.emit("error", err);
        } else {
          this.emit("signin", data.token);
        }
      });
    });
  }
  signupClick() {
    const signup = this.body.querySelector("[data-signup]");
    signup.addEventListener("click", (e) => {
      e.preventDefault();
      this.emit("signup");
    });
  }
}

module.exports = Signin;
```

Let's also create the `Signup` class. This will follow the same pattern from the `Signin` class. To see how it must be, create the `src/components/signup.js` like this:

``` javascript
import NTask from "../ntask.js";
import Template from "../templates/signup.js";

class Signup extends NTask {
  constructor(body) {
    super();
    this.body = body;
  }
  render() {
    this.body.innerHTML = Template.render();
    this.body.querySelector("[data-name]").focus();
    this.addEventListener();
  }
  addEventListener() {
    this.formSubmit();
  }
  formSubmit() {
    const form = this.body.querySelector("form");
    form.addEventListener("submit", (e) => {
      e.preventDefault();
      const name = e.target.querySelector("[data-name]");
      const email = e.target.querySelector("[data-email]");
      const password = e.target.querySelector("[data-password]");
      const opts = {
        method: "POST",
        url: `${this.URL}/users`,
        json: true,
        body: {
          name: name.value,
          email: email.value,
          password: password.value
        }
      };
      this.request(opts, (err, resp, data) => {
        if (err || resp.status === 412) {
          this.emit("error", err);
        } else {
          this.emit("signup", data);
        }
      });
    });
  }
}

module.exports = Signup;
```

To finish this initial flow, we still have to create the observer class and then, load it inside `src/index.js` file. This will be the responsible code for starting all the interaction among the application's components. The observer class will be called `App`. His constructor will perform the instance of all components, and will have two main methods: the `init()` (responsible to start the component's interactions) and `addEventListener()` (responsible to listen and treat the component's events).
To understand this implementation, let's create the `src/app.js` file, following this code below:

``` javascript
import Signin from "./components/signin.js";
import Signup from "./components/signup.js";

class App {
  constructor(body) {
    this.signin = new Signin(body);
    this.signup = new Signup(body);
  }
  init() {
    this.signin.render();
    this.addEventListener();
  }
  addEventListener() {
    this.signinEvents();
    this.signupEvents();
  }
  signinEvents() {
    this.signin.on("error", () => alert("Authentication error"));
    this.signin.on("signin", (token) => {
      localStorage.setItem("token", `JWT ${token}`);
      alert("You are logged in!");
    });
    this.signin.on("signup", () => this.signup.render());
  }
  signupEvents() {
    this.signup.on("error", () => alert("Register error"));
    this.signup.on("signup", (user) => {
      alert(`${user.name} you were registered!`);
      this.signin.render();
    });
  }
}

module.exports = App;
```

Basically, it was created the component's events for successful authentication, authentication error, access the sign up screen, successful sign up and sign up error. The `init()` method starts the homepage, which is the sign in screen, and then, executes the `addEventListener()` method, to listen all the components events that are encapsulated inside the methods `signinEvents()` and `signupEvents()`.

To finish this chapter, edit the `src/index.js` file, to be able to load and initiate the `App` class inside the `window.onload()` event and then, start all the interactive flow of the application:

``` javascript
import App from "./app.js"

window.onload = () => {
  const main = document.querySelector("main");
  new App(main).init();
};
```

Let’s test it? If you followed step-by-step these implementations, you'll have a basic sign in and sign up flow. To run this application, first, open two terminal: one for API server and the other for the client-side app.

In both terminals, you must run the command `npm start`. Now these applications will be available in the addresses:

* **NTask API:** [https://localhost:3000](https://localhost:3000)
* **Ntask Web:** [http://localhost:3001](http://localhost:3001)

As we are using a not valid digital certificate for production environment, it is quite probable that your browser will block the first access to the API. In case it happens, just open in the browser: [https://localhost:3000](https://localhost:3000)

Then, and add an exception in your browser to enable the API access. See this image below to understand how to add an exception on Mozilla Firefox browser, for example:

![Adding and exception to API address](images/adicionando-excecao-ntask-api.png)

Now that the API has full access, go to the NTask web address: [http://localhost:3001](http://localhost:3001)

If everything is right, you'll be able to access these screens:

![Sign in screen](images/ntask-web-signin.png)

![User sign up screen](images/ntask-web-signup.png)

![Registering a new user](images/ntask-web-cadastrando-usuario.png)

![Logging in with new account](images/ntask-web-logando.png)

### Conclusion

Our new project is taking shape, and we are closer to build a functional system for real users, all of this integrating the client application with the API that was built in the last chapters.

In this chapter, we created only the environment and some screens, enough to structure the application. Keep reading, because in the next chapter we'll go deeper until finish this project. See ya!
