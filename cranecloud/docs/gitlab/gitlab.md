# GITLAB

## Gitlab Self Managed Configuration
After successfully, deploying a dockerised application using the Gitlab Community docker image, the setup configuration steps are detailed below.


## GitLab launching
Our self-managed GitLab is available at the given app deployment URL which can be found on your Cranecloud app dashboard. The following screen should appear: The first launch may take several minutes.
 ![](../img/gitlab4.png)
 Figure 1: GitLab CE login page


After logging in, the following screen should appear in figure 2:

 ![](../img/gitlab3.png)

## Initial configuration of the portal

The initial configuration involves changing a few portal settings. First, turn off open registration for everyone so that only users authorized by the admin can complete registration. This can be done by clicking the view settings button available on the upper bar with a warning. On the new page, uncheck Sign-up enabled and save the changes.

The next step should be to change the root user. To do this go under profile and then account settings and enter the name in the Change username field. Approve by clicking on Update username.

The last step will be the change password under the profile page, enter a temporary and new password and then approve the change by clicking save password.

## Create a new repository

After the initial configuration, we can create our first repository. To do this, go to the page at projects/new and click create a blank project or Import project.

 ![](../img/gitlab2.png)

 Figure 3: Project creation page

You can import a project by selecting Import project from Repo from URL, then providing the Git repository URL address. After a while, your first repository will be copied to your Gitlab.

 ![](../img/gitlab1.png)

 Figure 4: Project Import

## Creating a CI/CD pipeline
To create a CI?CD pipeline for the project, click the main menu on the left, CI/CD, and then Editor. An option to create a .gitlabb-ci.yml file, which will contain our pipeline definitions, will appear on the screen. This file will be created in the Git repository.

Once the CI file is committed and approved, GitLab will launch the process. To check the results, go to CI/CD -> pipelines in the menu on the left. On the screen, we should see that our first task has already been started.

We can go to the details of this task by clicking on the pending button or build number #1. After a while, the task should be built and tested.


1. To create a project, navigate to the User dashboard, click on the `ADD` button at the top right hand side of the screen.
   ![](../img/projectsDashboard.png)

2. Once the button is clicked, a modal containing a form to create a project. Let us use a case study: Create a Tensorflow project.

   - Select Datacenter location from available locations.
   - Enter project name.
   - Enter project's organisation
   - Enter project type
   - Enter project Description.
   - Click the `Create Project` button to submit the form.
   - The page will reload and display your newly created project.

   ![](../img/create_project_form.png)

3. Once the project has been successfully created it will show in the user dashboard.
   ![](../img/user_projects.png)

4. To access a particular project, click on the project card to drill in.

## Editing A Project

1. To edit your project, click on the project card.

   ![](../img/singularProject.png)

2. Once inside the project,an apps page appears similar to that one below.

   ![](../img/apps_page.png)

3. On the apps page, click on the settings icon on the bottom left corner of the page.

   ![](../img/projectSettings.png)

4. The project settings page will appear.

   ![](../img/projectSettingsPage.png)

5. You can choose to edit the project name or description.

   ![](../img/projectUpdateModal.png)

## Deleting A Project

1. To delete a project, click on the project card.

   ![](../img/singularProject.png)

2. Once inside the project,an apps page appears similar to that one below.

   ![](../img/apps_page.png)

3. On the apps page, click on the settings icon on the bottom left corner of the page.

   ![](../img/projectSettings.png)

4. That takes you to the page below that has a "Delete Project" button. Click the _Delete Project_ button.

   ![](../img/projectSettingsPage.png)

5. On clicking the button, a modal appears prompting you to go ahead retype the project name after which the delete but will be enabled for you to commence with deleting the project.

   ![](../img/projectDelete.png)
   ![](../img/projectDeleteEnabled.png)

**_ Once you click the delete button, you will be unable to recover the deleted project. _**

## Project Creation Errors

**Note: The following errors may occur when creating Projects**

1. In case the project name you entered contains space or symbols you will get this error below.
   ![](../img/projectError1.png)

2. If you do not fill one of the modal fields, you get this error.
   ![](../img/projectError2.png)

3. When you try to create a project with the same name as one of your existing projects you will get this error.  
   ![](../img/projectError3.png)

4. When the project name exceeds 18 characters, you will get the error below.
   ![](../img/projectError4.png)

## Project Billing

1. To get your project billing information, while inside a project click `Billing` on the sidebar.

   ![](../img/BillingSideBar.png)

2. On the right side of the sidebar a similar page will appear showing the available metrics for the project.

   ![](https://user-images.githubusercontent.com/69305400/170105260-52d8aa77-4213-4851-9fd9-44b180c84ab9.png)

3. In case you need to make payment for your app usage then click the pay bill button. Note the button remains disabled util a particular threshold is attained.

   ![](https://user-images.githubusercontent.com/69305400/179727501-fba11575-d660-4569-a932-eb11bc61e538.png)

4. After selecting the payment methods you can go ahead to make payment and you will recieve an email showing for your payment receipt. In the dashboard you can be able to view transactions, invoices and receipts and all these can be downloaded. Below is a example of a receipt with the provision of being able to download it as a PDF.

   ![](https://user-images.githubusercontent.com/32802973/177199923-1fc721fa-e02f-475d-9777-33bb02179eb2.png)

   ![](https://user-images.githubusercontent.com/32802973/177199908-397dc1c5-1210-4efc-9bcf-326b77fa0c0f.png)

5. You can pay using assigned credits and your credits can be viewed on the page too.

   ![](../img/Credits.png)
