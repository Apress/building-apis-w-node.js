# Introduction to Node.js

## What's Node.js?

Node.js is a low level and a highly scalable platform, it was created to be explicitly an experiment in asynchronous processing. Using Node.js you will program directly with many network and internet protocols or use libraries that have access to OS resources. To program in Node.js you only need to master JavaScript language – that’s right, only JavaScript! And the JavaScript runtime used in this platform is the famous Javascript V8, which is also used in Google Chrome.

## Main features

### Single-Thread

A single-thread architecture works with only the main thread instance for each initiated process. If you are used to programming in multi-thread architecture, like Java or .NET to use the concurrency processing, unfortunately it won't be possible with Node.js. However, you can works with parallel processing using multiple process.

For example, you can use a native library from Node.js called `clusters` that allows you to implement a network of multiple processes. In Node.js you can create **N-1 processes** to work with parallel processing, while the main process (aka master process) works on balancing the load among the other processes (aka slave process). If your server has multiple cores, applying this technique will optimize the processing's usage.

Another way is to use asynchronous programming, which is one of the main resources from JavaScript. The asynchronous functions on Node.js work as **Non-blocking I/O**, in case your application has to read a huge file it will not block the CPU, because the IO operation runs out of a thread pool, allowing your application to continue to process other tasks requested by other users.

### Event-Loop

Node.js has **event-oriented paradigm**. It follows the same philosophy of event-oriented on JavaScript client-side, the only difference is the type of event, meaning that there are not events such as mouse's `click`, `key up` from the keyboard or any other events from web browsers. In fact, Node.js works with **I/O events**, for example: `connect` from a database, `open` a file or read `data` from a streaming and many others.

![The event-loop process](images/event-loop.png)

The Event-Loop is the responsible agent in noticing and emitting events. Simply speaking, it is an **infinite loop** which in each iteration, it verifies in the event queue if some event was triggered. When an event is triggered, the Event-Loop execute it and send it to the queue of executed events. When an event is running, we can write any business logic on it inside the callback function commonly used in JavaScript.

## Why do I need to learn Node.js?

* **JavaScript everywhere:** Node.js uses JavaScript as a server-side programming language. This feature allows you to cut short your learning process, after all, the language is the same as JavaScript client-side. Your main challenge in this platform is to learn how to works with the asynchronous programming to take most of this technique. Another advantage of working with JavaScript is that you can keep an easy maintenance project – of course, as long as you know who to write codes using JavaScript.
You are going to easily find professionals for your projects and you’re going to spend less time studying a new server-side language. A technical JavaScript advantage in comparison with other back-end languages is that you will no longer use specifical frameworks to parse JSON objects (JavaScript Object Notation); after all, the JSON from the client-side will be same as in the server-side. There are also cases of applications running databases that persists JSON objects, such as the popular NoSQL: MongoDB, CouchDB and Riak. An important detail is that the Node.js uses many functionalities from ECMAScript 6, allowing you write an elegant and robust JavaScript code.

* **Active community:** This is one of the biggest advantages in Node.js. Currently, there are many communities around the world striving to make Node.js a huge platform, either writing posts and tutorials, giving speeches in some events or publishing and maintaining new modules.

* **Active open-source modules:** Today there are +300.000 open-source modules published into NPM which can be used instantly for Node.js projects.

* **Ready to realtime:** Node.js have become popular thanks to some frameworks for realtime communications between client and server. The **SockJS** and **Socket.IO** are good examples. They are compatible with the protocol **WebSockets** and they allow you to traffic data through a single bidirectional connection, treating all data by JavaScript events.

* **Big players:** LinkedIn, Wallmart, Groupon, Microsoft, Netflix, Uber and Paypal are some of the big companies using Node.js and there are many more.

### Conclusion

Up to this point, all the theory was explained: main concepts and advantages about why use Node.js. In the next chapters, we are going to see how it works on practice, with a single condition! Open your mind to the new and read this book with excitement to enjoy this book!
