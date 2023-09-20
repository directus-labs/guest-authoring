---

title: 'Deploy to Digital Ocean Apps'

description: 'Launch Directus on DigitalOcean App Platform: Easy, fast deployment for efficient data management in the cloud'

author:

name: 'Matthew Ruffino'

avatar_file_name: '1676510086127.jpg'

---

  

## Introduction

  

Directus has quickly become a big part of my life in such a short amount of time. What began as a journey for finding the best backend turned into an passion-filled adventure introducing it to everyone I can.

  

And today, I am proud to be able to share that passion with you to help you learn how to quickly Deploy Directus onto the Digital Ocean App Platform.

  

There are some caveats at the moment, but this hopefully helpful how-to will be updated as both DO and Directus evolve over time.

  

## Before You Start
You will need to decide if you are wanting to deploy for development or for production. As this will change the way you configure your app and possibly database. 

If you configuring for development, you do not need a redis server or managed database. (you can attach a development database during configuration steps that can be upgraded to a managed one at a later time)

If you are configuring for production (which we are in this post), you will need to setup a managed database and redis droplet. 

- ### Create Managed DB

	First create a managed Postgres DB. This is as easy as going to the [Databases](https://cloud.digitalocean.com/databases) page and following the steps. If you already have a database, you can skip this step.

- ### Create Redis Droplet
	You will need to create a redis droplet to keep synchronization across your different containers. 

	You can easily create a redis droplet by following these steps [here](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-20-04). If you already have an external redis server, you can use that too.

	##### Note: This will get easier over time as Directus has made big changes to redis support recently and is expected to support clustered redis soon which is what the managed redis from DO uses.

- ### Create a Spaces Object Storage
	File uploads are not persistant unless you have an external volume. With apps platform, that would be using the Spaces Object Storage which works very similiarly to S3. You can setup storage by visiting this [here](https://cloud.digitalocean.com/spaces).



<!-- ## Your Sections Here -->

  

## Summary