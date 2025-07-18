# Projects on Crane Cloud

## Project naming convention

1. All project names should start with a letter.
2. Project names are only unique for a given user

## Creating a project

To create a project, navigate to the User dashboard, click on the `+ New Project` button at the top right hand side of the screen.

2. Once the button is clicked, a modal containing a form to create a project.

   - Select `Datacenter` location from available locations.
   - Enter `project name`.
   - Select `project organisation`
   - Select `project type`
   - Enter `project Description`
   - Click the `Create Project` button to submit the form.
   - The page will reload and display your newly created project.

Once the project has been successfully created it will show in the user dashboard.

## Editing a project

 - To edit your project, click on the project card.

 - While on the project dashboard page, click on the `project settings` on the bottom left corner of the page.

The project settings is composed of three sections which include:

   1. Project Details, which presents the meta data about a project
   2. Membership, which presents the owner and various members on the project with their specific roles
   3. Manage Project, which presents the project management controls such as updating, disabling and deleting a project.

 On clicking the `Update` button under the `Manage Project` section you can choose to update details for your project.


## Migrating a project

Project migration allows you to move an existing project from one server location (data center) to another within Crane Cloud.

### What is project migration?

`Project migration` is the process of transferring a project and all its associated deployments from the current server location to a different one. This is useful when optimizing for performance, reducing latency for your users, or managing infrastructure more efficiently.




- To migrate your project, click on the project card.

- While on the project dashboard page, click on the `project settings` on the bottom left corner of the page.

- Under the `General` tab in the project settings, scroll to the `Migrate Project` section.

- A form titled `Migrate project to a different server location` will appear.

   - Tick if it is a machine learning project.
   - Select the new `Project Location` from the dropdown menu.
   - Click the `Migrate Project` button to proceed.

Once clicked, the project and its deployments will be moved to the newly selected server location.

### Important Notes

- `Environment variables` are not saved by Crane Cloud during migration — they must be manually added again after the project is migrated.

- `App URLs will change` — all app deployments within the project will receive new URLs based on the new server. Make sure to update any services or clients that depend on the old URL.

 It is highly recommended to back up your environment variables and other critical settings in a safe place before initiating the migration process.


## Disabling a project

- On clicking the `Disable` button under the `Manage Project` section, a modal appears prompting you to confirm your action which will put a pause to all project resources.

## Deleting a project

- On clicking the `Delete` button under the `Manage Project` section, a modal appears prompting you to go ahead retype the project name after which the delete button will be enabled for you to continue to delete the project.

**Once you click the delete button, you will be unable to recover the deleted project.**

