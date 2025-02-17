Getting an application running on Crane Cloud is essentially working out how to package it as a deployable image. Once packaged it can be deployed to the Crane Cloud platform.

In this guide, we will learn how to containerize a sample HTML Application using Docker and then deploy to Crane Cloud.

## Prerequisites

Before we begin, we assume that you already have the following:

1. Fork <a href="https://github.com/crane-cloud-Apps/sample-html-app" target="_blank" rel="noopener noreferrer">sample-html-app on Github.</a>

2. Docker installed on your machine. If you haven't got Docker installed, you can use <a href="https://docs.docker.com/get-docker/" target="_blank" rel="noopener noreferrer">this resource</a> to get you up and running.

3. Docker Hub account. If you need to create one first, <a href="https://hub.docker.com/signup" target="_blank" rel="noopener noreferrer">Click here</a>

## Create a Dockerfile for your app

In our application's directory, we shall create a `Dockerfile` and in it, let us write the following commands

```js
# Use an official nginx image as the base image
FROM nginx:alpine

# Copy the HTML files into the container
COPY . /usr/share/nginx/html

# Expose port 80 so that it can be accessed outside of the container
EXPOSE 80
```

## Build a Docker Image for your app

You must note that the Dockerfile is like the heart of our dockerization process, without it we cannot build a Docker image for our application. 

We can therefore build our docker image by running this command `docker build -t <yourDockerhubUsername>/html-app .`

- `<yourDockerhubUsername>/html-app` is the docker image name I chose for this guide, whereby `<yourDockerHubUsername>` should be replaced by your DockerHub account username.

## Run our Docker Image locally

To confirm that the image was successfully built, run `docker run -p 8080:80 <yourDockerhubUsername>/html-app`. This will run our containerized app locally on `port 8080`.

- The `-p` flag redirects a public port to a private port inside the container and the private port in this case is the port value `80` we exposed in our Dockerfile while building the image.

Next, we are going to push our local image to DockerHub so that we can access our application remotely and deploy it on Crane Cloud.

## Push local Docker Image to Docker Hub

We also need to access DockerHub in order to push our local image to our newly created project.

Inorder to do that, run `docker login` and authenticate with your DockerHub credentials

Before we can push the image, we need to tag it. Docker tags are a way of referring to your image that conveys useful information about its version. This is done in the format `docker tag <local-image-name> <yourDockerhubUsername>/<project-name>:<version>`

In our case, this will be `docker tag <yourDockerhubUsername>/html-app <yourDockerhubUsername>/html-app:latest`

- `<yourDockerhubUsername>/html-app` is the DockerHub repository I have created and after it is a tag `latest`

After adding a tag to your image, you can proceed to push your local image along with its tag to DockerHub using the command `docker push <yourDockerhubUsername>/html-app:latest`

## Deploy to Crane Cloud

- Login to your Crane Cloud account, incase you do not have one you can simply create one by visiting [here](../authentication/registration.md)

- From your dashboard create a project, you can see how its done [here](../projects/projects.md)

- Inside that project, you create an app called `my-html-app` as illustrated below

![](../img/deploy_html.png)

After providing the above information, we can then deploy our app by clicking the `DEPLOY` button.

Thank you for following this guide.
