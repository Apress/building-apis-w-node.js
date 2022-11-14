# Building the client-side app - Part 2

Continuing with the client application construction, up to now we have an application with the main layout that connects with the API and allows to authenticate a user to access the system. In this chapter, we are going to build the main features for users be able to manage their tasks.

## Views and components for task's CRUD

The tasks template construction will be quite complex, but in the end, will be great! This template must list all user's tasks, so this function will receive as an argument a tasks list and using the function `tasks.map()` will generate a template's array of the tasks.

In the end of this new array generation, the function `.join("")` will be executed and will concatenate all items returning a single template string of tasks. To make this manipulation and generation of the tasks templates easier, this logic will be encapsulated via `renderTasks(tasks)` function. Otherwise, it displays a message about empty task list.

To understand better this implementation, create the file `src/templates/tasks.js` as below:

``` javascript
const renderTasks = tasks => {
  return tasks.map(task => {
    let done = task.done ? "ios-checkmark" : "ios-circle-outline";
    return `<li class="item item-icon-left item-button-right">
      <i class="icon ion-${done}" data-done
        data-task-done="${task.done ? 'done' : ''}"
        data-task-id="${task.id}"></i>
      ${task.title}
      <button data-remove data-task-id="${task.id}"
        class="button button-assertive">
        <i class="ion-trash-a"></i>
      </button>
    </li>`;
  }).join("");
};
exports.render = tasks => {
  if (tasks && tasks.length) {
    return `<ul class="list">${renderTasks(tasks)}</ul>`;
  }
  return `<h4 class="text-center">The task list is empty</h4>`;
};
```

Using the attributes `data-task-done`, `data-task-id`, `data-done` and `data-remove`, we are gonna work on how to create components for tasks manipulation. These attributes will be used to trigger some events to allow deleting a task (via method `taskRemoveClick()`) and/or setting which task will be done (via method `taskDoneCheckbox()`). These business rules will be written on `src/components/tasks.js` file:

``` javascript
import NTask from "../ntask.js";
import Template from "../templates/tasks.js";

class Tasks extends NTask {
  constructor(body) {
    super();
    this.body = body;
  }
  render() {
    this.renderTaskList();
  }
  addEventListener() {
    this.taskDoneCheckbox();
    this.taskRemoveClick();
  }
  renderTaskList() {
    const opts = {
      method: "GET",
      url: `${this.URL}/tasks`,
      json: true,
      headers: {
        authorization: localStorage.getItem("token")
      }
    };
    this.request(opts, (err, resp, data) => {
      if (err) {
        this.emit("error", err);
      } else {
        this.body.innerHTML = Template.render(data);
        this.addEventListener();
      }
    });
  }
  taskDoneCheckbox() {
    const dones = this.body.querySelectorAll("[data-done]");
    for(let i = 0, max = dones.length; i < max; i++) {
      dones[i].addEventListener("click", (e) => {
        e.preventDefault();
        const id = e.target.getAttribute("data-task-id");
        const done = e.target.getAttribute("data-task-done");
        const opts = {
          method: "PUT",
          url: `${this.URL}/tasks/${id}`,
          headers: {
            authorization: localStorage.getItem("token"),
            "Content-Type": "application/json"
          },
          body: JSON.stringify({done: !done})
        };
        this.request(opts, (err, resp, data) => {
          if (err || resp.status === 412) {
            this.emit("update-error", err);
          } else {
            this.emit("update");
          }
        });
      });
    }
  }
  taskRemoveClick() {
    const removes = this.body.querySelectorAll("[data-remove]");
    for(let i = 0, max = removes.length; i < max; i++) {
      removes[i].addEventListener("click", (e) => {
        e.preventDefault();
        if (confirm("Do you really wanna delete this task?")) {
          const id = e.target.getAttribute("data-task-id");
          const opts = {
            method: "DELETE",
            url: `${this.URL}/tasks/${id}`,
            headers: {
              authorization: localStorage.getItem("token")
            }
          };
          this.request(opts, (err, resp, data) => {
            if (err || resp.status === 412) {
              this.emit("remove-error", err);
            } else {
              this.emit("remove");
            }
          });
        }
      });
    }
  }
}

module.exports = Tasks;
```

Now that we have the component responsible for listing, updating and deleting tasks, let's implement the template and component responsible for adding a new task. This will be easier, because it will be a template with a simple form to register new tasks, and in the end, it redirects to the task list. To do it, create the file `src/templates/taskForm.js`:

``` javascript
exports.render = () => {
  return `<form>
    <div class="list">
      <label class="item item-input item-stacked-label">
        <span class="input-label">Task</span>
        <input type="text" data-task>
      </label>
    </div>
    <div class="padding">
      <button class="button button-positive button-block">
        <i class="ion-compose"></i> Add
      </button>
    </div>
  </form>`;
};
```

Then, create its respective component which will have only the form submission event from the encapsulated function `formSubmit()`. Create the file `src/components/taskForm.js`:

``` javascript
import NTask from "../ntask.js";
import Template from "../templates/taskForm.js";

class TaskForm extends NTask {
  constructor(body) {
    super();
    this.body = body;
  }
  render() {
    this.body.innerHTML = Template.render();
    this.body.querySelector("[data-task]").focus();
    this.addEventListener();
  }
  addEventListener() {
    this.formSubmit();
  }
  formSubmit() {
    const form = this.body.querySelector("form");
    form.addEventListener("submit", (e) => {
      e.preventDefault();
      const task = e.target.querySelector("[data-task]");
      const opts = {
        method: "POST",
        url: `${this.URL}/tasks`,
        json: true,
        headers: {
          authorization: localStorage.getItem("token")
        },
        body: {
          title: task.value
        }
      };
      this.request(opts, (err, resp, data) => {
        if (err || resp.status === 412) {
          this.emit("error");
        } else {
          this.emit("submit");
        }
      });
    });
  }
}

module.exports = TaskForm;
```

## Views and components for logged users

To finish the screens creation of our application, we are gonna build the last screen, which will display the logged user's data and a button to be able the user to cancel his account. This screen will have a component very easy to implement as well, because this will only treat the event of the account's cancellation button. Create the `src/templates/user.js`:

``` javascript
exports.render = user => {
  return `<div class="list">
    <label class="item item-input item-stacked-label">
      <span class="input-label">Name</span>
      <small class="dark">${user.name}</small>
    </label>
    <label class="item item-input item-stacked-label">
      <span class="input-label">Email</span>
      <small class="dark">${user.email}</small>
    </label>
  </div>
  <div class="padding">
    <button data-remove-account
      class="button button-assertive button-block">
      <i class="ion-trash-a"></i> Cancel account
    </button>
  </div>`;
};
```

Now that we have the user's screen template, let's create its respective component in the `src/components/user.js` file, following the code below:

``` javascript
import NTask from "../ntask.js";
import Template from "../templates/user.js";

class User extends NTask {
  constructor(body) {
    super();
    this.body = body;
  }
  render() {
    this.renderUserData();
  }
  addEventListener() {
    this.userCancelClick();
  }
  renderUserData() {
    const opts = {
      method: "GET",
      url: `${this.URL}/user`,
      json: true,
      headers: {
        authorization: localStorage.getItem("token")
      }
    };
    this.request(opts, (err, resp, data) => {
      if (err || resp.status === 412) {
        this.emit("error", err);
      } else {
        this.body.innerHTML = Template.render(data);
        this.addEventListener();
      }
    });
  }
  userCancelClick() {
    const button = this.body.querySelector("[data-remove-account]");
    button.addEventListener("click", (e) => {
      e.preventDefault();
      if (confirm("This will cancel your account, are you sure?")) {
        const opts = {
          method: "DELETE",
          url: `${this.URL}/user`,
          headers: {
            authorization: localStorage.getItem("token")
          }
        };
        this.request(opts, (err, resp, data) => {
          if (err || resp.status === 412) {
            this.emit("remove-error", err);
          } else {
            this.emit("remove-account");
          }
        });
      }
    });
  }
}

module.exports = User;
```

## Creating the main menu

To make this application more elegant and interactive, we are going to also create in its footer the main menu to help users interact with the tasks list and users settings. To create this screen, first we need to create its template, which will have only three buttons: **tasks**, **add task** and **logout**. Create the file `src/templates/footer.js`:

``` javascript
exports.render = path => {
  let isTasks = path === "tasks" ? "active" : "";
  let isTaskForm = path === "taskForm" ? "active" : "";
  let isUser = path === "user" ? "active" : "";
  return `
    <div class="tabs-striped tabs-color-calm">
      <div class="tabs">
        <a data-path="tasks" class="tab-item ${isTasks}">
          <i class="icon ion-home"></i>
        </a>
        <a data-path="taskForm" class="tab-item ${isTaskForm}">
          <i class="icon ion-compose"></i>
        </a>
        <a data-path="user" class="tab-item ${isUser}">
          <i class="icon ion-person"></i>
        </a>
        <a data-logout class="tab-item">
          <i class="icon ion-android-exit"></i>
        </a>
      </div>
    </div>`;
};
```

Then, create its corresponding component file: `src/components/menu.js`:

``` javascript
import NTask from "../ntask.js";
import Template from "../templates/footer.js";

class Menu extends NTask {
  constructor(body) {
    super();
    this.body = body;
  }
  render(path) {
    this.body.innerHTML = Template.render(path);
    this.addEventListener();
  }
  clear() {
    this.body.innerHTML = "";
  }
  addEventListener() {
    this.pathsClick();
    this.logoutClick();
  }
  pathsClick() {
    const links = this.body.querySelectorAll("[data-path]");
    for(let i = 0, max = links.length; i < max; i++) {
      links[i].addEventListener("click", (e) => {
        e.preventDefault();
        const link = e.target.parentElement;
        const path = link.getAttribute("data-path");
        this.emit("click", path);
      });
    }
  }
  logoutClick() {
    const link = this.body.querySelector("[data-logout]");
    link.addEventListener("click", (e) => {
      e.preventDefault();
      this.emit("logout");
    })
  }
}

module.exports = Menu;
```

## Treating all screen's events

Our project has all the necessary components to build a task list application, now to finish our project we need to assemble all pieces of the puzzle! To start, let's modify the `src/index.js` so it can manipulate not only the `<main>` tag but also the `<footer>` tag, because this new tag will be used to handle events of the footer menu.

Edit the `src/index.js` applying this simple modification:

``` javascript
import App from "./app.js";

window.onload = () => {
  const main = document.querySelector("main");
  const footer = document.querySelector("footer");
  new App(main, footer).init();
};
```

Now, to finish our project, we have to update the object `App` so it can be responsible for loading all the components that were created and, treat the events of each component. Doing this change we will ensure the correct flow for all screen's transition, the menu transition and the data traffic between the `ntask-api` and `ntask-web` projects. To do this, edit the `src/app.js`, coding this huge script:

``` javascript
import Tasks from "./components/tasks.js";
import TaskForm from "./components/taskForm.js";
import User from "./components/user.js";
import Signin from "./components/signin.js";
import Signup from "./components/signup.js";
import Menu from "./components/menu.js";

class App {
  constructor(body, footer) {
    this.signin = new Signin(body);
    this.signup = new Signup(body);
    this.tasks = new Tasks(body);
    this.taskForm = new TaskForm(body);
    this.user = new User(body);
    this.menu = new Menu(footer);
  }
  init() {
    this.signin.render();
    this.addEventListener();
  }
  addEventListener() {
    this.signinEvents();
    this.signupEvents();
    this.tasksEvents();
    this.taskFormEvents();
    this.userEvents();
    this.menuEvents();
  }
  signinEvents() {
    this.signin.on("error", () => alert("Authentication error"));
    this.signin.on("signin", (token) => {
      localStorage.setItem("token", `JWT ${token}`);
      this.menu.render("tasks");
      this.tasks.render();
    });
    this.signin.on("signup", () => this.signup.render());
  }
  signupEvents(){
    this.signup.on("error", () => alert("Register error"));
    this.signup.on("signup", (user) => {
      alert(`${user.name} you were registered!`);
      this.signin.render();
    });
  }
  tasksEvents() {
    this.tasks.on("error", () => alert("Task list error"));
    this.tasks.on("remove-error", () => alert("Task delete error"));
    this.tasks.on("update-error", () => alert("Task update error"));
    this.tasks.on("remove", () => this.tasks.render());
    this.tasks.on("update", () => this.tasks.render());
  }
  taskFormEvents() {
    this.taskForm.on("error", () => alert("Task register error"));
    this.taskForm.on("submit", () => {
      this.menu.render("tasks");
      this.tasks.render();
    });
  }
  userEvents() {
    this.user.on("error", () => alert("User load error"));
    this.user.on("remove-error", () => alert("Cancel account error"));
    this.user.on("remove-account", () => {
      alert("So sad! You are leaving us :(");
      localStorage.clear();
      this.menu.clear();
      this.signin.render();
    });
  }
  menuEvents() {
    this.menu.on("click", (path) => {
      this.menu.render(path);
      this[path].render();
    });
    this.menu.on("logout", () => {
      localStorage.clear();
      this.menu.clear();
      this.signin.render();
    })
  }
}

module.exports = App;
```

Phew! It’s done! We built our simple, but useful client application to interact with our current API. Let’s test it? You just need to restart the client application and use it normally. Below, you ca see some new screens to access, take a look:

![Adding task](images/ntask-web-cadastrando-tarefa.png)

![Listing and checking some tasks](images/ntask-web-listando-tarefas.png)

![User settings screen](images/ntask-web-dados-de-usuario.png)

### Final conclusion

Congratulations! If you reached this far with your application running perfectly, so you have finished this book successfully. I hope you've learned a lot about Node.js platform reading this book, and especially on how to build a simple, but useful REST API, because that's the essence of the ebook. I believe I have passed the necessary knowledge for you, faithful reader.

Remember that all sources are available into my personal GitHub. Just access this link: [github.com/caio-ribeiro-pereira/building-apis-with-nodejs](https://github.com/caio-ribeiro-pereira/building-apis-with-nodejs).

Thank you very much for reading this book!
