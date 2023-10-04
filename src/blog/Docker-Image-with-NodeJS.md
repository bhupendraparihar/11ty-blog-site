---
title: Docker Image with NodeJS
author: Bhupendra Parihar
date: 2023-10-04 16:15:32
tags: ['post', 'featured']
image: /assets/blog/code3.jpg
description: How to create a docker image from a node js project and run it as a docker container
---

## The first question you should ask is why docker?

Docker provides a containerized environment to run your application. The benefit of doing so is ease of deployment, it requires minimum resources, and it is very useful in creating replicas in case of horizontal scaling(load balancing).

## How do you install docker?

You can easily install docker, by download Docker Desktop for your operating system. Follow the installer, and this step will also automatically installs the docker-cli, which we are going to use.

## Steps to create a docker image

- Create a NodeJs project or you can choose the existing one as well. For this example I am using a super simple express server application.

So, Lets create a folder and then create a package.json file.

```json
{
  "name": "simple-web",
  "dependencies": {
    "express": "*"
  },
  "scripts": {
    "start": "node index.js"
  }
}
```

Then create a index.js file

```javascript
const express = require('express');

const app = express();

app.get('/', (req, res) => {
  res.send('Hi There' + new Date());
});

app.listen(8080, () => {
  console.log('Listening on port 5000');
});
```

In a nutshell, the above application is creating a express server on default url and return the 'Hi There text with current date'. The application is listening on port 5000. So if you open your browser and then run localhost:5000, you will see the output.

## Create a Dockerfile for image creation

- Create a file named Dockerfile. Remember this file does not have any extension.

- The first line in your Dockerfile is to provide the base image

  ```
  FROM node:alpine3.17
  ```

- Select the working directory to work with, so that the source code of your project may not touch or effect any other container specific file.

  ```
  WORKDIR /usr/app
  ```

- Copy the package.json into the working directory

  ```
  COPY ./pakage.json ./
  ```

- Install the dependencies

  ```
  RUN npm install
  ```

- Copy the remaining files and content to the working directory

  ```
  COPY ./ ./
  ```

- Provide the default command to run when the image is being used
  ```
  CMD ["npm", "start"]
  ```

The overall content will look like this

```
FROM node:alpine3.17

WORKDIR /usr/app

# install some dependencies

COPY ./package.json ./
RUN npm install
COPY ./ ./

CMD ["npm", "start"]

```

## Lets create the image

Go to the terminal

```
docker build -t jforjs/simple-webapp .
```

jforjs/simple-webapp is a tag name for the image generated.

## Run the docker container with the image created

```
docker run -p 8080:5000 jforjs/simple-webapp
```

You can open the browser and check by hitting the url localhost:8080, remember port 5000 is the port used inside the container, and we are connecting to that port by our port 8080.

## How to check what is inside my container

```
docker exec -it jforjs/simple-webapp sh
```

Hope you enjoyed the article.
