# Managing modules with NPM

## What does NPM do?

Just like RubyGems from Ruby or Maven from Java, Node.js also has a package manager, which is called NPM *(Node Package Manager)*. It has become so popular throughout the community that since the **0.6.X Node.js version** it was integrated within the main Node.js installer, becoming the standard manager of this platform. This helped the developers at that time, because it made several projects converge to this tool.

![npmjs.org](images/site-npm.jpg)

Nowadays, the NPM homepage hosts more than **230k of modules** created by developers, companies and community. **+149 million downloads are made daily and +3.4 million downloads are made monthly**.

## Top NPM commands

Use NPM is very easy to use. You can use it to manage your project's dependencies and you create commands for tasks automation purposes, everything is created and managed into the `package.json` file. See bellow a list of the top npm commands:

* `npm init`: displays a simple wizard to help you create and describe your new project;
* `npm install module_name`: install a module;
* `npm install -g module_name`: install a global module;
* `npm install module_name --save`: install a module and add it into the `package.json` file, inside `dependencies`;
* `npm install module_name --save-dev`: install a module and add it into the `package.json` file, inside `devDependencies`;
* `npm list`: lists all the modules installed on the project;
* `npm list -g`: lists all the global modules installed on the OS;
* `npm remove module_name`: uninstall a module from the project;
* `npm remove -g module_name`: uninstall a global module;
* `npm remove module_name --save`: uninstall a module from the project and also remove it from the attribute `dependencies`;
* `npm remove module_name --save-dev`: uninstall a module from the project and also remove it from the attribute `devDependencies`;
* `npm update module_name`: updates the module version;
* `npm update -g module_name`: updates the a global module version;
* `npm -v`: displays the npm current version;
* `npm adduser username`: creates an account to [npmjs.org](https://npmjs.org);
* `npm whoami`: displays details of your public npm's profile (you must create an account using the previous command);
* `npm publish`: publishes a module into npmjs.org (it's necessary to have an active account first);

## Understanding package.json file

All Node.js projects are called modules. But, what is a module? Throughout the book, note that I am going to use the terms module, library and framework, in practice, they all mean the same thing.

The term **module** was taken from the JavaScript's concept, which is a language that works with a **modular architecture**. When we create a Node.js project, that is, a module, this project is followed by a descriptor file of modules, called `package.json`.

This file is essential to a Node.js project. A `package.json` badly written can cause bugs or even prevent your project to be executed, because this file has some key attributes, that are read by both Node.js and the NPM.

To demonstrate it, see bellow a simple example of a `package.json` that describes the main attributes of a module:

``` json
{
  "name": "my-first-node-app",
  "description": "My first node app",
  "author": "User <user@email.com>",
  "version": "1.2.3",
  "private": true,
  "dependencies": {
    "module-1": "1.0.0",
    "module-2": "~1.0.0",
    "module-3": ">=1.0.0"
  },
  "devDependencies": {
    "module-4": "*"
  }
}
```

With those few attributes, you are able to describe about your module. The attribute `name` is the main one. It defines a programmatic name of the project, this module name will be called via `require("my-first-node-app")` function. It must be written in a clean and short way, offering an abstract about what it will be the module.

The `author` attribute is the one which informs the name and email of the author. You can use the format `Name <email>` in order to websites, such as [npmjs.org](https://npmjs.org) can read this information correctly.

Another main attribute is the `version`, it defines the current version of the module. It is highly recommended that you use this attribute to allow a module installation via `npm install my-first-node-app` command. The attribute `private` is optional, it is only a boolean that determines if the project is an open-source or is a private project.

Node.js modules work with **three levels of versioning**.
For example, the version `1.2.3` is divided into the levels:

1. **Major: X.0.0**
2. **Minor: 0.X.0**
3. **Patch: 0.0.X**

Note the **X** means the current level version to update.

In the previous `package.json` have 4 modules, each one uses a different semantic version. The first one, the `"module-1"`, has a fixed version `1.0.0`. Use this kind of version to install dependencies whose updates can break the project if you change the version.

The `"module-2"` already has a certain update flexibility. It uses the character `"~"` which allows you to update a module as a `patch` level `1.0.x` (it only update the `x` version). Generally, these updates are safe, they brings improvements and bug fixes.

The `"module-3"` updates versions which are greater or equal to `1.0.0` in all the version levels. In many cases, using `">="` can be risky, because the dependency can be updated to a `major` or `minor` level and, consequently, can bring some modifications that can break your application.

The last one, the `"module-4"` uses the `"*"` character. This one will always catch the latest version of the module in any version level. It also can cause troubles in the updates and it has the same behavior as the `module-3` versioning. Generally, it is used only in the `devDependencies` which are dependencies used for tests purposes and do not affect the application behavior. **Do not use this kind of version in production environment!**

In case you want to have a more precise control over the versions of your dependencies after their installation in your project, run the `npm shrinkwrap` command. It will lock all dependencies version within the file `npm-shrinkwrap.json` allowing you to have control over the dependencies versions of your project. This command is very used in production's environment, where the versions need to be stricter.

## NPM Task automation

You can also automate tasks using the `npm` command. In practice, you can only create new executable commands running `npm run command_name`. To declare those new commands, just create them inside the attribute `scripts` on `package.json` like this example bellow:

``` json
{
  "name": "my-first-node-app",
  "description": "My first node app",
  "author": "User <user@email.com>",
  "version": "1.2.3",
  "private": true,
  "scripts": {
    "start": "node app.js",
    "clean": "rm -rf node_modules",
    "test": "node test.js"
  },
  "dependencies": {
    "module-1": "1.0.0",
    "module-2": "~1.0.0",
    "module-3": ">=1.0.0"
  },
  "devDependencies": {
    "module-4": "*"
  }
}
```

Note that this new `package.json` created three scripts: `start`, `clean` and `test`. These scripts are executable now via their commands: `npm run start`, `npm run clean` and `npm run test`.
As shortcut, only the `start` and `test` commands can be executed by the alias: `npm start` and `npm test`. Within the scripts, you can run both the commands `node`, `npm` or any other global command from the OS. The `npm run clean` is an example of a global command from the `bash`, which internally runs the command: `rm -rf node_modules` to deletes all files from the `node_modules` folder.

### Conclusion

If you reached this point, you have learned the basics about managing dependencies and automating tasks using npm. It is highly important to understand these concepts, because it will be frequently used throughout this book.

Fasten your seatbelts and get ready! In the next chapter we are going to start our API RESTful pilot project, which is going to be worked on throughout the next chapters. As spoiler, I am going to tell you about what this project will be.

In the next chapters, we are going to create a simple API REST system to manage tasks. Not only an API is going to be developed but also we'll create a web application to consume data from this API. All development is going to happen using some Node.js popular frameworks: Express to create the API resources, Sequelize to persist data in a relational database, Passport to deal with user authentications and a lot more.
