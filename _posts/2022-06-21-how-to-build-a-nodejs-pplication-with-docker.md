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
  $ nano package.json
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
> ~/node_project/app.js

```js
...

app.use(express.static(path));
app.use('/', router);

app.listen(port, function () {
  console.log('Example app listening on port 8080!')
})
```
The finished `app.js` file includes all of the following lines of code:
> ~/node_project/app.js

```js
const express = require('express');
const app = express();
const router = express.Router();

const path = __dirname + '/views/';
const port = 8080;

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

app.use(express.static(path));
app.use('/', router);

app.listen(port, function () {
  console.log('Example app listening on port 8080!')
})
```
Save and close the file when you are finished.

Next, add some static content to the application. Start by creating the `views` directory:
```bash
$ mkdir views
```
Open the landing page file, `index.html`:
```bash
$ nano views/index.html
```
Add the following code to the file, which will import Boostrap and create a jumbotron component with a link to the more detailed `sharks.html` info page:
> ~/node_project/views/index.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <title>About Sharks</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">
    <link href="css/styles.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css?family=Merriweather:400,700" rel="stylesheet" type="text/css">
</head>

<body>
    <nav class="navbar navbar-dark bg-dark navbar-static-top navbar-expand-md">
        <div class="container">
            <button type="button" class="navbar-toggler collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false"> <span class="sr-only">Toggle navigation</span>
            </button> <a class="navbar-brand" href="#">Everything Sharks</a>
            <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
                <ul class="nav navbar-nav mr-auto">
                    <li class="active nav-item"><a href="/" class="nav-link">Home</a>
                    </li>
                    <li class="nav-item"><a href="/sharks" class="nav-link">Sharks</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
    <div class="jumbotron">
        <div class="container">
            <h1>Want to Learn About Sharks?</h1>
            <p>Are you ready to learn about sharks?</p>
            <br>
            <p><a class="btn btn-primary btn-lg" href="/sharks" role="button">Get Shark Info</a>
            </p>
        </div>
    </div>
    <div class="container">
        <div class="row">
            <div class="col-lg-6">
                <h3>Not all sharks are alike</h3>
                <p>Though some are dangerous, sharks generally do not attack humans. Out of the 500 species known to researchers, only 30 have been known to attack humans.
                </p>
            </div>
            <div class="col-lg-6">
                <h3>Sharks are ancient</h3>
                <p>There is evidence to suggest that sharks lived up to 400 million years ago.
                </p>
            </div>
        </div>
    </div>
</body>

</html>
```
The following is a quick break-down of the various elements in your `index.html` file. The top-level navbar allows users to toggle between the **Home** and **Sharks pages**. In the navbar-nav subcomponent, you are using Bootstrap’s active class to indicate the current page to the user. You’ve also specified the routes to your static pages, which match the routes you defined in `app.js`:
> ~/node_project/views/index.html

```html
...
<div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
   <ul class="nav navbar-nav mr-auto">
      <li class="active nav-item"><a href="/" class="nav-link">Home</a>
      </li>
      <li class="nav-item"><a href="/sharks" class="nav-link">Sharks</a>
      </li>
   </ul>
</div>
...
```
Additionally, you’ve created a link to your shark information page in your jumbotron’s button:
> ~/node_project/views/index.html

```html
...
<div class="jumbotron">
   <div class="container">
      <h1>Want to Learn About Sharks?</h1>
      <p>Are you ready to learn about sharks?</p>
      <br>
      <p><a class="btn btn-primary btn-lg" href="/sharks" role="button">Get Shark Info</a>
      </p>
   </div>
</div>
...
```
There is also a link to a custom style sheet in the header:
> ~/node_project/views/index.html

```html
...
<link href="css/styles.css" rel="stylesheet">
...
```
You will create this style sheet at the end of this step.

Save and close the file when you are finished.

With the application landing page in place, you can create your shark information page, sharks.html, which will offer interested users more information about `sharks`.

Open the file:
```bash
$ nano views/sharks.html
```
Add the following code, which imports Bootstrap, the custom style sheet, and detailed information about certain sharks:
> ~/node_project/views/sharks.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <title>About Sharks</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">
    <link href="css/styles.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css?family=Merriweather:400,700" rel="stylesheet" type="text/css">
</head>
<nav class="navbar navbar-dark bg-dark navbar-static-top navbar-expand-md">
    <div class="container">
        <button type="button" class="navbar-toggler collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false"> <span class="sr-only">Toggle navigation</span>
        </button> <a class="navbar-brand" href="/">Everything Sharks</a>
        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
            <ul class="nav navbar-nav mr-auto">
                <li class="nav-item"><a href="/" class="nav-link">Home</a>
                </li>
                <li class="active nav-item"><a href="/sharks" class="nav-link">Sharks</a>
                </li>
            </ul>
        </div>
    </div>
</nav>
<div class="jumbotron text-center">
    <h1>Shark Info</h1>
</div>
<div class="container">
    <div class="row">
        <div class="col-lg-6">
            <p>
                <div class="caption">Some sharks are known to be dangerous to humans, though many more are not. The sawshark, for example, is not considered a threat to humans.
                </div>
                <img src="https://assets.digitalocean.com/articles/docker_node_image/sawshark.jpg" alt="Sawshark">
            </p>
        </div>
        <div class="col-lg-6">
            <p>
                <div class="caption">Other sharks are known to be friendly and welcoming!</div>
                <img src="https://assets.digitalocean.com/articles/docker_node_image/sammy.png" alt="Sammy the Shark">
            </p>
        </div>
    </div>
</div>

</html>
```
Note that in this file, you once again use the `active` Bootstrap class to indicate the current page.

Save and close the file when you are finished.

Finally, create the custom CSS style sheet that you’ve linked to in `index.html` and `sharks.html` by creating a `css` folder in the `views` directory:
```bash
$ mkdir views/css
```
Open the style sheet:
```bash
$ nano views/css/styles.css
```
Add the following code, which will set the desired color and font for your pages:
> ~/node_project/views/css/styles.css

```css
.navbar {
	margin-bottom: 0;
}

body {
	background: #020A1B;
	color: #ffffff;
	font-family: 'Merriweather', sans-serif;
}

h1,
h2 {
	font-weight: bold;
}

p {
	font-size: 16px;
	color: #ffffff;
}

.jumbotron {
	background: #0048CD;
	color: white;
	text-align: center;
}

.jumbotron p {
	color: white;
	font-size: 26px;
}

.btn-primary {
	color: #fff;
	text-color: #000000;
	border-color: white;
	margin-bottom: 5px;
}

img,
video,
audio {
	margin-top: 20px;
	max-width: 80%;
}

div.caption: {
	float: left;
	clear: both;
}
```
In addition to setting font and color, this file also limits the size of the images by specifying a `max-width` of 80%. This will prevent the pictures from taking up more room than you would like on the page.

Save and close the file when you are finished.

With the application files in place and the project dependencies installed, you are ready to start the application.

If you followed the initial server setup tutorial in the prerequisites, you will have an active firewall permitting only SSH traffic. To permit traffic to port `8080` run:
```bash
$ sudo ufw allow 8080
```
To start the application, make sure that you are in your project’s root directory:
```bash
$ cd ~/node_project
```
Start the application with node app.js:
```bash
$ node app.js
```
Navigate your browser to `http://your_server_ip:8080`. The following is your landing page:
![Everything Sharks](/assets/img/blog/landing_page.png)
Clicking on the **Get Shark Info** button will take you to the following information page:
![Everything Sharks-Info](/assets/img/blog/sharks.png)
You now have an application up and running. When you are ready, quit the server by typing CTRL + C. You can now move on to creating the Dockerfile that will allow you to recreate and scale this application as desired.

### Step 3 — Writing the Dockerfile

Your Dockerfile specifies what will be included in your application container when it is executed. Using a Dockerfile allows you to define your container environment and avoid discrepancies with dependencies or runtime versions.

Following [these guidelines on building optimized containers](https://www.digitalocean.com/community/tutorials/building-optimized-containers-for-kubernetes), will make your image as efficient as possible by minimizing the number of image layers and restricting the image’s function to a single purpose — recreating your application files and static content.

In your project’s root directory, create the Dockerfile:
```bash
$ nano Dockerfile
```
Docker images are created using a succession of layered images that build on one another. The first step will be to add the base image for your application that will form the starting point of the application build.

You can use the [node:10-alpine image](https://hub.docker.com/_/node/). The `alpine` image is derived from the [Alpine Linux](https://alpinelinux.org/) project, and will help keep your image size down. For more information about whether or not the alpine image is the right choice for your project, please see the full discussion under the **Image Variants** section of the [Docker Hub Node image page](https://hub.docker.com/_/node/).

Add the following FROM instruction to set the application’s base image:
> ~/node_project/Dockerfile

```
FROM node:10-alpine
```
This image includes Node.js and npm. Each Dockerfile must begin with a `FROM` instruction.

By default, the Docker Node image includes a non-root **node** user that you can use to avoid running your application container as **root**. It is a recommended security practice to avoid running containers as **root** and to [restrict capabilities within the container](https://docs.docker.com/engine/security/security/#linux-kernel-capabilities) to only those required to run its processes. We will therefore use the **node** user’s home directory as the working directory for our application and set them as our user inside the container. For more information about best practices when working with the Docker Node image, see this [best practices guide](https://github.com/nodejs/docker-node/blob/master/docs/BestPractices.md).

To fine-tune the permissions on your application code in the container, create the `node_modules` subdirectory in `/home/node` along with the `app` directory. Creating these directories will ensure that they have the correct permissions, which will be important when you create local node modules in the container with `npm install`. In addition to creating these directories, set ownership on them to your **node** user:
> ~/node_project/Dockerfile

```
...
RUN mkdir -p /home/node/app/node_modules && chown -R node:node /home/node/app
```
For more information on the utility of consolidating `RUN` instructions, see this [discussion of how to manage container layers](https://www.digitalocean.com/community/tutorials/building-optimized-containers-for-kubernetes#managing-container-layers).

Next, set the working directory of the application to `/home/node/app`:
> ~/node_project/Dockerfile

```
...
WORKDIR /home/node/app
```
If a `WORKDIR` isn’t set, Docker will create one by default, so it’s a good idea to set it explicitly.

Next, copy the `package.json` and `package-lock.json` (for npm 5+) files:
```
...
COPY package*.json ./
```
Adding this `COPY` instruction before running `npm install` or copying the application code allows you to take advantage of Docker’s caching mechanism. At each stage in the build, Docker will check to see if it has a layer cached for that particular instruction. If you change the `package.json`, this layer will be rebuilt, but if you don’t, this instruction will allow Docker to use the existing image layer and skip reinstalling your node modules.

To ensure that all of the application files are owned by the non-root **node** user, including the contents of the `node_modules` directory, switch the user to **node** before running `npm install`:
> ~/node_project/Dockerfile

```
...
USER node
```
After copying the project dependencies and switching the user, run `npm install`:
> ~/node_project/Dockerfile

```
...
RUN npm install
```
Next, copy your application code with the appropriate permissions to the application directory on the container:
> ~/node_project/Dockerfile

```
...
COPY --chown=node:node . .
```
This will ensure that the application files are owned by the non-root **node** user.

Finally, expose port `8080` on the container and start the application:
> ~/node_project/Dockerfile

```
...
EXPOSE 8080

CMD [ "node", "app.js" ]
```
`EXPOSE` does not publish the port, but instead functions as a way of documenting which ports on the container will be published at runtime. CMD runs the command to start the application — in this case, [node app.js](https://github.com/nodejs/docker-node/blob/master/docs/BestPractices.md#cmd).

>Note: There should only be one CMD instruction in each Dockerfile. If you include more than one, only the last will take effect.

There are many things you can do with the Dockerfile. For a complete list of instructions, please refer to Docker’s [Dockerfile reference documentation](https://docs.docker.com/engine/reference/builder/).
This is the complete Dockerfile:
> ~/node_project/Dockerfile

```
FROM node:10-alpine

RUN mkdir -p /home/node/app/node_modules && chown -R node:node /home/node/app

WORKDIR /home/node/app

COPY package*.json ./

USER node

RUN npm install

COPY --chown=node:node . .

EXPOSE 8080

CMD [ "node", "app.js" ]
```
Save and close the file when you are finished editing.

Before building the application image, add a [.dockerignore file](https://docs.docker.com/engine/reference/builder/#dockerignore-file). Working in a similar way to a [.gitignore file](https://git-scm.com/docs/gitignore), .dockerignore specifies which files and directories in your project directory should not be copied over to your container.

Open the `.dockerignore` file:
``` bash
$ nano .dockerignore
```
Inside the file, add your local node modules, npm logs, Dockerfile, and `.dockerignore` file:
> ~/node_project/.dockerignore

```
node_modules
npm-debug.log
Dockerfile
.dockerignore
```
If you are working with [Git](https://git-scm.com/) then you will also want to add your .git directory and .gitignore file.

Save and close the file when you are finished.

You are now ready to build the application image using the [docker build](https://docs.docker.com/engine/reference/commandline/build/) command. Using the `-t` flag with docker build will allow you to tag the image with a memorable name. Because you’re going to push the image to Docker Hub, include your Docker Hub username in the tag. You can tag the image as `nodejs-image-demo`, but feel free to replace this with a name of your own choosing. Remember to also replace `your_dockerhub_username` with your own Docker Hub username:
```bash
$ docker build -t your_dockerhub_username/nodejs-image-demo .
```
The . specifies that the build context is the current directory.

It will take some time to build the image. Once it is complete, check your images:
```bash
$ docker images
```
> Output

```
REPOSITORY                                         TAG                 IMAGE ID            CREATED             SIZE
your_dockerhub_username/nodejs-image-demo          latest              1c723fb2ef12        8 seconds ago       73MB
node                                               10-alpine           f09e7c96b6de        13 monthss ago      82.7MB
```
It is now possible to create a container with this image using docker run. You will include three flags with this command:

- -p: This publishes the port on the container and maps it to a port on your host. You can use port 80 on the host, but you should feel free to modify this as necessary if you have another process running on that port. For more information about how this works, see this discussion in the Docker docs on port binding.
- -d: This runs the container in the background.
- --name: This allows you to give the container a memorable name.

Run the following command to build the container:
```bash
$ docker run --name nodejs-image-demo -p 80:8080 -d your_dockerhub_username/nodejs-image-demo
```
Once your container is up and running, you can inspect a list of your running containers with [docker ps](https://docs.docker.com/engine/reference/commandline/ps/):
```bash
$ docker ps
```
>Output

```
CONTAINER ID   IMAGE                        COMMAND                  CREATED          STATUS          PORTS                                   NAMES
e50ad27074a7   your_dockerhub_username/nodejs-image-demo    "docker-entrypoint.s…"   13 seconds ago   Up 12 seconds   0.0.0.0:80->8080/tcp, :::80->8080/tcp   nodejs-image-demo
```
With your container running, you can now visit your application by navigating your browser to `http://your_server_ip`:
Navigate your browser to `http://your_server_ip:8080`. The following is your landing page:
![Everything Sharks](/assets/img/blog/landing_page.png)
Now that you have created an image for your application, you can push it to Docker Hub for future use.

### Step 4 — Using a Repository to Work with Images

By pushing your application image to a registry like Docker Hub, you make it available for subsequent use as you build and scale your containers. To demonstrate how this works, you will push the application image to a repository and then use the image to recreate your container.

The first step to pushing the image is to log in to the Docker Hub account you created in the prerequisites:
```bash
docker login -u your_dockerhub_username 
```
When prompted, enter your Docker Hub account password. Logging in this way will create a `~/.docker/config.json` file in your user’s home directory with your Docker Hub credentials.

You can now push the application image to Docker Hub using the tag you created earlier, `your_dockerhub_username/nodejs-image-demo`:
```bash
$ docker push your_dockerhub_username/nodejs-image-demo
```
Let’s test the utility of the image registry by destroying our current application container and image and rebuilding them with the image in our repository.

First, list your running containers:
```
$ docker ps
```
> Output

```
CONTAINER ID   IMAGE                        COMMAND                  CREATED         STATUS         PORTS                                   NAMES
e50ad27074a7   your_dockerhub_username/nodejs-image-demo   "docker-entrypoint.s…"   5 minutes ago   Up 5 minutes   0.0.0.0:80->8080/tcp, :::80->8080/tcp   nodejs-image-demo
```
Using the CONTAINER ID listed in your output, stop the running application container. Be sure to replace the highlighted ID below with your own `CONTAINER ID`:
```bash
$ docker stop e50ad27074a7
```
List your all of your images with the -a flag:
```bash
$ docker images -a
```
The following is output with the name of your image, `your_dockerhub_username/nodejs-image-demo`, along with the `node` image and the other images from your build:
> Output

```
REPOSITORY                                           TAG                 IMAGE ID            CREATED             SIZE
your_dockerhub_username/nodejs-image-demo            latest              1c723fb2ef12        7 minutes ago       73MB
<none>                                               <none>              2e3267d9ac02        4 minutes ago       72.9MB
<none>                                               <none>              8352b41730b9        4 minutes ago       73MB
<none>                                               <none>              5d58b92823cb        4 minutes ago       73MB
<none>                                               <none>              3f1e35d7062a        4 minutes ago       73MB
<none>                                               <none>              02176311e4d0        4 minutes ago       73MB
<none>                                               <none>              8e84b33edcda        4 minutes ago       70.7MB
<none>                                               <none>              6a5ed70f86f2        4 minutes ago       70.7MB
<none>                                               <none>              776b2637d3c1        4 minutes ago       70.7MB
node                                                 10-alpine           f09e7c96b6de        13 months ago       82.7MB
```
Remove the stopped container and all of the images, including unused or dangling images, with the following command:
```
docker system prune -a
```
Type `y` when prompted in the output to confirm that you would like to remove the stopped container and images. Be advised that this will also remove your build cache.

You have now removed both the container running your application image and the image itself. For more information on removing Docker containers, images, and volumes, please see [How To Remove Docker Images, Containers, and Volumes](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes).

With all of your images and containers deleted, you can now pull the application image from Docker Hub:
```bash
$ docker pull your_dockerhub_username/nodejs-image-demo
```
List your images once again:
```
$ docker images
```
You will see your application image:

> Output

```
REPOSITORY                                     TAG                 IMAGE ID            CREATED             SIZE
your_dockerhub_username/nodejs-image-demo      latest              1c723fb2ef12        11 minutes ago      73MB
```
You can now rebuild your container using the command from Step 3:
```bash
$ docker run --name nodejs-image-demo -p 80:8080 -d your_dockerhub_username/nodejs-image-demo
```
List your running containers:
```bash
$ docker ps
```
> Output

```
CONTAINER ID   IMAGE                        COMMAND                  CREATED         STATUS         PORTS                                   NAMES
f6bc2f50dff6   your_dockerhub_username/nodejs-image-demo   "docker-entrypoint.s…"   6 seconds ago   Up 5 seconds   0.0.0.0:80->8080/tcp, :::80->8080/tcp   nodejs-image-demo
```
Visit `http://your_server_ip` once again to view your running application.

### Conclusion

In this tutorial you created a static web application with Express and Bootstrap, as well as a Docker image for this application. You used this image to create a container and pushed the image to Docker Hub. From there, you were able to destroy your image and container and recreate them using your Docker Hub repository.

If you are interested in learning more about how to work with tools like Docker Compose and Docker Machine to create multi-container setups, you can look at the following guides:

- [How To Install Docker Compose on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-docker-compose-on-ubuntu-18-04).
- [How To Provision and Manage Remote Docker Hosts with Docker Machine on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-provision-and-manage-remote-docker-hosts-with-docker-machine-on-ubuntu-18-04).

For general tips on working with container data, see:

- [How To Share Data between Docker Containers](https://www.digitalocean.com/community/tutorials/how-to-share-data-between-docker-containers).
- [How To Share Data Between the Docker Container and the Host](https://www.digitalocean.com/community/tutorials/how-to-share-data-between-the-docker-container-and-the-host).

If you are interested in other Docker-related topics, please see our complete library of [Docker tutorials](https://www.digitalocean.com/community/tags/docker).

[Docker]: https://www.docker.com/
[Express]: https://expressjs.com/
[Bootstrap]: https://getbootstrap.com/
[Docker Hub]: https://hub.docker.com/
[Docker Hub account]: https://docs.docker.com/docker-hub/
[packagejson]: https://docs.npmjs.com/cli/v8/configuring-npm/package-json
[npm]: https://www.npmjs.com/
[MIT licence]: https://opensource.org/licenses/MIT
[middleware fun]: https://expressjs.com/en/guide/writing-middleware.html