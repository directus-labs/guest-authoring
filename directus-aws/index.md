---
title: 'How to Deploy Directus to AWS'
description: 'An how to guide on how to host a self hosted instance of Directus on AWS, use s3 for file storage and AWS '
author:
  name: 'Trust Jamin'
  avatar_file_name: 'jamin.png'
---

## Introduction

[AWS](https://astro.build/) is a highly performant web framework used for building content-heavy websites. In this tutorial, you will learn how to deploy a self hosted instance of Directus to AWS, connect it to an s3 storage, and an AWS RDS postgreSQL database.

## Before You Start

You will need:

- An Amazon Web Service account ([AWS](https://aws.amazon.com/free)) with .
- A Directus project - you can use [Directus Cloud](https://directus.cloud/) or [run it yourself](https://docs.directus.io/getting-started/quickstart.html).

## Set up AWS Simple Storage Service (s3)

Login to your AWS and search for s3 on the console or navigate to the [s3 page](https://s3.console.aws.amazon.com/s3/home) to create a new storage bucket.

When creating a new bucket, ensure ACLs is disabled and public access is blocked for the bucket for privacy (you can update these settings to suit your needs).

Copy the name of the bucket and store somewhere for later use

## Set up an Amazon Relational Database Service (RDS)

On your AWS console, head over to the [RDS page](https://eu-west-2.console.aws.amazon.com/rds/home) or search for RDS on the search bar to create a new database.

In the create new database page,  select a standard create and on the engine options select `PostgreSQL` (Directus also supports other databases such as, `MySQL`, `OracleDB`, `MicrosoftSQL` on AWS RDS).

On the settings options, create a name for your database instance and create a username and password credentials to login to the database.

On the connectivity options, choose don't connect to an EC2 instance (this is because you haven't created an EC2 instance yet), and choose the default virtual private cloud (VPC) for connecting to the database.

Select the `No` option for public access for the Database to ensure the database can only be connect via VPC security firewall.
For the VPC security group (firewall), choose the default security group and choose password authentication for database authentication.

This will create a new `PostgreSQL` database for you on RDS.

## Set up Amazon Elastic Compute Cloud (EC2) Instance

AWS EC2 are private virtual cloud servers you can spin up to run your applications on AWS cloud

To create an EC2 instance, search for EC2 or head over to the [EC2  page](https://us-east-1.console.aws.amazon.com/ec2/home) and click on Launch instance.

Add a name for the server called `Directus Server` and select the Amazon Linux image (you can also select other Linux image to suit your need).

Create a key pair that you can use for logging in to the EC2 instance.

Also for network settings, select existing security group  and choose the default security group for connecting to the EC2 instance.

For storage options, the default selection meets the recommended requirements for running Directus.

Click on the `Launch Instance` button and this will create a new EC2 instance for you

### Set up permission

To ensure that the EC2 instance can be accessed and operated by a security group, head over to the security group in your EC2 instance and add the following inbound rules:
| Name  | Security group rule ID | IP version | Type | Protocol | Port Range | Source |
| --- | --- | --- | --- | --- | --- | --- |
| -  | your default security group | IPv4 | SSH | TCP | 22 | 0.0.0.0/0 |
| -  | your default security group | IPv4 | HTTP | TCP | 80 | 0.0.0.0/0 |
| -  | your default security group | IPv4 | HTTPS | TCP | 443 | 0.0.0.0/0 |

Create an outbound rule also for connecting to the the database:
| Name  | Security group rule ID | IP version | Type | Protocol | Port Range | Source |
| --- | --- | --- | --- | --- | --- | --- |
| -  | your default security group | IPv4 | PostgreSQL | TCP | 5432 | 0.0.0.0/0 |

Save the rules

## Set up Directus on AWS EC2

To set up Directus on EC2 you first need to install using the command:

```bash
sudo yum install -y docker
```

Next, install Docker-compose with the command:

```bash
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
```

### Install Docker

### Install Docker Compose

### Give Permission to folder

## Next Steps
