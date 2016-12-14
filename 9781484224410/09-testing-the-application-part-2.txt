# Testing the application - Part 2

Continuing the tests implementation, now we will focus on write some tests for the resources: **tasks** and **users**.

## Testing task's endpoints

To test the endpoints of task's resource, we are going to cheat the **JWT authentication**. After all, will be necessary to correctly test the results of this resource and also the others resources which involves user's authentication. To start it, let's create the structure for the tasks tests.

Create the file `test/routes/tasks.js` with the following code:

``` javascript
import jwt from "jwt-simple";

describe("Routes: Tasks", () => {
  const Users = app.db.models.Users;
  const Tasks = app.db.models.Tasks;
  const jwtSecret = app.libs.config.jwtSecret;
  let token;
  let fakeTask;
  beforeEach(done => {
    // Runs before each test...
  });
  describe("GET /tasks", () => {
    describe("status 200", () => {
      it("returns a list of tasks", done => {
        // Test's logic...
      });
    });
  });
  describe("POST /tasks/", () => {
    describe("status 200", () => {
      it("creates a new task", done => {
        // Test's logic...
      });
    });
  });
  describe("GET /tasks/:id", () => {
    describe("status 200", () => {
      it("returns one task", done => {
        // Test's logic...
      });
    });
    describe("status 404", () => {
      it("throws error when task not exist", done => {
        // Test's logic...
      });
    });
  });
  describe("PUT /tasks/:id", () => {
    describe("status 204", () => {
      it("updates a task", done => {
        // Test's logic...
      });
    });
  });
  describe("DELETE /tasks/:id", () => {
    describe("status 204", () => {
      it("removes a task", done => {
        // Test's logic...
      });
    });
  });
});
```

Detailing how to cheat the authentication part, we are going to reuse the module `jwt-simple` to create a valid token which will be used in the header of all the tests. This token will be repeatedly generated within the callback of the function `beforeEach(done)`. But, to generate it, we have to delete all users first using the function `Users.destroy({where: {}})` and then, create a new one via `Users.create()` function.

We'll do the same with tasks creation, but instead of using the function `Tasks.create()` it will be used the function `Tasks.bulkCreate()` which allows sending an array of tasks to be inserted in a single execution (this function is very useful for inclusion in a plot of data).

The tasks are going to use the `user.id` field, created to ensure that they are from the authenticated user. In the end, let's use the first task created by the piece `fakeTask = tasks[0]` to reuse its `id` on the tests that need a task `id` as a route parameter. We'll generate a valid token using the function `jwt.encode({id: user.id}, jwtSecret)`.

Both the objects `fakeTask` and `token` are created in a scope above the function `beforeEach(done)`, so they can be reused on the tests. To understand in detail, you need to write the following implementation:

``` javascript
beforeEach(done => {
  Users
    .destroy({where: {}})
    .then(() => Users.create({
      name: "John",
      email: "john@mail.net",
      password: "12345"
    }))
    .then(user => {
      Tasks
        .destroy({where: {}})
        .then(() => Tasks.bulkCreate([{
          id: 1,
          title: "Work",
          user_id: user.id
        }, {
          id: 2,
          title: "Study",
          user_id: user.id
        }]))
        .then(tasks => {
          fakeTask = tasks[0];
          token = jwt.encode({id: user.id}, jwtSecret);
          done();
        });
    });
});
```

With the pre-test routine ready, we are going to write all the tasks tests, starting with the `GET /` route. On it, it is performed a request via `request.get("/tasks")` function, using also the function `set("Authorization", `JWT ${token}`)` to allows sending a header on the request, which in this case, the header `Authorization` is sent along with the value of a valid token.

To make sure the test is successfully accomplished:

1. We check the `status 200` via `expect(200)` function;
2. We apply a simple validation to ensure that the array of size 2 will be returned via `expect(res.body).to.have.length 2) ` function;
3. We compare if the titles of the first two tasks are the same as those created by the function `Tasks.bulkCreate()`;

``` javascript
describe("GET /tasks", () => {
  describe("status 200", () => {
    it("returns a list of tasks", done => {
      request.get("/tasks")
        .set("Authorization", `JWT ${token}`)
        .expect(200)
        .end((err, res) => {
          expect(res.body).to.have.length(2);
          expect(res.body[0].title).to.eql("Work");
          expect(res.body[1].title).to.eql("Study");
          done(err);
        });
    });
  });
});
```

To test the successful case of the route `POST /tasks`, there is no secret: basically is informed a header with an authentication token and a title for a new task. As result, we test if the answer returns `200` status code and if the object `req.body` has the same title as the one which was sent to register a new task.

``` javascript
describe("POST /tasks", () => {
  describe("status 200", () => {
    it("creates a new task", done => {
      request.post("/tasks")
        .set("Authorization", `JWT ${token}`)
        .send({title: "Run"})
        .expect(200)
        .end((err, res) => {
          expect(res.body.title).to.eql("Run");
          expect(res.body.done).to.be.false;
          done(err);
        });
    });
  });
});
```

Now we are going to test two simple flows of the route `GET /tasks/:id`. In the successful case we'll use the `id` of the object `fakeTask` to make sure a valid task will be returned. To test how the application behaves when a `id` of an invalid task is informed, we are going to use the function `expect(404)` to test the `status 404` that indicates if the request did not find a resource.

``` javascript
describe("GET /tasks/:id", () => {
  describe("status 200", () => {
    it("returns one task", done => {
      request.get(`/tasks/${fakeTask.id}`)
        .set("Authorization", `JWT ${token}`)
        .expect(200)
        .end((err, res) => {
          expect(res.body.title).to.eql("Work");
          done(err);
        });
    });
  });
  describe("status 404", () => {
    it("throws error when task not exist", done => {
      request.get("/tasks/0")
        .set("Authorization", `JWT ${token}`)
        .expect(404)
        .end((err, res) => done(err));
    });
  });
});
```

To finish the tests, we are going to test the successful behavior of the routes `PUT /tasks/:id` and `DELETE /tasks/:id`. Both of them will use the same functions, except that a test is going to execute the function `request.put()` and the other one is going to execute `request.delete()`. But, both of them expect that the request returns a `204` status code via `expect(204)` function.

``` javascript
describe("PUT /tasks/:id", () => {
  describe("status 204", () => {
    it("updates a task", done => {
      request.put(`/tasks/${fakeTask.id}`)
        .set("Authorization", `JWT ${token}`)
        .send({
          title: "Travel",
          done: true
        })
        .expect(204)
        .end((err, res) => done(err));
    });
  });
});
describe("DELETE /tasks/:id", () => {
  describe("status 204", () => {
    it("removes a task", done => {
      request.delete(`/tasks/${fakeTask.id}`)
        .set("Authorization", `JWT ${token}`)
        .expect(204)
        .end((err, res) => done(err));
    });
  });
});
```

We have finished the tests of tasks resources. In case you execute the command `npm test` again, you are going to see the following result:

![Testing task's resource](images/testes-de-tarefas.png)

## Testing user's endpoints

To test the user's resource is even simpler, because basically we are going to use all that were explained in the last tests. To start it, create the file `test/routes/users.js` with the following structure:

``` javascript
import jwt from "jwt-simple";

describe("Routes: Tasks", () => {
  const Users = app.db.models.Users;
  const jwtSecret = app.libs.config.jwtSecret;
  let token;
  beforeEach(done => {
    // Runs before each test...
  });
  describe("GET /user", () => {
    describe("status 200", () => {
      it("returns an authenticated user", done => {
        // Test's logic...
      });
    });
  });
  describe("DELETE /user", () => {
    describe("status 204", () => {
      it("deletes an authenticated user", done => {
        // Test's logic...
      });
    });
  });
  describe("POST /users", () => {
    describe("status 200", () => {
      it("creates a new user", done => {
        // Test's logic...
      });
    });
  });
});
```

The pre-test logic is going to be simplified, however it will have the generation of a valid token. See below how to implement the `beforeEach(done)` function:

``` javascript
beforeEach(done => {
  Users
    .destroy({where: {}})
    .then(() => Users.create({
      name: "John",
      email: "john@mail.net",
      password: "12345"
    }))
    .then(user => {
      token = jwt.encode({id: user.id}, jwtSecret);
      done();
    });
});
```

Now, to implement these tests, let's get starting to test the `GET /user` route, which must returns the authenticated user’s data which basically sends a token and receive as a response the user’s data that were created from `beforeEach(done)` function.

``` javascript
describe("GET /user", () => {
  describe("status 200", () => {
    it("returns an authenticated user", done => {
      request.get("/user")
        .set("Authorization", `JWT ${token}`)
        .expect(200)
        .end((err, res) => {
          expect(res.body.name).to.eql("John");
          expect(res.body.email).to.eql("john@mail.net");
          done(err);
        });
    });
  });
});
```

Then, let's write the tests to the `DELETE /user` route. In this case, is very simple, you just need to send a token and wait for the `204` status code.

``` javascript
describe("DELETE /user", () => {
  describe("status 204", () => {
    it("deletes an authenticated user", done => {
      request.delete("/user")
        .set("Authorization", `JWT ${token}`)
        .expect(204)
        .end((err, res) => done(err));
    });
  });
});
```

To finish the last test, we are going to implement the test for new user's route. This one, doesn't require a token, after all, it is an open route for new users register an account in the API. See below this test’s code:

``` javascript
describe("POST /users", () => {
  describe("status 200", () => {
    it("creates a new user", done => {
      request.post("/users")
        .send({
          name: "Mary",
          email: "mary@mail.net",
          password: "12345"
        })
        .expect(200)
        .end((err, res) => {
          expect(res.body.name).to.eql("Mary");
          expect(res.body.email).to.eql("mary@mail.net");
          done(err);
        });
    });
  });
});
```

Now, if you execute the command `npm test` again, you'll see a report like this:

![Testing user’s resource](images/testes-de-usuario.png)

### Conclusion

If you reached this step, then you have developed a small but powerful API using Node.js and SQL database. Everything is already working and was tested to ensure the code quality in the project.

In the next chapter, we'll use a very useful tool for generating API documentation. Keep reading because there are a lot of cool stuff to explore!
