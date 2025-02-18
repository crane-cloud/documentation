MIRA is Crane Cloud's solution for users who want to deploy applications without dealing with containerization. Instead of creating Docker images manually, you can simply upload your project code as a zip file, and MIRA handles the containerization and deployment process automatically.

## Supported frameworks

- HTML-CSS-JS Applications
- React
- Flask
- Django
- NodeJS
- Laravel

## Deployment process

### Prerequisites

- Active Crane Cloud account
- Created project
- Application code zipped in `.zip` format

### Get started

- Log into your Crane Cloud account
- Navigate to your desired project
- Click the `Apps` tab in the left navigation
- Click `New App` button in the top right corner
- Select the `Deploy with MIRA` tab in the deployment modal

### Configure deployment

#### Framework selection

- Choose your application's framework from the supported list

#### Registry selection

Choose between:

- Harbor (Crane Cloud's self-managed registry)
- DockerHub (Public registry under cranecloud repository)

#### Image configuration

- Image name: This will be used for both the container image and application name
- Version: specify the image tag for versioning


#### Code Upload

- Upload your application's codebase as a `.zip` file
- Ensure all necessary files are included in the zip

### Deploy

- Verify all fields are filled correctly
- Click the `Deploy` button
- Wait for the provisioning process to complete

### Access your application

Upon successful deployment:

- Your application will appear in your projects list
- You will receive a URL to access your deployed application

### Troubleshooting
If you encounter any issues during deployment:

- Verify your zip file contains all necessary files
- Check that you've selected the correct framework
- Ensure your code follows standard project structure for the chosen framework

Need help? Contact our support team `info@cranecloud.io` if you encounter any issues during deployment.