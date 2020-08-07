# Database Support

This feature is still in beta mode and under continous iterations.

## Supported versions

We currently support the following database flavors:
- Postgres 10.8
- mysql 8.0
- mariadb 10.5

## Provisioning the database

Once you provide the required information for database support, a database instance is provisioned on your behalf and credentials are attached to the application environment. 

The corresponding application will have the following Environment Variables added to its environment
- DB_HOST  --- The Database Host
- DB_USER  --- The database user permitted to access the provisioned database
- DB_PASSWORD --- Password for the database user.
- DB_PORT --- The Port at which the database host is allowing connections
- DB_DATABASE --- The Name of the database provisioned.

Our Users intending to use this feature should expect these environment variables while developing and use them to establish database connections. 

For a set by step guide on how to deploy an application that requires database support please follow the guide [here](../applications/dockerWithDB.md)
