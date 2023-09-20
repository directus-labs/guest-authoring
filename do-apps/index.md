---

title: 'Deploy to Digital Ocean Apps'

description: 'Launch Directus on DigitalOcean App Platform: Easy, fast deployment for efficient data management in the cloud'

author_name: 'Matthew Ruffino'

avatar_file_name: '1676510086127.jpg'

---

  

## Introduction

  

Directus has quickly become a big part of my life in such a short amount of time. What began as a journey for finding the best backend turned into an passion-filled adventure introducing  it to everyone I know and using it  everywhere I can.

  

And today, I am proud to be able to share that passion with you, to help you learn how to "quickly" deploy Directus onto the Digital Ocean App Platform. Using DO Apps will create a fresh build of Directus for you with any extensions you want to add by simply editing the included Dockerfile. 

Now you might be thinking why would I want to use the DO App Platform? Well it's a fully managed solution. You do not have to worry about any underlying infrastructure and it can auto-scale automatically to keep your backend running smoothly.

  

## Before You Start
You will need to decide if you are wanting to deploy for development or for production. As this will change the way you configure your app and possibly database. 

If you configuring for development, you do not need a redis server or managed database. (you can attach a development database during configuration steps that can be upgraded to a managed one at a later time)

If you are configuring for production (which we are in this post), you will need to setup a managed database and redis droplet. 

You will also need to fork my [repo](https://github.com/BlackDahlia313/directus-docker-do) that includes a Dockerfile you can customize  to your own needs.

- ### Create Managed DB

	First create a managed Postgres DB. This is as easy as going to the [Databases](https://cloud.digitalocean.com/databases) page and following the steps. If you already have a database, you can skip this step.

- ### Create Redis Droplet
	You will need to create a redis droplet to keep synchronization across your different containers. 

	You can easily create a redis droplet by following these steps [here](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-20-04). If you already have an external redis server, you can use that too.

	##### Note: This will get easier over time as Directus has made big changes to redis support recently and is expected to support clustered redis soon which is what the managed redis from DO uses.

- ### Create a Spaces Object Storage
	File uploads are not persistant unless you have an external volume. With apps platform, that would be using the Spaces Object Storage which works very similiarly to S3. You can setup storage by visiting this [here](https://cloud.digitalocean.com/spaces).

## Create The App
![Go to apps and click create](1.jpg)
Head over to the apps section and click on "Create App". This is where your adventure begins.

![enter image description here](2.jpg)
On this page you will need to select the repo that you [forked over](https://github.com/BlackDahlia313/directus-docker-do) earlier. Go ahead and connect your account to github and you will be able to see your repo.

![enter image description here](3.jpg)
You when you reach this page you will see a random generated name for your app. Below it, you will see the directus-docker-do or whatever you have decided to name the repo you forked earlier. This will be declared as a webservice, which is what we want for Directus.

You will also be able to edit the plan for your app (how many containers and how powerful) and also to attach a managed DO database.

![enter image description here](4.jpg)
First let's select the managed database you created. While you don't have to "attach" your managed db to the app, it's a best practice as you don't have to worry about any firewall/allow list settings. You can also view the system resources from the same panel.
<!-- ## Your Sections Here -->

  

## Summary