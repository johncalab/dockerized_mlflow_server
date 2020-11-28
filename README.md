Work in progress...

# Dockerized mlflow server with authentication
A simple `docker-compose` app to easily deploy an `mlflow` server, together with an nginx reverse proxy for authentication.

Step-by-step instructions on how to deploy can be found at the bottom of this README.

## What's in this repo?


## mlflowui
The app consists of two services.
The first is `mlflowui`, the mlflow server.
The corresponding dockerfile is pretty straightforward: use a python3 base image and install the necessary packages.
In `docker-compose.yml` you'll find the entrypoint under `command`.
This can be customized depending on your needs.
The simplest way to use this is by
```
command: mlflow server --host 0.0.0.0
```
this will keep all artifacts and parameters (aka the `mlflow backend`) inside the docker container.
If you need these to persist in case the containers go down, you should use a volume.

A more robust way to deal with artifacts and the backend is to have them stored elsewhere (for example in an S3 bucket and a SQL server, respectively).
To do this, you need to populate the `.mlfenv` file accordingly (see below).

## nginxauth
The second service is `nginxauth`, an nginx reverse proxy.
This is how we force users to provide credentials.
To use this service, we first need to create the `.htpasswd` file (see instructions below).


## Instructions
To deploy locally, follow the steps below.

### Adding credentials
Pick a username, here we use `incredibleuser`.
Run
```
htpasswd -c revproxy/.htpasswd incredibleuser
```
this will prompt you to enter a password.

### Backend and artifact store
If you don't care about storing artifacts and parameters elsewhere, then swap the line beginning with `command` in the `docker-compose.yml` with `command: mlflow server --host 0.0.0.0`, and ignore the rest of this section.

#### backend
If you want to store experiment parameters (the mlflow backend) in a separate SQL server, first populate the `.mlfenv` file.
The only variable that matters is `BACKEND_STORE_URI`.
You can either populate that directly with a long string, or populate each chunk.

#### artifacts
When using a custom backend, mlflow requires a location for artifacts.
Populate the `.mlfenv` file with the address.
The default provided will just store artifacts inside the mlflow container (in this case restarting the container will delete the artifacts).

### Launch
Once you've populated the `.mlfenv` file with backend and artifacts, run the following commands to start the app.
Note: if you are not using remote backend or artifacts the first two commands are not needed.

```
set -a
source .mlfenv
docker-compose up --build -d
```

To kill
```
docker-compose down --remove-orphans
```
