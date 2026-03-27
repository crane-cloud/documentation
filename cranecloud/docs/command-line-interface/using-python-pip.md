# Crane Cloud Command Line Interface

Crane Cloud CLI is a powerful command-line tool that enables seamless interaction with Crane Cloud services, allowing you to manage projects, applications, and configurations efficiently.

## Installation

### Prerequisites
- Python 3.9 or higher
- pip (Python package installer)

### Install via pip
```bash
pip install cranecloud
```

For system-wide installation (may require administrator privileges):
```bash
sudo pip install cranecloud
```

### Upgrade to the latest version
```bash
pip install --upgrade cranecloud
```

### Verify installation
Verify that the installation was successful:
```bash
cranecloud --help
```

## Getting Started

### Authentication and Account Management

**Login:** Authenticate with your Crane Cloud credentials.
```bash
cranecloud auth login
```

**Logout:** Log out and clear stored credentials.
```bash
cranecloud auth logout
```

**User Information:** Retrieve information about the logged-in user.
```bash
cranecloud auth user
```

## Configuration Management

### View Current Configuration
**Display CLI Configuration:** Shows current CLI configuration settings.
```bash
cranecloud config get-config
```

### Cluster Management
**List Available Clusters:** Shows all clusters accessible to your account.
```bash
cranecloud clusters list
```

**Set Current Cluster:** Set a specific cluster as the current working cluster.
```bash
cranecloud clusters use-cluster <cluster_id>
```

## Project Management

### List and View Projects
**List All Projects:** Show all projects available in your account.
```bash
cranecloud projects list
```

**Get Project Details:** View detailed information about a specific project.
```bash
cranecloud projects info --id <project_id>
```

### Create and Configure Projects
**Create a New Project:** Create a project with specified parameters.
```bash
cranecloud projects create -n <project_name> -d "<project_description>" -t <project_type> -o <organisation>
```

Required Options:
- `-n, --name`: Name of the project

- `-d, --description`: Description of the project

- `-t, --project_type`: Type of the project

- `-o, --organisation`: Organisation the project belongs to

Optional Options:

- `-c, --cluster_id`: ID of the cluster where the project will be created

**Set Current Project:** Set a project as the current working context.
```bash
cranecloud projects set <project_id>
```

### Project Cleanup
**Delete a Project:** Remove a project by ID.
```bash
cranecloud projects delete --id <project_id>
```

## Application Management

### Deploy Applications
**Deploy a New App:** Deploy an application with comprehensive configuration.
```bash
cranecloud apps deploy -n <app_name> -i <image> -c "<start_command>" -r <replicas> -o <port> -e "KEY=VALUE"
```

Parameters:

- `-n`: Application name

- `-i`: Container image

- `-c`: Start command for the application

- `-r`: Number of replicas

- `-o`: Port number

- `-e`: Environment variables (can be used multiple times)

**Simple Deploy:** Initiate an app deployment interactively.
```bash
cranecloud apps deploy
```

### Manage Existing Applications
**List Applications:** Show all apps within the current project.
```bash
cranecloud apps list
```

**Get App Details:** View detailed information about a specific application.
```bash
cranecloud apps info <app_id>
```

**Update Application:** Modify app configuration or settings.
```bash
cranecloud apps update <app_id>
```

**Delete Application:** Remove an application by ID.
```bash
cranecloud apps delete --id <app_id>
```

## Workflow Examples

### Basic Setup Workflow

**Login to your account:**
```bash
cranecloud auth login
```

**List available clusters:**
```bash
cranecloud clusters list
```

**Set your working cluster:**
```bash
cranecloud clusters use-cluster <cluster_id>
```

**Create or select a project:**
```bash
cranecloud projects create -n "my-project" -d "My application project" -t "web" -o "my-org"
```

**Set the project as current:**
```bash
cranecloud projects set <project_id>
```

**Deploy your application:**
```bash
cranecloud apps deploy -n "my-app" -i "nginx:latest" -c "nginx -g 'daemon off;'" -r 2 -o 80 -e "ENV=production"
```

**Check deployment status:**
```bash
cranecloud apps list
```

**View app details:**
```bash
cranecloud apps info <app_id>
```

## Additional Commands

**Help Information:** Display comprehensive help and available commands.
```bash
cranecloud --help
```

**Command-Specific Help:** Get help for specific commands.
```bash
cranecloud <command> --help
```
