---
title: 'Building an Inventory Management System with Directus Data Studio'
description: 'In this article, I will guide readers through creating an inventory management system using Directus Data Studio. The system will allow users to set up inventory systems, and track and manage their inventory, including adding new items and updating stock levels. Furthermore, readers will learn how to set up roles and assign permissions to other prospective users.'
author:
 name: 'Victory Tuduo'
 avatar_file_name: './image.jpg'
---

## Introduction

In our data-driven world today, efficient information management is essential for organizations, from businesses to schools and hospitals. Proper information management helps prevent losses, reduce costs, and ensure that organizations are always aware of their stock levels, thus avoiding any unfortunate out-of-stock situations. For something as sensitive as a hospital, keeping track of office supplies and medical inventory is crucial. With a reliable documenting system (commonly known as an inventory system) they can monitor stock levels, track shelf life, identify the most consumed items, and restock as needed, making the entire process easier to manage and far more economical.

In this article, we'll guide you through building an inventory management system using Directus Data Studio. This tool, built on the open data platform Directus, enables the collection, storage, management, and distribution of data in an accessible and transparent way. Directus Data Studio allows users to create intuitive interfaces to interact with the database and manipulate data to meet specific needs. Let's dive in and see how we can streamline your inventory management process with Directus.

### What is Directus?

Directus is an open-source data platform that helps users create and manage their data using several methods. It provides a database for the user and also provides tools for managing, visualizing, and interacting with the data. Additionally, Directus can function as a headless CMS, making data manipulation straightforward for both technical and non-technical users.

## Before You Start

Before you get started, you'll need a user account on Directus. If you do not already have one, follow the steps below to create an account:

- set up a Directus cloud user account using the [registration link](https://directus.cloud/register).

- After signing up and creating an account, you will be prompted to fill in the details for your name and team information which can also be changed later on. Fill in all the necessary information to complete the user creation process. 

- After the registration process is completed you will be required to set up a new project using Directus. For this tutorial, I have created a new project *inventory-system*. Create a project of your choice name, open up the project, and log in with your account credentials to access the project dashboard. You will be presented with an interface similar to the image below:

![Directus dashboard](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722118321903_Screenshot+2024-07-27+at+23.11.42.png)
<!-- ## Your Sections Here -->

## Getting Familiar with the Data Studio

Having provided a background of what Directus and the Data Studio are in the previous section, we will first look at how to navigate the Data Studio, after which we will build an inventory management system using this knowledge in the subsequent sections.

### Creating a New Collection

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

In the screenshot above, I have added five drugs to the *products* collection. Currently, our data is represented as a table and is unorganized. We can rearrange the table by dragging and dropping the column headers and perform search operations using the query field at the top.

![rearranging items](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722122399080_ScreenRecording2024-07-28at00.10.19-ezgif.com-video-to-gif-converter.gif)

In the GIF above, we rearranged the table columns and also demonstrated searches using the product name and quantity.

### Custom Filters

In addition to the search filter, we can create custom filters to query our database. Take for instance, one might have hundreds of drugs in the data and want to know what drugs are within a certain range of quantity. Using the *product_quantity* field of type number, we can create such a filter:

![sorting using custom filters](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722123849619_ScreenRecording2024-07-28at00.22.56-ezgif.com-video-to-gif-converter.gif)

From the above GIF, we created a filter that checked the *product_quantity* field and returned drugs whose quantities were equal to or less than 100.  Similar options are available to check against the date field type.

For the string field *product_name,* we can query the items to determine if any of them meet the following criteria:

![custom filtering](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722124001022_ScreenRecording2024-07-28at00.26.28-ezgif.com-video-to-gif-converter.gif)

Note that these operations are also case-sensitive. More complex filters can be created using **AND/OR** to perform logical operations on different filters to get data that meets multiple criteria.

![logical and/or](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722125483012_image.png)

Here, using the **AND/OR** operator, we are filtering for data with a product name containing the letter “p” and a product quantity equivalent to 100.

### Sorting the Table

Using the *product expiry date* we can sort the table using the date for each item so that we can get items with closer expiry dates at the top of the table. The image below shows a reorganized table sorted using the dates in descending order:

![sorted items using date](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722125361795_image.png)

### Managing User Access

Also, from the admin dashboard, we can add users, manage access, and assign roles to to-be users on the platform. These roles are used to provide a level of restriction to the type of activities that can be carried out by different users. For example, an admin can create, read, update, and delete content, but other user roles can be restricted to only update or read permissions.

To create a new user role, click on **Settings** from the left side menu and select **Access Control**. In this section, clicking on the **circle plus icon** opens a modal to create new roles. By default, we have two roles already defined: an Admin role and a Public role. Here we will enable read operations for the Public role and create a new role for *Secretary* with the operations **Read** and **Update** allowed.

![creating user roles and managing permissions](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722127010914_ScreenRecording2024-07-28012247-ezgif.com-video-to-gif-converter.gif)

Using the access control feature, we can create roles and change the level of permissions for different operations in any role. There are 3 options to choose from namely: full access, no access and use custom. The use custom option allows permissions to be available only when certain conditions are met. These conditions are similar to the conditions we specified when creating a query. For example, we can set that a particular role can delete an item off the table only when the *product_quantity* is equal to zero.

### Assigning Roles and Inviting Users

To invite new users to the project and assign them to one of the created roles, navigate to **User Directory** from the left side menu. We have an option to manually create new user data or send an invite to the user using their email address. On this page, we can also sort users by role and use search and custom filters to query the data.

![adding users and assigning roles](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722128199928_ScreenRecording2024-07-28014358-ezgif.com-video-to-gif-converter.gif)

In the GIF above, we defined user information for the new user in the first method, while for the second we sent an invite to the user by email to have them set up their password and fill in their information when they accept the invite.

## Building an Inventory Management System

Now that we are familiar with working with Directus Data Studio, in this section we will set up an inventory management system using the features we have discussed so far. We will create additional collections for suppliers, office sundries, pharmacies, and orders. We will also create relationships between these collections to demonstrate how data can be linked and queried across different collections. Finally, we will create a dashboard to visualize the data in the collections and generate reports based on the inventory data.

### Creating New Collections

For additional collections, we will create the following fields:

- A *pharmacies* collection with fields for *pharmacy name* and *drugs sold*. The *drugs sold* field will be in a relationship with the *products* collection.
![phamarcies collection](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722604222305_Screenshot+2024-08-02+094814.png)

- A collection *office sundries* with fields *item name*, *item quantity*, *cost per unit*, *total cost* and a *reorder quantity*.
![office sundries collection](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722604544626_Screenshot+2024-08-02+100456.png)

- A *suppliers* collection with fields *supplier name* and *item sold*. The *item sold* field will be a relationship with the *office sundries* collection.
![suppliers collection](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722604674681_Screenshot+2024-08-02+101021.png)

- And finally, an *orders* collection. For this collection, we will add the *id*, *status*, *date created*, and *user created* optional fields while creating the collection.
![orders collection optional fields](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722605813453_Screenshot+2024-08-02+102911.png)

These fields will allow users to track the date the order details are created and managed, know who created the order information, and also create and edit orders in draft mode until it is time to have them published. Next, we will add two fields *drugs to be ordered* and *sundries to be ordered* which will establish a relation with *products* and *office sundries* collections respectively. Each field will allow items from the related collection to be added in a table data form.

*drugs to be ordered*
![drugs to be ordered field](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722609215484_ScreenRecording2024-08-02103736_wmv1-ezgif.com-video-to-gif-converter.gif)

*sundries to be ordered*
![sundries to be ordered](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722606088614_Screenshot+2024-08-02+103829.png)

### Adding Items to New Collections

In the *office sundries* collection we will add office stationary items using the fields we had defined in the previous section.
![added item to the office sundries collection](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722604939764_Screenshot+2024-08-02+101933.png)

In the image above, we added inventory for two items, a stapler and a set of A4 papers. We also specified the quantity available and the cost per unit (the symbol for the currency can be specified in the settings. I have specified pounds for this example), the total cost, and the reorder quantity. The reorder quantity is the quantity at which the item should be reordered to avoid running out of stock.

In a similar fashion, we will add items to the *pharmacies* and *suppliers* collections. The *pharmacies* collection will contain the names of pharmacies and the drugs they sell, while the *suppliers* collection will contain the names of suppliers and the items they sell, which are linked to the *products* and *office sundries* collection respectively.

Supplier information can be added using the field as shown below:

![Adding data to suppliers collection](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722609649329_ScreenRecording2024-08-02102153_wmv1-ezgif.com-video-to-gif-converter.gif)

We can add pharmacy information as shown in the GIF below:

![Adding data to pharmacies collection](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722607185597_ScreenRecording2024-08-02100143-ezgif.com-video-to-gif-converter.gif)

We are using the *id* field as the foreign key when creating relationships between the collections. The *id* field is the primary key for each collection and is used to link data across different collections.

We can also create orders and reference the appropriate collections, in the *orders* collection:

![drugs to be ordered field](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722609215484_ScreenRecording2024-08-02103736_wmv1-ezgif.com-video-to-gif-converter.gif)

### Creating Dashboard Insight to Visualize Data

To create a visual dashboard, from the left side navigation, select **Insights** and click on **Create Dashboard**. In the new window that opens, we will select the data we want to visualize and the type of visualization we want to use. We will create a dashboard that shows the quantity of products in the inventory and helps identify at a glance product expiry dates and products at low quantities.

![Creating new dashboard](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722608570570_Screenshot+2024-08-02+105459.png)

In the image above, I created a new dashboard and named it *hospital inventory overview*. Within this dashboard, we will add panels we will use to monitor data from our collections. This approach will help us to easily track the number of products in the inventory, the number of products that are about to expire, and the number of products that are out of stock.

We will first create the **Line Chart** panel with the following information:
![Added chart for products collection](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722609703406_Screenshot+2024-08-02+110002.png)

Next, we will add a **Label** with the following data:
![Added label for products chart](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722609661377_Screenshot+2024-08-02+111519.png)

Above, we added line charts to show the quantities of products and sundries available and help keep track of them. Using this chart, we can easily see when quantities are low and need to be reordered. Using the **Labels** option, we added titles to each chart. We will duplicate the chart and label and use it in a similar manner for the *office sundries* collection. At the end, your dashboard will look similar to the image below:

![dashboard look](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722610088913_image.png)

### Generating Insights Based on Data

All drugs in our *products* collection have an expiry date, and it is crucial to monitor the shelf life so one doesn't end up with expired products. To do this, we will use the **List** panel to order products by their expiry dates. This will help us monitor the shelf life of our products and ensure that we do not have expired products in our inventory.

To keep track of product quantities, we can add a **List** panel that shows products with quantities less than 100:

![list for product quantites](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722610543416_image.png)

A title *Less than 100* can be added on top of this to make it easier to identify the purpose of the data at a glance.

Finally, we will add another **List** panel to track products by their expiry date and arrange them in that order. The display template option will contain the product name, quantity, and expiry date.

![List for product by expiry date](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722611373822_image.png)

At the end, we have the following dashboard:

![Dashboard final look](https://paper-attachments.dropboxusercontent.com/s_07AA095E1E97E60FEBB8F5C4D8B67B7165C803D1B9DA3869CB29C0C2E5110EBF_1722611346632_image.png)

## Summary

Congratulations on concluding this tutorial. With this tutorial, we gained insight into how to use Data Studio and the process of setting up an inventory management system on Directus without having extensive technical knowledge.