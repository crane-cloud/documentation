## Accessing application settings

- Navigate to your project dashboard
- Select your application
- Click `App Settings` under the `Other` section in the sidebar

## Configurable sections
### General details

View essential information:

- Application name
- Alias
- Image
- Application Status
- Replicas
- Age
- Port
- Current revision

### Image settings

Update container image configuration:

- Application image path
- Number of replicas
- Private image settings (if applicable)
- Authentication details for private repositories

### Environment and ports

Manage runtime configuration:

- Environment variables (key-value pairs)
- Port settings
- Entry Command for application startup

### Domain and URLs

Configure application access:

- View application link (public URL)
- View internal URL
- Set up custom domain

### Application revisions

- Track deployment history
- View timestamps for each revision
- Rollback to previous versions if needed

### Custom domain configuration
Beta Program Access

- Custom domains are available to Beta users
- Email `info@cranecloud.io` with subject `Beta Program` to request access

#### Setting up custom domain

During Application Creation:

- Check `Custom Domain` box
- Enter your domain name

For Existing Applications:

- Navigate to application settings
- Check `Custom Domain` box
- Enter your domain name
- Click `Update` button

#### DNS configuration

Follow provided instructions for A-Record setup

Different DNS providers require specific steps
Examples provided for:

- NameCheap
- GoDaddy
- Other providers (refer to provider documentation)

#### Reverting custom domain

- Click `Revert` button to return to default Crane Cloud URL

### CI/CD Integration
Setting up webhook

- In App Settings, scroll to `Manage Application`
- Click `Set up CI/CD` button

#### Generate webhook link
- Default tag (`latest`) or
- Custom tag (e.g. `production`)

#### Configuring DockerHub

- Go to your DockerHub repository
- Navigate to Webhooks tab
- Add new webhook:
    - Enter webhook name
    - Paste generated Crane Cloud URL
- Save webhook configuration

### Deployment
- Each deployment entry includes the following information; 
   - Revision
   - Image 
   - Replicas
   - Created At
   - Actions 

### Rollback
- Click the Rollback button under the Actions column for a selected revision.

- This will revert your application to the state it was in during that deployment, using the specified image and configuration.

- Use rollback if a recent update causes issues or if you want to restore a previously stable version.

### Updating an application
- On clicking the Update App button under the App Settings section, a form appears allowing you to modify your application configuration.

- You can update the following fields; 
   - Application Name
   - Image
   - Port
   - Number of replicas
   - Entry Command 

After making the necessary changes, click the Update button to save and apply the new configuration to your application.
### Disabling an application

- On clicking the `Disable` button under the `Manage Application` section, a modal appears prompting you to confirm your action which will put a pause to your application.

### Deleting an application

- On clicking the `Delete` button under the `Manage Application` section, a modal appears prompting you to go ahead retype the application name after which the delete button will be enabled for you to continue to delete the application.

**Once you click the delete button, you will be unable to recover the deleted application.**

Need help? Contact our support team `info@cranecloud.io` if you encounter any issues when managing your application.