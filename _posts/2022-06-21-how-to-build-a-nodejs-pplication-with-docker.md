---
layout: post
title: "How To Build a Node.js Application with Docker"
date: 2022-06-21
category:
  - NodeJS
  - Ubuntu
image: assets/img/blog/nodejs-with-docker.png
author: Andy
tags: Ubuntu Applicaions NodeJS Docker
---
#### Introduction

The [Docker][Docker] platform allows developers to package and run applications as containers. A container is an isolated process that runs on a shared operating system, offering a lighter weight alternative to virtual machines. Though containers are not new, they offer a number of benefits — including process isolation and environment standardization — that are growing in importance as more developers use distributed application architectures.

When building and scaling an application with Docker, the starting point is typically creating an image for your application, which you can then run in a container. The image includes your application code, libraries, configuration files, environment variables, and runtime. Using an image ensures that the environment in your container is standardized and contains only what is necessary to build and run your application.

In this tutorial, you will create an application image for a static website that uses the [Express][Express] and [Bootstrap][Bootstrap] frameworks. You will then build a container using that image and push it to [Docker Hub][Docker Hub] for future use. Finally, you will pull the stored image from your Docker Hub repository and build another container, demonstrating how you can recreate and scale your application.

#### Prerequisites

To follow this tutorial, you will need:

- One Ubuntu 22.04 server, set up following this Initial Server Setup guide.
- Docker installed on your server, following Steps 1 and 2 of How To Install and Use Docker on Ubuntu 22.04.
- Node.js and npm installed, following [these instructions on installing with the PPA managed by NodeSource]({% link _posts/2022-02-08-nodejs-on-ubuntu-22-04.md %}).
- A Docker Hub account. For an overview of how to set this up, refer to [this introduction][Docker Hub account] on getting started with Docker Hub.

#### Step 1 — Installing Your Application Dependencies

To create your image, you will first need to make your application files, which you can then copy to your container. These files will include your application’s static content, code, and dependencies.

Create a directory for your project in your non-root user’s home directory. The directory in this example named `node_project`, but you should feel free to replace this with something else:
```bash
  $ mkdir node_project
```
Navigate to this directory:
```bash
  $ cd node_project
```
This will be the root directory of the project.

Next, create a [`package.json`][packagejson] file with your project's dependencies and other identifying information. Open the file with `nano` or your favorite editor:
```bash
  $nano package.json
```
Add the followind information about the project, insluding its name, author, license, entrypoint, and dependencies. Be sure to replace the author information with your own neme and contact details:

> ~/node_project/package.json

```json
{
  "name": "nodejs-image-demo",
  "version": "1.0.0",
  "description": "nodejs image demo",
  "author": "Sammy the Shark <sammy@example.com>",
  "license": "MIT",
  "main": "app.js",
  "keywords": [
    "nodejs",
    "bootstrap",
    "express"
  ],
  "dependencies": {
    "express": "^4.16.4"
  }
}
```
This file includes the project name, author, and license under which it is being shared. npm [recommends][packagejson] making your project name short and descriptive, and avoiding duplicates in the [npm registry][npm]. This example lists the [MIT license][MIT licence] in the license field, permitting the free use and distribution of the application code.

Additionally, the file specifies:

- "main": The entrypoint for the application, app.js. You will create this file next.
- "dependencies": The project dependencies — in this case, Express 4.16.4 or above.

Though this file does not list a repository, you can add one by following these guidelines on [adding a repository to your *package.json* file][packagejson]. This is a good addition if you are versioning your application.

Save and close the file by typing `CTRL + X`. Press Y and then ENTER to confirm your changes.

To install your project’s dependencies, run the following command:
```bash
  $ npm install
```
This will install the packages you've listed in your [`package.json`][packagejson] file in your project directory.
You can now move on to building the application files.

#### Step 2 — Creating the Application Files

You will create a website that offers users information about sharks. This application will have a main entrypoint, `app.js`, and a `views` directory that will include the project’s static assets. The landing page, `index.html`, will offer users some preliminary information and a link to a page with more detailed shark information, `sharks.html`. In the views directory, you will create both the landing page and `sharks.html`.

First, open `app.js` in the main project directory to define the project’s routes:
```bash
  $ nano app.js
```
In the first part of this file, create the Express application an Router objects and define the base directory and port as constants:

> ~/node_project/app.js

```js
const express = require('express');
const app = express();
const router = express.Router();

const path = __dirname + '/views/';
const port = 8080;
```
The `require` function loads the `express` module, which is used to create the `app` and `router` objects. The `router` object will perform the routing function of the application, and as you define HTTP method routes you will add them to this object to define how your application will handle requests.

This section of the file also sets a couple of constants, `path` and `port`:

- `path`: Defines the base directory, which will be the views subdirectory within the current project directory.
- `port`: Tells the app to listen on and bind to port `8080`.

Next, set the routes for the application using the router object:
> ~/node_project/app.js

```js
...

router.use(function (req,res,next) {
  console.log('/' + req.method);
  next();
});

router.get('/', function(req,res){
  res.sendFile(path + 'index.html');
});

router.get('/sharks', function(req,res){
  res.sendFile(path + 'sharks.html');
});
```
The router.use function loads a [middleware function][middleware fun] that will log the router’s requests and pass them on to the application’s routes. These are defined in the subsequent functions, which specify that a `GET` request to the base project URL should return the `index.html` page, while a `GET` request to the `/sharks` route should return `sharks.html`.

Finally, mount the `router` middleware and the application’s static assets and tell the app to listen on port `8080`:





[Docker]: https://www.docker.com/
[Express]: https://expressjs.com/
[Bootstrap]: https://getbootstrap.com/
[Docker Hub]: https://hub.docker.com/
[Docker Hub account]: https://docs.docker.com/docker-hub/
[packagejson]: https://docs.npmjs.com/cli/v8/configuring-npm/package-json
[npm]: https://www.npmjs.com/
[MIT licence]: https://opensource.org/licenses/MIT
[middleware fun]: https://expressjs.com/en/guide/writing-middleware.html