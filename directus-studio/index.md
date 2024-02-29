---
title: 'Building a Customer Support Ticket Form with Directus Studio'
description: 'Learn how to build a customer support ticket form with Directus Studio, enhancing service efficiency and user experience in an easy steps.'
author:
  name: 'Ayodele Aransiola'
  avatar_file_name: 'ayodele.png'
---

## Introduction
Building a user-friendly and useable product is important, but an effective customer feedback and support is pivotal for business success. [Directus](https://directus.io), an open-source flexible platform that offers solutions such as the Headless CMS, Backend-as-a-service more amazing features that can be tailored to various needs, including creating a dynamic support ticket form for your product. This article will guide you through using Directus Studio to build a support ticket form that allows customers to submit their queries or issues on your product offerings.

## Before You Start
1. Ensure that you have access to [Directus Cloud](https://directus.cloud/) or local development environment using [docker](https://docs.directus.io/getting-started/quickstart.html).

2. Plan out the fields and data types you'll need for your support ticket form. Common fields include customer name, contact information, issue category, issue description, status, and submission date.

3. Think about who will be using the system. You'll likely need roles for customers (to submit tickets) and support staff (to view and resolve tickets). Consider the permissions each role will need.
<!-- ## Your Sections Here -->
## Designing the Support Ticket Schema
The major part of the support ticket system is its schema. In Directus Studio:

1. **Create a New Collection:** Name it `support_tickets`. This collection will store all customer queries. You can decide which field will be the primary key of your collection. Creating a fresh collection, Directus suggest an initial field, `id` as the unique key of your collection. Rename the id field to `ticket-id` which is the default and leave the type as auto-incremented integer.
```
:::hint

After giving your collection a name, the next page allow you to select some fields such as `status`, `created on`, `updated on`, and `updated by`. You can add any of these field to your form based on your needs.

:::
```
2. **Define the Fields:**
   - `customer_name` and `customer_email`: The data type can be string or text. This allow you to identify the customer.
   - `issue_type` (Dropdown): Define lists like 'Billing', 'Technical Issue', 'General Inquiry', etc.
   - `description` (Text Field): Detailed description of the issue.
   - `status` (Dropdown): Options like 'Open', 'In Progress', 'Resolved'.
   - `created_at` (Date Field): Automatically captures the ticket submission time.

**:bulb: Note**:
Make sure to check the "required" option for critical fields like customer_email and issue_type to prevent incomplete submissions.

![fields and layout of a sample support ticket form](schema.png)

Ensure each field is correctly set up for its purpose, especially concerning data types, placeholder where necessary, and default values.

## Configuring Roles and Permissions
Directus’s flexibility in permissions allows you to control access precisely:
![Roles and permissions tab on Directus studio](access-control.png)

1. **Navigate to Roles & Permissions** in Directus Studio and create a new role named `Customer` or `User`.
2. **Set Permissions for the `Customer` Role:** Allow `read` and `create` access to the `support_tickets` collection. It's crucial to restrict `read` access to only their tickets, ensuring privacy and security.
![Applying rules to what user can see](rule-setting.png)

```
:::Rule Setting

Directus allows you to apply filters to permissions. To ensure users can only see their own tickets, you'll apply a filter on the Read permission. According to the image above, you'll allow the user to see only the tickets created that the email is equal to their own email. You need to choose the field that links the ticket to the user. This is often a user ID or user email field in the ticket collection that references the user who submitted the ticket. Let's say this field is named `Customer_email`.
Set the condition to match the current user's email: for the field Customer_email, select the operator equals/contains, and for the value, you should use a special variable that represents the current user's email. In Directus, this is typically represented as `$CURRENT_USER.email`. You can read more about user fileds on the [documentation](https://docs.directus.io/reference/system/users.html).

Test this setting with a test account before going live.

:::
```

## Inviting Customers to Use the Form
To allow customers to submit tickets, you must add them to Directus and assign them the `Customer`/`User` role based on what the configuration is:

1. **Add Users in Directus Studio:** Go to the Users section and invite customers via email.
![Invite user to your directus platform](user-directory.png)
2. **Assign Roles:** Ensure each customer user is assigned the `Customer` role to access and submit the support ticket form.

## Managing Support Tickets
Admins and support staff can manage tickets directly within Directus Studio:

- Use the collection interface to view, sort, and filter tickets.
- Update ticket statuses as you work through them, and use the `customer_email` field to respond to customers directly.

Using Directus Studio to build a support ticket form offers a customizable and scalable solution for managing customer queries and issues. By following the steps outlined in this article, businesses can improve their support processes, ensuring customers receive timely and effective assistance.

**Best Practices:** Regularly review and refine your support ticket process based on customer feedback and ticket analysis.

## Summary
In this article, you have been able to learn how to use Directus to create a customer support ticket system. Key skills include:
- ability to navigate and utilize Directus Studio for creating and managing collections, a fundamental aspect of building custom applications.
- The ability to create and implement a schema for a support ticket system, including the selection of fields and data types, showcases a deep understanding of database management and user experience design.
- You have also been able to understand Directus's role-based access control (RBAC) system.

## Next Steps
To build upon the skills learnt in this article, consider the following steps:

1. Read more on the amazing offerings of Directus on their [Documentation](https://docs.directus.io/): Explore the Directus Documentation, especially sections related to API usage, advanced permission configurations, and webhooks. This will enhance your ability to customize and extend the functionality of your support ticket system.

2. Sign Up for [Directus Cloud](https://directus.cloud/): If you haven't already, sign up for Directus Cloud to experience the platform's full capabilities without the need to manage your infrastructure. 

3. Setup Directus Locally: Watch this [Youtube video](https://www.youtube.com/watch?v=ZOCfBBTMtoE) that walk you through how to setup directus locally on your PC.

