# Deploy An Application Using A DockerHub Image

John Doe supports the backend API of his company and they have been trying to explore the Crane cloud solution as a platform to host their backend APIs. He has his App image stored in the DockerHub image registry.

Joe Doe would need to undertake the steps below to have the Flask API on Crane Cloud:

**User Actions:**

1\. After logging in to your Crane Cloud Account and creating a Project, drill into a particular project. and click the _Create button_ at the top right corner of the Apps dashboard page.
![](../img/appDashboard.png)

2\. A modal containing the App deployment form will then pop up.
![](../img/deployapp.png)



3\. IF the application image is a docker public image:

- Enter your App Name

- Select number of app replicas you want

- Enter absolute URL to the docker image -- this is your dockerhub username/image_name in this example above it is "nginxdemos/hello"

- Enter the entry command if any

- Enter the port on which you want your App to run (defaults to port 80)

- Enter and Add any App specific environment variables

- Press the Proceed button

4\. IF the application image is a private docker image:

- Enter your App Name

- Enter absolute URL to the docker image

- Check the Private Image checkbox

- When checked, it displays a form for you to enter your DockerHub credentials used for authentication when the image is being pulled. In the server field, enter Url path to image

- ![](../img/privateImage.png)

- Enter the entry command if any

- Enter the port on which you want your App to run (defaults to port 80)

- Enter and Add any App specific environment variables

- Press the Proceed button

**_Note:_** These steps apply to all Apps whose images are stored on Dockerhub.
