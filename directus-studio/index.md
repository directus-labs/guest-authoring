---
title: "Building a Customer Support Ticket Form with Directus Studio"
description: "Learn how to build a customer support ticket form with Directus Studio, enhancing service efficiency and user experience in an easy steps."
author:
  name: "Ayodele Aransiola"
  avatar_file_name: "ayodele.png"
---

## Introduction

Building a user-friendly and useable product is important, but an effective customer feedback and support is pivotal for business success. [Directus](https://directus.io), an open-source flexible platform that offers solutions such as the Headless CMS, Backend-as-a-service more amazing features that can be tailored to various needs, including creating a dynamic support ticket form for your product. This article will guide you through using Directus Studio to build a support ticket form that allows customers to submit their queries or issues on your product offerings.

## Before You Start

**:cloud: Instance**
Imagine a scenario where your business receives numerous customer inquiries daily. Managing these inquiries efficiently becomes importantant. In this article, you will learn how [Directus](https://directus.io) can be helpful in building a seamless support ticket handling system for your business.
<!-- ## Your Sections Here -->


- Ensure that you have setup a Directus account either on the [Directus Cloud](https://directus.cloud/) or on your local development environment using [docker](https://docs.directus.io/getting-started/quickstart.html). Watch this youtube [video](https://www.youtube.com/watch?v=ZOCfBBTMtoE) for a step-by-step guide in setting up Directus locally on your computer.

- Log into your Directus account, you'll create a specific collection for your support ticket, agent, and message. This collections will include the form for `Customer` to drop a ticket, the support collection will be for the `Support Agent` assigned to each ticket, while the message collection is for communication on the ticket. Below is the structure of the schema for each collections:

	1. Support Ticket: Ticket_Id, Name, Email, Issue Type (options: Billing, Technical Issue, General Inquiry), Description, status (Open, In Progress, Resolved), asigned_to, and Submission Date.
	2. Agents: agent_id (uuid), assigned (relationship: manyToOne, related collection: directus user)
   3. Ticket Message: message_id, date_created, ticket_message, relationship (relationship: ManyToOne, related collection: support_tickets `ticket_id`)

- Configuring roles for the support agents and the customers:
	-  `Customers` can create tickets and view the status of their own tickets, as well as add message to the ticket.

	-  `Support Agents` has access to view assigned tickets, update statuses, add message for the customer.

Setting up these roles and permissions ensures that customers have a straightforward way to submit and track their inquiries while giving support agents the tools needed to manage and resolve tickets efficiently.

## Designing the Agent Schema
The support agent schema is useful for managing a dedicated support team: below are the few fields to include in the `agent` schema:

1. Create a New Collection: Name it `agents`. This collection is where all your agents will be added. Rename the `id` to `agent-id` which will be your primary key and change the type to `uuid`.
2. Create a new field, `assigned` the field type will be `many-to-one`. The related collection will be `directus_users`. This will allow you to select your agents and add them to this collection.
3. Configure the access to this collection to be visible to only admin. Save your chnages.


## Designing the Support Ticket Schema

The major part of the support ticket system is its schema. In Directus Studio:

1. Create a New Collection: Name it `support_tickets`. This collection will store all customer queries. Creating a fresh collection, Directus suggest an initial field, `id` as the unique key of your collection. Rename the id field to `ticket-id` which will be your primary key and leave the type as auto-incremented integer.

```
:::hint

After giving your collection a name, the next page allow you to select some fields such as `status`, `created on`, `updated on`, and `updated by`.

:::
```

2. Define the Fields:
   - `customer_name` and `customer_email`: The data type is `text`. This allow you to identify the customer.
   - `issue_type` (dropdown): Define lists like 'Billing', 'Technical Issue', 'General Inquiry', etc.
   - `description` (Text Field): Detailed description of the issue.
   - `status` (Dropdown): Options like 'Open', 'In Progress', 'Resolved'.
   - `created_at` (Date Field): Automatically captures the ticket submission time.
   - `assigned_to` (ManyToOne, selecting the support agent collection): this is used by the admin to assign the ticket to a support agent. This means many tickets can be assigned to one support agent. This field will only be visible to the admin.

**:bulb: Note**:
Make sure to check the "required" option for critical fields like customer_email and issue_type to prevent incomplete submissions.

![fields and layout of a sample support ticket form](schema.png)

Ensure each field is correctly set up for its purpose, especially concerning data types, placeholder where necessary, and default values.

## Designing the Ticket_message Schema
The message schema is useful for follow-up messages between an assigned support agent and the customer. Below are the few fields to include in the `agent` schema:

1. Create a New Collection: Name it `ticket_message`. Rename the `id` to `message-id` which will be your primary key and change the type to `uuid`.
2. Add the `date_created` and `user_created` field before clicking next.
3. Create a new field, `ticket_message`, this field will be for adding messages to the ticket for communication and follow-up.
4. Create a new field, `relationship` the field type will be `many-to-one`. The related collection will be `support_tickets`, choose `ticket_id` in the dropdown. This will allow you to select which ticket the message is meant for.
5. Configure the access to this collection to be visible to only admin. Save your chnages.

Following all the above listed steps, you will have three collections by now. The collection for the ticket_submission, agents, and ticket message. All linked together.

## Implementing the communication flow
This section highlight the communication flow of your support ticket system between customers and support agents:

**For Customers:**
- Customers submit tickets through a Directus form, detailing their issue or inquiry.
- They can view the status of their submitted tickets.
- Access to a direct message with the support agent handling their case.

**For Support Agents:**
- Agents can view incoming tickets assigned to them.
- They update ticket statuses (e.g., from Open to In Progress to Resolved) and communicate directly with customers through the ticket's messaging system.

**For Admin:**
- Admin can view incoming tickets and assign the ticket to an available agent (sales, support) to act on them.
- Admin can update ticket statuses (e.g., from Open to In Progress to Resolved) and communicate directly with customers through the ticket's messaging system.

## Configuring Roles and Permissions

Directus’s flexibility in permissions allows you to control access precisely:
![Roles and permissions tab on Directus studio](access-control.png)

1. **Navigate to Roles & Permissions** in Directus Studio and create two new roles named `Customer`, `Agent`. Agents will be assigned to review and treat tickets.
2. **Set Permissions for the `Customer` Role:** Allow `read` and `create` access to the `support_tickets` collection. It's crucial to restrict `read` access to only their tickets, ensuring privacy and security.
   ![Applying rules to what user can see](rule-setting.png)
3. **Set Permissions for the `Agent` Role:** Allow `read` and `update` access to the `support_tickets` assigned to them. Also, grant a full `create`, `read`, and `update` role to the Agent on the `ticket_message` collection. 
![Applying rules to Agent, to limit what they can see](directus-agent-role.gif)

```
:::Rule Setting

Directus allows you to apply filters to permissions. To ensure users can only see their own tickets, you'll apply a filter on the Read permission. According to the image above, you'll allow the user to see only the tickets they created. You need to choose the field that links the ticket to the user. This is often a user ID or user email field in the ticket collection that references the user who submitted the ticket. This same goes to the permission set on the Agent role as well. You can read more about user fileds on the [documentation](https://docs.directus.io/reference/system/users.html).

> Test this setting with a test account before going live.

:::
```

## Inviting Customers to Use the Form

To allow customers to submit tickets, you must add them to Directus and assign them the `Customer`/`User` role based on what the configuration is:

1. Add Users in Directus Studio: Go to the Users section and invite customers via email.
   ![Invite user to your directus platform](user-directory.png)
2. **Assign Roles:** Ensure each customer user is assigned the `Customer` role to access and submit the support ticket form.

## Basic System Walkthrough

### Customer
1. The customer logs into the Directus platform and navigates to the support ticket form.
2. They fill out the form with their details and issue description and submit the ticket.
3. The customer can then go to their ticket overview page to see the status of their ticket and any messages from support agents.

### Support Agents
1. The support agent views a dashboard of all tickets, filtered by their current status.
2. Upon selecting a ticket, the agent can see all details provided by the customer and start a conversation through the messaging system within the ticket.
3. As they work on resolving the ticket, the agent updates the ticket's status and, once resolved, adds a resolution summary for the customer to view.

## Best Practices
An effective support ticket system is an evolving tool. Regular reviews of customer feedback and ticket resolution times can highlight areas for improvement. Adjusting the ticket schema, roles, and communication flow based on this analysis ensures that the system remains efficient and responsive to customer needs.

## Summary

In this article, you have been able to create a customized support ticket system that streamlines the process of managing customer inquiries and issues for your business. This guide has outlined a specific approach to setting up such a system, emphasizing a clear communication flow and practical implementation for both customers and support agents. With Directus, you have the flexibility to tailor this system to fit your business's unique needs, enhancing your ability to provide excellent customer support.

## Next Steps

To build upon the skills learnt in this article, consider the following steps:

1. Read more on the amazing offerings of Directus on their [Documentation](https://docs.directus.io/): Explore the Directus Documentation, especially sections related to API usage, advanced permission configurations, and webhooks. This will enhance your ability to customize and extend the functionality of your support ticket system.

2. Sign Up for [Directus Cloud](https://directus.cloud/): If you haven't already, sign up for Directus Cloud to experience the platform's full capabilities without the need to manage your infrastructure.

3. Setup Directus Locally: Watch this [Youtube video](https://www.youtube.com/watch?v=ZOCfBBTMtoE) that walk you through how to setup directus locally on your PC.
