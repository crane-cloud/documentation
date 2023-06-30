# Crane Cloud Documentation
> A centralised location for all information regarding usage of the platform.

## How to setup the project:
 #### Clone the repository
  ` git clone https://github.com/crane-cloud/documentation.git`

 * If `develop` is not up to date, `git checkout develop` for it is the default base branch for this project.
### Run App locally
#### Install pip or upgrade its version by running
 
     python3 -m pip install --upgrade pip

#### Install and creating a virtual environment run
 
     python3 -m pip install virtualenv

#### To create a virtual environment run

     python3 -m virtualenv name-of-your-virtual-environment

**NOTE** Remember to add the virtual environment to your gitignore.

#### To activate the virtual evnvironment created run 

     source env/bin/activate

#### To install the packages in the requirements.txt file run

    pip install -r requirements.txt

#### To run the docs, run:

     mkdocs serve

### Run app with docker

#### Create a docker image
     docker build -t cc-docs .

#### Start the container

     docker run -d -p 3000:80 --name cc-docs-app cc-docs

#### Checkout application
Should be running on http://localhost:3000


## Deployment
* The Deployed staging app is [here.](https://staging-docs.cranecloud.io/)
* The Deployed production app is [here.](https://docs.cranecloud.io/)