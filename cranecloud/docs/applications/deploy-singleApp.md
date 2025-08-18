### Basic information

Application name

- Enter a unique name for your application
- This name will be used to identify your application within the platform.

### Image configuration
If deploying a public image

- Simply specify your image name and tag

If deploying a private image

- Check the `Private Image` box to expand additional required fields:
    - Username (*)
    - Email
    - Password
    - Registry (e.g., docker.io)

### Port Configuration
- Enter the port number your application will expose.
- Default port is 80 if left blank.
- Ensure the port matches the one your app listens to inside the container.


### Number of replicas
- Specify how many instances of your app to run
- Defaults to 1 if not specified
- Increase for better availability and load distribution

### Entry Command
- You can specify an entry point or command for your application.

- This will override the default CMD or ENTRYPOINT defined in your image.

- Leave blank to use the default.

### Environment variables
Two ways to add environment variables:

#### Manual entry

- Enter key-value pairs for each variable
- Click `Add` to input variables one by one

#### Bulk import

- Use `Paste Environment variables from a file` option
- Copy and paste multiple variables at once

### Deployment process

- Fill in all required fields (Application Name, Image).
- (Optional) Add Port, Entry Command, and Environment Variables as needed.
- Add any necessary environment variables
- Review your configuration
- Click the `Deploy` button

Your application will begin deploying once all required information is provided and validated.

Need help? Contact our support team `info@cranecloud.io` if you encounter any issues during deployment.