# Crane Cloud Command Line Interface

Crane Cloud CLI is a powerful command-line tool that enables seamless interaction with Crane Cloud services, allowing you to manage projects, applications, and configurations efficiently.

## Installation

### Prerequisites

- Python 3.9 or higher
- pip (Python package installer)

### Install via pip

```bash
> pip install cranecloud
```

For system-wide installation (may require administrator privileges):

```bash
> sudo pip install cranecloud
```

### Upgrade to the latest version

```bash
> pip install --upgrade cranecloud
```

### Verify installation

Verify that the installation was successful:

```bash
> cranecloud
```

### Authentication and account management

**Login:** Authenticate the user with their credentials.

```bash
> cranecloud auth login
```

**Logout:** Log out the user, clearing the stored credentials.

```bash
> cranecloud auth logout
```

**User information:** Retrieve information about the logged-in user.

```bash
> cranecloud auth user
```

### Project management

**List projects:** Show a list of projects available in the user's account.

```bash
> cranecloud projects list
```

**Create a project:** Create a new project.

```bash
> cranecloud projects create [OPTIONS]
```

Required [OPTIONS]:

- `-n, --name`: Name of the project

- `-d, --description`: Description of the project

- `-t, --project_type`: Type of the project

- `-o, --organisation`: Organisation the project belongs to

Optional Options:

- `-c, --cluster_id`: ID of the cluster where the project will be created


## 

**Delete a project:** Delete a project by ID or name.

```bash
> cranecloud projects delete --id <project_id>
```

**Get project details:** View detailed information about a project.

```bash
> cranecloud projects info --id <project_id>
```

### Configuration management

**List config:** Show a list of config available for the current user.

```bash
> cranecloud config get-config
```

**Set current project to use:** This is projects apps commands will default to.

```bash
> cranecloud projects use-project <project_id>
```

**Set current cluster to use:** This is cluster projects commands will default to.

```bash
> cranecloud projects use-project <cluster_id>
```

### App commands

**Deploy an app:** Initiate an app.

```bash
> cranecloud apps deploy
```

**List apps:** Show apps within a project.

```bash
> cranecloud apps list 
```

**Update an app:** Update app information.

```bash
> cranecloud apps update <app_id>
```

**Get app details:** View detailed information about a specific app.

```bash
> cranecloud apps info <app_id>
```

**Delete app:** Delete an app by ID or name.

```bash
> cranecloud apps delete --id <app_id>
```

### Other useful commands

**Help information:** Show manual.

```bash
> cranecloud --help
```
