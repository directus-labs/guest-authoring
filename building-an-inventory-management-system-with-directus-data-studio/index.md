---
title: 'Building an Inventory Management System with Directus Data Studio'
description: 'In this article, I will guide readers through the process of creating an inventory management system using Directus Data Studio. The system will allow users to set up inventory systems, track and manage their inventory, including adding new items, and updating stock levels. Furthermore, readers will learn how to set up roles and assign permissions to other prospective users.'
author:
  name: 'Victory Tuduo'
  avatar_file_name: './image.jpg'
---

## Introduction
In our data-driven world today, efficient information management is essential for organizations of all kinds, from businesses to schools and hospitals. Proper information management helps prevent losses, reduce costs, and ensure that organizations are always aware of their stock levels, thus avoiding any unfortunate out-of-stock situations. For something as sensitive as a hospital, keeping track of both office supplies and medical inventory is crucial. With a reliable documenting system (commonly known as an inventory system) they can monitor stock levels, track shelf life, identify the most consumed items, and restock as needed, making the entire process easier to manage and far more economical.

In this article, we'll guide you through building an inventory management system using Directus Data Studio. This tool, built on the open data platform Directus, enables the collection, storage, management, and distribution of data in an accessible and transparent way. Directus Data Studio allows users to create intuitive interfaces to interact with the database and manipulate data to meet specific needs. Let's dive in and see how we can streamline your inventory management process with Directus.

### What is Directus?
Directus is an open-source data platform that helps users create and manage their data using several methods. It provides a database for the user and also provides tools for managing, visualizing, and interacting with the data. Additionally, Directus can function as a headless CMS, making data manipulation straightforward for both technical and non-technical users.

## Before You Start
Before you get started, you'll need a user account  on Directus. If you do not already have one, follow the steps below to create an account:
- set up a Directus cloud user account using the [registration link](https://directus.cloud/register). 

- After signing up and creating an account, you will be prompted to fill in the details for your name and team information which can also be changed later on. Fill in all the necessary information to complete the user creation process. 

- After the registration process is completed you will be required to set up a new project using Directus. For this tutorial, I have created a new project *inventory-system*. Create a project of your choice name, open up the project, and log in with your account credentials to access the project dashboard. You will be presented with an interface similar to the image below:

![Directus dashboard](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722118321903_Screenshot+2024-07-27+at+23.11.42.png)
<!-- ## Your Sections Here -->

## Building an Inventory Management System
Having provided a background of what Directus and the Data Studio are in the previous section, we will demonstrate how an inventory management system can be set up using these technologies, in the subsequent sections.

### Creating a new collection
From the interface in the image above, we can manage the users, content, and files on the project. We will start by creating a collection for our inventory products. Click on the **Create Collection** button and enter *products* for the table name, leave the other options in their default state, and create the collection. 

![created products collection](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722118917321_Screenshot+2024-07-27+at+23.21.46.png)


In the image above, an *id* field has been created for us, serving as the primary key for our collection. For our product inventory system, we will add fields for product name, product expiry date, and product quantity. Directus offers a variety of options for defining product data types, selection methods, data orchestration (in the form of grouped data), and relationships between fields. For the product name and product quantity fields, we will use the **Text** input type, and for the product expiry date, we will use the **Date** type.

product name

![creating product name field](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722119645516_Screenshot+2024-07-27+at+23.31.22.png)


product quantity

![creating product quantity field](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722121239780_Screenshot+2024-07-28+at+00.00.11.png)


product expiry date

![creating product expiry date](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722119697024_Screenshot+2024-07-27+at+23.33.35.png)


In the images above we created the fields for our inventory system, and currently have four fields in total within our project. Note that although *product_name* and *product_quantity* are both **Text** input types, the former uses a type string, while the latter uses a type integer. This is due to the datatypes these respective fields will contain later on.


![products collection fields](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722119764889_Screenshot+2024-07-27+at+23.35.52.png)


Using these fields, we will create product entries as if we are maintaining inventory records for a hospital. In the following section, we will populate and organize data using these fields.

### Adding Items to the Collection
In this section, we will create new items within the *products* collection, using the fields we defined in the previous section. To begin, select **Content** from the left side menu and click on **Create new**.


![creating new items in the product collection](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722120347992_Screenshot+2024-07-27+at+23.45.05.png)


When we click on the **Create Item** button, we will be prompted to fill in content using the earlier defined fields for each item. We have also previously set all fields as required, as their information is crucial to the inventory tracking process.


![items added to table](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722121683567_Screenshot+2024-07-28+at+00.06.15.png)


In the screenshot above, I have added five drugs to the *products* collection. Currently, our data is represented in as a table and is unorganized. We can rearrange the table by dragging and dropping the column headers and perform search operations using the query field at the top.


![rearranging items](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722122399080_ScreenRecording2024-07-28at00.10.19-ezgif.com-video-to-gif-converter.gif)


In the GIF above, we rearranged the table columns and also demonstrated searching using the product name and quantity.

### Custom Filters
In addition to the search filter, we can create custom filters to query our database. Take for instance, one might have hundreds of drugs in the data and want to know what drugs are within a certain range of quantity. Using the *product_quantity* field of type number, we can create such a filter:


![sorting using custom filters](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722123849619_ScreenRecording2024-07-28at00.22.56-ezgif.com-video-to-gif-converter.gif)


From the above GIF, we created a filter that checked the *product_quantity* field and returned drugs whose quantities were equal to or less than 100.  Similar options are available to check against the date field type.

For the string field *product_name,* we can query the items to determine if any of them meet the following criteria:


![custom filtering](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722124001022_ScreenRecording2024-07-28at00.26.28-ezgif.com-video-to-gif-converter.gif)


Note that these operations are also case-sensitive. More complex filters can be created using **AND/OR** to perform logical operations on different filters to get data that meets multiple criteria.


![logical and/or](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722125483012_image.png)


Here, using the **AND/OR** operator, we are filtering for data with a product name containing the letter “p” and a product quantity equivalent to 100.

### Sorting the table
Using the *product expiry date* we can sort the table using the date for each item so that we can get items with closer expiry dates at the top of the table. The image below shows a reorganized table sorted using the dates in descending order:


![sorted items using date](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722125361795_image.png)



### Managing user access

Also, from the admin dashboard, we can add users, manage access, and assign roles to to-be users on the platform. These roles are used to provide a level of restriction to the type of activities that can be carried out by different users. For example, an admin can create, read, update, and delete content, but other user roles can be restricted to only update or read permissions.

To create a new user role, click on **Settings** from the left side menu and select **Access Control**. In this section, clicking on the **circle plus icon** opens a modal to create new roles. By default, we have two roles already defined: an Admin role and a Public role. Here we will enable read operations for the Public role and create a new role for *Secretary* with the operations **Read** and **Update** allowed.


![creating user roles and managing permissions](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722127010914_ScreenRecording2024-07-28012247-ezgif.com-video-to-gif-converter.gif)


Using the access control feature, we can create roles and change the level of permissions for different operations in any role. There are 3 options to choose from namely: full access, no access and use custom. The use custom option allows permissions to be available only when certain conditions are met. These conditions are similar to the conditions we specified when creating a query. For example, we can set that a particular role can delete an item off the table only when the *product_quantity* is equal to zero.

### Assigning Roles and Inviting Users
To invite new users to the project and assign them to one of the created roles, navigate to **User Directory** from the left side menu. We have an option to manually create new user data or send an invite to the user using their email address. On this page, we can also sort users by role and use search and custom filters to query the data.


![adding users and assigning roles](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722128199928_ScreenRecording2024-07-28014358-ezgif.com-video-to-gif-converter.gif)


In the GIF above, we defined user information for the new user in the first method, while for the second we sent an invite to the user by email to have them set up their password and fill in their information when they accept the invite.

## Summary
Congratulations on concluding this tutorial. With this tutorial, we gained insight into how to use Data Studio and the process of setting up an inventory management system on Directus without having extensive technical knowledge.
