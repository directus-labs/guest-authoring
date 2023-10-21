---
title: 'Hosting a Directus Application on an Ubuntu Server'
description: 'Learn how to host a Directus application on Ubuntu server with Docker, Nginx, and SSL encryption. A step-by-step guide for seamless deployment and security.'
author:
  name: 'Yusuf Akorede'
  avatar_file_name: 'add-to-directory'
---

## Introduction

In this tutorial, you will learn how to deploy a directus application within a Docker container on an Ubuntu Linux server and connect it to a custom domain, such as `https://directus.exampledomain.com`.

This guide covers setting up Docker, configuring Docker Compose, using Nginx as a reverse proxy, and securing your application with SSL. Additionally, you'll discover how to run the application as a background service, ensuring seamless operation and easy management.

## Prerequisites

1. A Directus Project: Have a local Directus project ready to deploy. If you don't have one, you can follow the [Directus Quickstart Guide][quickstart] to create a new project.

2. Ubuntu Server: Ensure you have access to an Ubuntu server preferably via SSH. You can get a server from a cloud hosting provider like DigitalOcean, Linode, or AWS. Then configure your local machine to access the server via SSH.

3. Domain Name: Have a registered domain name with access to DNS settings.

4. Basic Command-Line Skills: Have basic knowledge of the Linux command line. And familiarity with commands like `scp` (SCP is used to upload files to the server) and `vi` (vi is used to edit files on the server).

## Upload Your Local Directus Application Folder to the Server

Assuming you have a local directus folder structure that looks like this:

```plaintext
├── database
│   └── data.db
├── docker-compose.yml
├── uploads
└── extensions
```

And the _docker-compose.yml_ file looks somewhat like this:

```yaml
version: '3'
services:
  directus:
    image: directus/directus:latest
    ports:
      - 8055:8055
    volumes:
      - ./database:/directus/database
      - ./uploads:/directus/uploads
      - ./extensions:/directus/extensions
    environment:
      KEY: 'replace-with-random-value'
      SECRET: 'replace-with-random-value'
      ADMIN_EMAIL: 'admin@example.com'
      ADMIN_PASSWORD: 'd1r3ctu5'
      DB_CLIENT: 'sqlite3'
      DB_FILENAME: '/directus/database/data.db'
      WEBSOCKETS_ENABLED: 1
# Add your other settings
```

:::info Security Note

Ensure you adjust the email and set a strong password for the admin user.

:::

Use `scp` (Secure Copy Protocol) to upload the local folder to your server.

From your local machine terminal, run:

```bash
scp -r /path/to/your/local/directus/folder username@server_ip:/path/to/your/remote/folder
```

Please replace _/path/to/your/local/directus/folder_ with the actual path to your local Directus application folder, and _/path/to/your/remote/folder_ with the corresponding path on your server.

Also, replace `username` with your server's username and `server_ip` with your server's public IP address.

:::info Database Note

Note that the database used in this tutorial is SQLite. For other types of databases like MySQL and PostgreSQL, you might have to create a database dump and export the dump to your remote server.
:::

## Preparing Your Ubuntu Server

Access your Ubuntu server from you local machine terminal via SSH:

```bash
ssh username@server_ip
```

In your server terminal, run the following commands:

```bash
sudo apt update
sudo apt upgrade -y
```

## Installing Docker and Docker Compose

To install Docker and Docker Compose, run the following commands:

```bash
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```

This install Docker and enable the docker service.

## Run your Directus Application

Run these commands to allow incoming traffic on port 8055:

```bash
sudo ufw allow 8055/tcp
sudo ufw enable
```

Run your Directus application using Docker Compose:

```bash
cd /path/to/your/directus/folder
sudo docker-compose up
```

On first run this will pull the docker image from the registry and then start your Directus application.

Your application should now be accessible at `http://your_server_ip:8055`.

## Running the Docker Container as a Background Service

Running your application like above is fine, but it will stop running when you close the terminal.

To ensure your application runs in the background and restarts automatically, you can create a systemd service.

:::info What is Systemd?

Systemd is a system and service manager for Linux operating systems. It provides a standard process for controlling the startup, management, and monitoring of services and applications. It is usually defined by a configuration file usually ending with the _.service_ extension.

:::

### Create a Systemd Service File

Create a file named _directus.service_ in the _/etc/systemd/system/_ directory:

```bash
sudo vi /etc/systemd/system/directus.service
```

Add the following content, updating the `WorkingDirectory` to your Directus project directory containing the _docker-compose.yml_ file:

```plaintext
[Unit]
Description=Directus Docker Service
Requires=docker.service
After=docker.service


[Service]
Restart=always
WorkingDirectory=/path/to/your/directory-containing-docker-compose
ExecStart=/usr/bin/docker-compose up
ExecStop=/usr/bin/docker-compose down


[Install]
WantedBy=multi-user.target
```

Save the file and exit the editor.

Let's step through the service file:

- [Unit] Section:

  - Description: Description of the systemd service.
  - Requires: Dependency on the Docker service.
  - After: Starts after the Docker service.

- [Service] Section:

  - Restart=always: The service restarts automatically on exit.
  - WorkingDirectory: Path to the directory containing the docker-compose.yml file.
  - ExecStart: Command to start Docker containers.
  - ExecStop: Command to stop Docker containers.

- [Install] Section:
  - WantedBy=multi-user.target: Service enabled on reaching the multi-user state.

### Enable and Start the Service

Enable the service to start on boot, and then start the service:

```bash
sudo systemctl enable directus.service
sudo systemctl start directus.service
```

Now, your dockerized Directus application is running as a background service and will automatically restart in case of failures or system reboots.

Run the following command to check the status of the service:

```bash
sudo systemctl status directus.service
```

You can also check your application at `http://your_server_ip:8055`.

## Configuring DNS Settings for Your Domain

Configuring DNS settings for your domain is a crucial step in making your Directus application accessible to users over the internet. Here's how to do it:

1. Access Your Domain Registrar's Website: Log in to the website of your domain registrar, where you initially purchased or registered your domain name. This is where you manage your domain settings.

2. Locate DNS Management or Domain Settings: Inside your domain registrar's dashboard, look for options like "DNS Management," "Domain Settings," or "Domain Management." These names might vary based on the registrar's interface.

3. Add a DNS Record for Your Subdomain: Create a new DNS record to point your subdomain (e.g., directus.exampledomain.com) to your server's public IP address. Depending on the registrar, you may need to choose the record type, which is usually an "A" record for IPv4 addresses. Enter your server's public IP address in the designated field.

4. Save the changes: After adding the DNS record, save the changes. DNS propagation might take some time, ranging from a few minutes to a few hours. During this period, the DNS changes will propagate across the internet, making your subdomain accessible.

## Setting Up Nginx as a Reverse Proxy

Nginx, often called 'engine-x,' is a powerful reverse proxy server widely used in web hosting.

As a reverse proxy, Nginx sits between clients and backend servers, forwarding client requests to the appropriate server.

Install Nginx on your server:

```bash
sudo apt install nginx -y
```

Create an Nginx configuration file named _directus_ for your domain:

```bash
sudo vi /etc/nginx/sites-available/directus
```

Add the following configurations, replacing `directus.exampledomain.com` with your domain name:

```nginx
server {
listen 80;
server_name directus.exampledomain.com;


location / {
proxy_pass http://localhost:8055;
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection 'upgrade';
proxy_set_header Host $host;
proxy_cache_bypass $http_upgrade;
}
}
```

This simply forward the request to the domain to the Directus application running at `localhost` port `8055`.

Let's step through the config file:

- listen 80: Listens for incoming connections on port 80, the default for HTTP traffic.

- server_name directus.exampledomain.com: Matches requests to this domain.

- location / { ... }: Handles all requests for directus.exampledomain.com.

- proxy_pass http://localhost:8055: Forwards requests to the Directus application.

- proxy_http_version 1.1: Uses the HTTP/1.1 protocol for Nginx-proxy communication.

- proxy_set_header Upgrade $http_upgrade: Essential for WebSocket connections.

- proxy_set_header Connection 'upgrade': Indicates connection upgrade.

- proxy_set_header Host $host: Sends original host information to the server.

- proxy_cache_bypass $http_upgrade: Bypasses caching for WebSockets.

To test the Nginx configuration files for syntax errors, you can use the following command:

```bash
sudo nginx -t
```

Create a symbolic link to enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/directus /etc/nginx/sites-enabled
```

And restart Nginx:

```bash
sudo systemctl restart nginx
```

Now you should be able to access your Directus application at `http://directus.exampledomain.com`.

## Securing Your Application with SSL (Optional but Recommended)

Securing your application with SSL (Secure Sockets Layer) encryption is highly recommended to protect data transmitted between your users and your server. Let's Encrypt offers free SSL certificates, and here's how to set it up for your Directus application:

1. Install Certbot: On your server, run the following commands to install Certbot and the Certbot Nginx plugin:

```bash
sudo apt install certbot python3-certbot-nginx -y
```

2. Obtain an SSL Certificate: Run Certbot with the --nginx option, specifying your domain (directus.exampledomain.com in this case):

```bash
sudo certbot --nginx -d directus.exampledomain.com
```

Certbot will interactively guide you through the setup process.

Ensure you select the option to redirect HTTP traffic to HTTPS when prompted. Certbot will automatically configure Nginx to use the SSL certificate.

:::info Renew certificate

Also ensure to renew the certificate before it expires to maintain a secure connection.
:::

3. Verify SSL Configuration: After the setup is complete, visit your Directus application using `https://directus.exampledomain.com` in a web browser. You should see a padlock icon indicating a secure SSL connection.

## Summary

This tutorial guided you through hosting a Directus application on an Ubuntu server, covering essential steps like Docker setup, firewall configuration, and SSL encryption. By following these instructions, you've ensured a secure, accessible, and continuously running environment for your Directus project.

Should you have any questions or encounter issues, feel free to refer back to this guide or seek assistance from the [directus community][chat]. Happy hosting!

[chat]: https://directus.chat 'Directus Community Chat'
[quickstart]: https://docs.directus.io/getting-started/quickstart.html 'Directus Quickstart'
