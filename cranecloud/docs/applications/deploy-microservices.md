This section explains how to deploy interconnected applications (microservices) on Crane Cloud, such as frontend and backend applications that need to communicate with each other.

## Deployment process

### Initial setup

- Click the `Create App` button
- Select `Deploy Microservices` option

### First application configuration (e.g. Backend)

Fill in required application details:

- Application name
- Image details
- Number of replicas (if needed)
- Add any necessary environment variables specific to this service

### Second application configuration (e.g. Frontend)

Fill in the second application's details

Configure environment variables:

- Create variables that reference the first application
- Select the source application (e.g. backend) for URL or other needed values
- Specify variable names that will be used to access these values

### Environment variables management

Each application has its own `Environment Vars` section in the settings

You can view and manage:

- Application-specific variables
- Inter-service connection variables
- Shared environment variables

## Best practices

- Use clear, descriptive names for your applications
- Document all environment variables
- Test inter-service communication variables before full deployment
- Maintain proper security practices when sharing service information

## Important notes

- Ensure all required environment variables are properly set
- Verify service connections before deploying
- Check application logs if services fail to communicate
- Consider security implications when sharing service information

Need help? Contact our support team `info@cranecloud.io` if you encounter any issues during deployment.