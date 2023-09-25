  

  

---

  

  

  

title: 'Deploy to Digital Ocean Apps'

description: 'Launch Directus on DigitalOcean App Platform: Easy, fast deployment for efficient data management in the cloud'

author_name: 'Matthew Ruffino'

avatar_file_name: '1676510086127.jpg'

  

---

  

## Introduction

  

Directus has become a central part of my technological journey. My exploration for the ultimate backend has led me to embrace the rabbit passionately. Now, I'm excited to help you deploy Directus on the Digital Ocean App Platform efficiently. With DO Apps, creating a new Directus setup, inclusive of any extensions you might want, is a breeze with a simple Dockerfile.

  

You might be thinking, "Why choose the DO App Platform?" Its ease of management is a key reason. With no underlying infrastructure to worry about and automatic scaling, it's perfect for both rapid development and full-scale production. If you already ingrained into the DO product line, than this is a great choice for you.

  

If you are looking for something that requires absolutely no setup, with automatic rolling updates than look not further than [Directus Cloud.](https://directus.io/pricing/cloud)

  

## Before You Start

  

There are required parts to run Directus within the DO App platform. I will give a brief list and what they are used for.

  

- Managed DB -- You will need a DB for Directus. This can be hosted within DO or outside.

- Redis Droplet -- If you plan to have a clustered app, you will need to setup a Redis droplet to be able to synchronize across multiple instances. (you will be soon be able to use managed redis from DO)

- DO Spaces -- This is needed for your files to be uploaded and remain persistant. This follows the same S3 protocol as Amazon S3. You can use any storage provider that Directus officially supports.

  

-  **Determine Your Deployment Purpose**: Are you setting up for development or production? This will dictate your app and database configurations.

-  **Development Environment**: No need for a Redis server or managed database. Just add a development database during setup, which can be upgraded later. Redis is only for clustered dev environments.

-  **Production Deployment (our focus)**: You'll require a managed database and a Redis droplet.

  

-  **Build your Dockerfile Repository**: Get the customizable Dockerfile from my [repository](https://github.com/BlackDahlia313/directus-docker-do) or build your own below.

  

-  **Extensions**: You can easily incorporate new extensions into Directus by tweaking the Dockerfile.

  

### Steps to Set Up Infrastructure

  

1.  **Create Dockerfile repo**:

- First we must create our Dockerfile. This is very easy and can be hosted in Github or Gitlab.

- Create a new repo (name it anything), than in the root of the new repo, create a blank Dockerfile.

- Below is an example Dockerfile that you can use. After creating the file. Hit commit the change to your repo! Also while we are here, Let's break it down so you can see what we are doing within the dockerfile.
	- We use corepack as it allows us to use pnpm which is the package manager we are using with Directus.
	- We enabled 8.7.6 of pnpm as its the latest version of this writing.
	- We pull the latest version of Directus in FROM. You can pick a specific version by looking at the available tags [here](https://hub.docker.com/r/directus/directus/tags). (this is also how you update Directus)
	- Chown gives the proper permissions needed for node to access the files
	- We expose port 8055 for Directus, DO Apps can read this and handles this automatically for us during deployment.
	- The RUN command allows us to bake in any extensions we want to include. Make sure to include a && and than include your install command for the extension. I have included [Directus Extension Field Actions](https://github.com/utomic-media/directus-extension-field-actions) and [Flows Manager](https://github.com/baguse/directus-extension-flow-manager) extensions as an example.
	- The bootstrap command is ran first to pull info from the ENV to either install the database (if it's empty) or migrate it to the latest version (if it already exists and has missing migrations).
	- The start command is now ran to start Directus!🎉



  
  

			# syntax=docker/dockerfile:1.4

			## Black Dahlia was here
			####################################################################################################
			## Create Your Directus Production Image

			FROM directus/directus
			USER root
			RUN corepack enable \
			&& corepack prepare pnpm@8.7.6 --activate \

			# Currently required, we'll probably address this in the base image in future release

			&& chown node:node /directus
			EXPOSE 8055
			USER node

			RUN pnpm install directus-extension-field-actions && pnpm install directus-extension-flow-manager && pnpm config set auto-install-peers true
			CMD : \
			&& node /directus/cli.js bootstrap \
			&& node /directus/cli.js start \
			;

  
  

##### Note: By choosing a version for Directus in your dockerfile, you can "version control" your backend to prevent accidental upgrades. If you leave the tag as it is above in the example dockerfile, it will always pull the latest version of Directus that is available. Anytime your deployment sees a commit, a build will automatically be ran for your App.
  

2.  **Create Managed DB**:

- Start with a managed Postgres DB. Navigate to the [Databases](https://cloud.digitalocean.com/databases) page and follow the instructions. If you've already got a database, move on to the next step.

  

3.  **Create Redis Droplet**:

- Establish a Redis droplet for synchronization across containers. Follow this [tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-20-04). If you have an existing external Redis server, that works too.

- As Directus evolves its Redis support, this process will become even more straightforward in the future as we will be able to use DO's Managed Redis service.

  

4.  **Setup Spaces Object Storage**:

- Persistent file uploads require external volume. On the apps platform, this means using the Spaces Object Storage, similar to S3. Set it up [here](https://cloud.digitalocean.com/spaces).

  

## Create The App

  

1. Navigate to the apps section and click "Create App".

2. On the subsequent page, link the repo that you forked earlier. Connect your account to GitHub to access your repository.

3. You'll encounter a page with an auto-generated name for your app. You should see the name of your forked repository below. This setup is recognized as a web service, fitting for Directus.

4. Adjust the app's plan to your requirements, including the number of containers and their capabilities. Here, you can also connect your managed DO database.

5. Opt for the managed database you crafted earlier. Attaching the database to the app is a recommended best practice, ensuring smooth firewall and security settings. Then, select your database cluster and user.

6. Define the base resources you desire for your backend. If you're running two or more containers, Redis becomes a necessity.

7. Input the necessary [environment variables](https://docs.directus.io/self-hosted/config-options.html) for Directus. A set of basic variables will help you get started, but ensure you complete all necessary fields. The bulk editor simplifies this task, and you can always return to edit or add more variables later.

  
  

	    KEY=enterakey
	    SECRET=enterasecret
	    DB_CLIENT=pg
	    DB_HOST=dbhost    
	    DB_PORT=25060   
	    DB_DATABASE=your_db 
	    DB_USER=doadmin
	    DB_PASSWORD=dbpass
	    DB_SSL__CA=""
	    STORAGE_LOCATIONS=digitalocean
	    STORAGE_DIGITALOCEAN_DRIVER=s3
	    STORAGE_DIGITALOCEAN_KEY="yourstoragekey"
	    STORAGE_DIGITALOCEAN_SECRET="yoursecret"
	    STORAGE_DIGITALOCEAN_ENDPOINT="endpointurl"
	    STORAGE_DIGITALOCEAN_BUCKET="bucketname"
	    STORAGE_DIGITALOCEAN_REGION="selectedregion"
	    REDIS_HOST="hostip"
	    REDIS_PORT=6379
	    REDIS_PASSWORD="redis-password"
	    CACHE_ENABLED=true  
	    CACHE_STORE=redis  
	    CACHE_AUTO_PURGE=true
	    MESSENGER_STORE=redis
	    SYNCHRONIZATION_STORE=redis
	    PUBLIC_URL=${APP_URL}
    
	    ADMIN_EMAIL=youradmin@mail.com
	    ADMIN_PASSWORD=myfirstadminpassword

  

##### For generating your KEY & SECRET, you can use this tool: [generate-secret.vercel.app/32](https://generate-secret.vercel.app/32).

  

8. For database connection settings, you have two options. You can either utilize Digital Ocean's predefined variables or stipulate your own. If you wish to delve deeper into Digital Ocean's environment variables, consult their [documentation](https://docs.digitalocean.com/products/app-platform/how-to/use-environment-variables/). Additionally, don't forget to incorporate the CA certificate, available within the database connection settings.

  

	##### Moving forward, you'll have the opportunity to designate a name for your application. You can also decide whether to integrate it into a DO project. For those unfamiliar, a DO project is a handy tool to collectively label and manage multiple resources.

  

9. Once you've reviewed your configurations and confirmed everything's in order, proceed by clicking "Create App". This action kickstarts the building and deployment phase.

  

## Deployment

  

During this step, all that's left for you is to patiently wait as your tailored Directus application undergoes construction and deployment. Digital Ocean will cache the image you've just built, ensuring it's available for spawning new containers during any auto-scaling events.

  

Post build completion, the deployment process takes the stage. This phase initializes Directus, oversees its bootstrap operations, and verifies the backend's functionality. If everything transpires without a hitch, you'll witness a successful deployment notification.

  

Upon entering the application dashboard, you'll encounter a comprehensive view of your application's health. It provides insights into recent deployments, facilitates forced deployments, and more. Prominently displayed is your generated application URL. If you ever feel the need to append a fresh URL, navigate to the settings tab. As an added convenience, SSL certification is already managed for you.

  

## Summary

  

This manual provides an uncomplicated route to deploying Directus on the DO App Platform. If you're acquainted with Digital Ocean's offerings, you'll find its App Platform an impeccable choice for backend hosting, further enriched by the myriad features the platform delivers.

  

**Key Highlights**:

  

- Any alterations to the Environment Variables or Dockerfile activate instantaneous re-builds and deployments.

- All deployments are executed smoothly, devoid of any downtime. Any unsuccessful builds will default back to the last triumphant version.

- Your data's integrity is preserved with Storage and Managed DB, ensuring your data's safety even if you decide to suspend the app for subsequent migrations.

  

Should any questions or hurdles arise, feel free to reach out to me on the Directus Discord platform. I'm always here to assist!

  

-onelove
