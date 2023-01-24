Getting an application running on Crane Cloud is essentially working out how to package it as a deployable image. Once packaged it can be deployed to the Crane Cloud platform.

In this guide, we will learn how to containerize a sample flask Application using Docker and then deploy to Crane Cloud. See [demo here](https://my-flask-app-595ccbea-3653-495f-a047.cranecloud.io/)

## Prerequisites

Before we begin, we assume that you already have the following:

1. Fork [sample-flask-app](https://github.com/crane-cloud-Apps/sample-flask-app) on Github.

2. Docker installed on your machine. If you haven't got Docker installed, you can use [this resource](https://docs.docker.com/get-docker/) to get you up and running.

3. Docker Hub account. If you need to create one first, [click here.](https://hub.docker.com/signup)

## Create a Dockerfile for your app

In our application's directory, we shall create a `Dockerfile` and in it, let us write the following commands

```js
# specifies which build engine used to build the image
FROM python:3.6

# specifies a working directory for our source code
WORKDIR /app

# copies your source code files
COPY . /app

# command executed to install dependencies
RUN pip install -r requirements.txt

# specifies a port number for our image to run in a docker container
EXPOSE 8080

# command to run our docker image in container
CMD ["python", "app.py"]
```

After the step above, you should have something like this

![](../img/DockerfileFlask.png)

## Build a Docker Image for your app

You must note that the Dockerfile is like the heart of our dockerization process, without it we cannot build a Docker image for our application. We can therefore build our docker image by running this command `docker build . -t  rhodinnagwere/flask-app`

- `rhodinnagwere/flask-app` is the image name I chose for this guide, but you can provide any name of your choice.

![](../img/buildFlask.png)

## Run our Docker Image locally

To confirm that the image was successfully built, run `docker run -p 4000:8080 rhodinnagwere/flask-app`. This will run our containerized app locally on `port 8080`.

- The `-p` flag redirects a public port to a private port inside the container and the private port in this case is the port value `8080` we exposed in our Dockerfile while building the image.

![](../img/runFlaskLocally.png)

Next, we are going to push our local image to DockerHub so that we can access our application remotely and deploy it on Crane Cloud.

## Push local Docker Image to Docker Hub

We also need to access DockerHub in order to push our local image to our newly created project.

Inorder to do that, let us run `docker login` and authenticate with our DockerHub credentials as illustrated below. Now we are successfully logged in to DockerHub on our terminal.

![](../img/dockerLoginNode.png)

Before we can push the image, we need to tag it. Docker tags are a way of referring to your image that conveys useful information about its version. This is done in the format `docker tag <local-image-name> dockerUsername/<project-name>:<version>`

In our case, this will be `docker tag rhodinnagwere/flask-app rhodinnagwere/flask-app:latest`

- `rhodinnagwere` is my DockerHub username
- `rhodinnagwere/flask-app` is the DockerHub repository I have created and after it is a tag `latest`

After adding a tag to your image, you can proceed to push your local image along with its tag to DockerHub using the command `docker push rhodinnagwere/flask-app:latest`

![](../img/pushFlaskImage.png)

## Deploy to Crane Cloud

- Login to your Crane Cloud account, incase you do not have one you can simply create one by visiting [here](../authentication/registration.md)

- From your dashboard create a project, you can see how its done [here](../projects/projects.md)

- Inside that project, you create an app called `my-flask-app` as illustrated below and incase you do not know how that is done, learn from [here](../applications/deployApp.md)

![](../img/deployFlask.png)

After providing the above information, we can then deploy our app by clicking the `DEPLOY` button.

![](../img/deployFlaskSuccess.png)

Our python flask application is successfully deployed on Crane Cloud.

Thank you for following this guide.
