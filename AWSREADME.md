# Deploying on AWS
A list of things that tripped me up while setting everything up on AWS.

## RDS
Make sure the database can accept connections!
Also, append `/postgres` at the end of the URI, mlflow will take care of the rest.

## EC2
Make sure the EC2 instance can accept both `ssh` and `http` connections!

### ssh
To ssh into your EC2 instance you will have already downloaded a `.pem` file somewhere.
Change permissions so that AWS does not complain.

```
chmod 400 mykeypair.pem
```

To find the exact user/host combination to ssh into your instance, go on the AWS console and click on: `EC2`, `instances`, `theinstanceid`, `connect`.

### incantations
To get things up and running on your EC2 instance, a couple of things need to be installed.
I'll write down the full list now, and then explain what they do.

First run these.

```
sudo yum update -y
sudo yum install git
sudo yum install httpd
sudo amazon-linux-extras install docker
sudo service docker start
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo usermod -a -G docker ec2-user
```

Then log out, log back in, and run the following (where `incredibleuser` is your desired username).

```
git clone https://github.com/johncalab/dockerized_mlflow_server.git
cd dockerized_mlflow_server
htpasswd -c nginxauth/.htpasswd incredibleuser
```

Populate the `.mlfenv` file, or modify `docker-compose.yml` as needed.
Launch!

```
set -a
source .mlfenv
docker-compose up --build -d
```

#### explanation
The first commands install git and httpd.
Then we install docker and docker-compose (see [here](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-basics.html)).
Then we give docker some privileges, after which we log out and back in.
We then clone our repo, cd into it, and set credentials for nginx.
Then we populate the `.mlfenv` file, for example with `vim`.

Don't forget, in vim you can save by pressing `esc :w`, quit by `esc :q`, save and quit `esc :wq`, save without quitting `esc :q!`.

### url
To view the mlflow ui, navigate to the address found under `Public IPv4 DNS` of the EC2 instance.
