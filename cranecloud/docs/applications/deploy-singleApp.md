### Basic information

Application name

- Enter a unique name for your application

Number of replicas

- Specify how many instances of your app to run
- Defaults to 1 if not specified
- Increase for better availability and load distribution

### Image configuration
If deploying a public image

- Simply specify your image name and tag

If deploying a private image

- Check the `Private Image` box to expand additional required fields:
    - Username (*)
    - Email
    - Password
    - Registry (e.g., docker.io)

### Environment variables
Two ways to add environment variables:

#### Manual entry

- Enter key-value pairs for each variable
- Click `Add` to input variables one by one

#### Bulk import

- Use `Paste Environment variables from a file` option
- Copy and paste multiple variables at once

### Deployment process

- Fill in all required fields
- Add any necessary environment variables
- Review your configuration
- Click the `Deploy` button

Your application will begin deploying once all required information is provided and validated.

Need help? Contact our support team `info@cranecloud.io` if you encounter any issues during deployment.