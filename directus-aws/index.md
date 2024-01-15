---
title: 'How to Deploy Directus to AWS EC2'
description: 'An how to guide on how to host a self hosted instance of Directus on AWS, use s3 for file storage and AWS '
author:
  name: 'Trust Jamin'
  avatar_file_name: 'jamin.png'
---

## Introduction

In this tutorial, you will learn how to deploy a self-hosted instance of Directus to Amazon Web Services (AWS) EC2, connect it to an AWS RDS PostgreSQL database and S3 storage bucket.

## Before You Start

You will need:

- An Amazon Web Service account ([AWS](https://aws.amazon.com)) with access to its `SecretKey` and `AccessKey`.
- A Directus project - using the [self-hosted guide](https://docs.directus.io/getting-started/quickstart.html).

## Set up an AWS RDS Database

Login to your AWS account and head to the [RDS page](https://eu-west-2.console.aws.amazon.com/rds/home) or search for RDS on the search bar to create a new database.

In the Create new database page,  select a standard create for database creation method, and on the engine options, select `PostgreSQL` (Directus also supports other databases such as, `MySQL`, `OracleDB`, `And Microsoft SQL` on AWS RDS).

![An AWS RDS database creation page with options to configure the database](create-db.png)

In the settings options, create a name for your database instance and a username and password credentials to log in to the database (You'll use these credentials to log in via Directus).

![An AWS RDS database creation page with options to configure the settings and login method of the database](db-settings.png)

- On the connectivity options, choose __Don't connect to an EC2 compute resource__ (this is because you haven't created an EC2 instance yet), and choose the default virtual private cloud (VPC) for connecting to the database.

- Select the `No` option for public access to the database to ensure the database can only be connected via the VPC security firewall.

- For the VPC security group (firewall), choose the default security group and select password authentication for database authentication.

![An AWS RDS database creation page to set up the connectivity options to the database](db-connection.png)

This will create a new PostgreSQL database for you on RDS.

Go to your newly created database page and on the connectivity page copy the `Endpoint` and `Port`

![An AWS RDS database creation page to set up the connectivity options to the database](db-details.png)

These details will serve as your credentials when connecting to the database to your Directus deployment:

```yml
  DB_CLIENT: "pg"
  DB_HOST: "YOUR_RDS_ENDPOINT"
  DB_PORT: YOUR_RDS_PORT
  DB_DATABASE: "postgres"
  DB_USER: "YOUR_DB_USERNAME"
  DB_PASSWORD: "YOUR_DB_USER_PASSWORD"

```

## Set up Amazon Elastic Compute Cloud (EC2) Instance

AWS EC2 are private virtual cloud servers you can spin up to run your applications on AWS cloud.

To create an EC2 instance, search for EC2 or head over to the [EC2  page](https://us-east-1.console.aws.amazon.com/ec2/home) and click `Launch instance`.

Add a name for the server called `Directus Server` and select the Amazon Linux image (you can also choose another image to suit your needs).

Create a key pair that you can use for logging in to the EC2 instance and for network settings, select the existing security group and choose the default security group for connecting to the EC2 instance.

![An AWS EC2 creation page with options to configure the  VPC security group](vpc.png)

For storage options, the default selection meets the requirements for running a Directus instance.

![An AWS EC2 creation page with options to configure the  Network settings](ec2-network.png)

Click on the `Launch Instance` button, and this will create a new EC2 instance for you

### Set Up EC2 Network Security Settings

To ensure that the EC2 instance can be accessed and operated from anywhere, head over to the security group in your EC2 instance and add the following inbound rules:

| Name  | Security group rule ID | IP version | Type | Protocol | Port Range | Source |
| --- | --- | --- | --- | --- | --- | --- |
| -  | your default security group | IPv4 | SSH | TCP | 22 | 0.0.0.0/0 |
| -  | your default security group | IPv4 | HTTP | TCP | 80 | 0.0.0.0/0 |
| -  | your default security group | IPv4 | HTTPS | TCP | 443 | 0.0.0.0/0 |

![EC2 Network security settings inbound rules](inbound.png)

Also, create an outbound rule for connecting to the database:

| Name  | Security group rule ID | IP version | Type | Protocol | Port Range | Source |
| --- | --- | --- | --- | --- | --- | --- |
| -  | your default security group | IPv4 | PostgreSQL | TCP | 5432 | 0.0.0.0/0 |

![EC2 Network security settings outbound rules](outbound.png)
Save these rules.

## Set up Directus on AWS EC2

On the EC2 instance page, click on the connect button to connect to your AWS EC2 instance using EC2 Instance Connect (you can also connect to EC2 via other methods such as a session manager or SSH client).

Once connected, run the command to install Docker on your EC2 server:

```bash
sudo yum install -y docker
```

Next, install Docker-compose with the command:

```bash
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
```

Grant permission to run the `docker-compose` command without using the `sudo` command, and start the docker service:

```bash
sudo chmod +x /usr/local/bin/docker-compose
sudo service docker start
```

Create a new directory called `directus` and navigate into the directory with the command:

```bash
mkdir directus && cd directus
```

In the `directus` directory, create a `docker-compose.yml` file and open it using `nano` with the command:

```bash
touch docker-compose.yml && nano docker-compose.yml
```

Update the `docker-compose.yml` file with the content:

```yml
version: "3"
services:
  directus:
    image: directus/directus:10.8.3
    ports:
      - 80:80
    volumes:
      - ./uploads:/directus/uploads
      - ./extensions:/directus/extensions
    environment:
      PORT: 80
      KEY: "replace-with-random-value"
      SECRET: "replace-with-random-value"
      ADMIN_EMAIL: "admin@example.com"
      ADMIN_PASSWORD: "d1r3ctu5"
      DB_CLIENT: "pg"
      DB_HOST: "YOUR_RDS_DB_URL"
      DB_PORT: 5432
      DB_DATABASE: "postgres"
      DB_USER: "YOUR_DB_USER"
      DB_PASSWORD: "YOUR_DB_PASSWORD"
      DB_SSL__REJECT_UNAUTHORIZED: false
      WEBSOCKETS_ENABLED: true
```

Save the file with `Ctrl O` and exit `nano` with `Ctrl X`.

To ensure that Directus can write and save data in the `extensions` and `uploads` directories, enter the following command to grant the current user ownership of the directory:

```bash
sudo chown $USER:$USER extensions uploads
```

Run the command `docker-compose up`, which should spin up a Directus instance on port `80` of your EC2 server.

Head to the EC2 dashboard and click on the instance ID; this will open the properties of your EC2 instance; here, you'll find the `Public IPv4 DNS.`

![An EC2 instance properties page showing the location of the Public IPv4 DNS value](ec2-public.png)

Open this URL and you should find Directus running.

## Set up AWS S3 (optional)

If you want to use AWS S3 for media storage, follow these additional steps:

Search for s3 on the console or navigate to the [s3 page](https://s3.console.aws.amazon.com/s3/home) to create a new storage bucket.

When creating a new bucket, disable ACLs (access control lists)  and block public access for the bucket for privacy (you can update these settings to suit your needs).

![An AWS s3 bucket creation page with options to configure the bucket](create-bucket.png)


![An AWS RDS database creation page to set up the connectivity options to the database](bucket-details.png)
Copy the name of the bucket and region to using in your `docker-compose.yml` configs:

```yml
  STORAGE_LOCATIONS: s3
  STORAGE_S3_DRIVER: s3
  STORAGE_S3_KEY: YOUR_AWS_ACCESS_KEY_ID
  STORAGE_S3_SECRET: YOUR_AWS_SECRET_ACCESS_KEY
  STORAGE_S3_BUCKET: YOUR_S3_BUCKET_NAME
  STORAGE_S3_REGION: YOUR_PREFERRED_REGION
  STORAGE_S3_ENDPOINT: s3.amazonaws.com
```

```
:::info AWS ACCESS KEY
For where to retrieve your AWS access key details, follow this [guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey).
:::
```

Re-run the Directus server with `docker-compose up` to implement the new changes added to the `docker-compose.yml` file.

Alternatively, you can run the command `docker-compose up -d` to run the Directus application on the background

## Next Steps

Some steps you could consider moving forward towards improving your deployment include:

- Improved security permissions for connecting to the created resources
- Consider utilizing [AWS Secrets Manager](https://us-east-1.console.aws.amazon.com/secretsmanager/) for managing your `ENV` variables
- Consider implementing a load balancer for scaling the incoming requests on your directus application.
- Compared to utilizing [Directus Cloud](https://directus.io/cloud), using a self-hosted instance of Directus on AWS has several limitations, like no backups, no rolling updates, round-the-clock monitoring, and the inability to automatically restart in the event of a crash.
