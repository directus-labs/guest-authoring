---
title: "Building a Customer Support Ticket Form with Directus Studio"
description: "Learn how to build a customer support ticket form with Directus Studio, enhancing service efficiency and user experience in an easy steps."
author:
  name: "Ayodele Aransiola"
  avatar_file_name: "ayodele.png"
---

This article will guide you through using [Directus Studio](https://directus.io) to build a support ticket form that allows customers to submit their queries or issues on your product offerings.

## Before You Start

- A Directus project. Follow the [Quickstart guide](https://docs.directus.io/getting-started/quickstart) to create one.

Log into your Directus account to set up the different collections of your support system, which revolves around three key components:

- Support Tickets: These is the main records where customer issues are logged and tracked through various stages from opening to resolution.
- Support Agents: Support agents are assigned to tickets to manage responses and updates, ensuring each ticket is handled efficiently.
- Ticket Messages: This feature facilitates communication between customers and agents directly within the ticket, allowing for a smooth issue resolve.

Each component plays an important role in the workflow, enabling both customers and agents to interact effectively within the system.

## Creating the Agent Collection

The support agent collection is useful for managing a dedicated support team. Create an `agents` collection and add the following fields:

- `agent-id`, primary key, type: UID.
- `assigned` many-to-one to `directus_users` (id)

This collection does not require any specific access control settings as only project admins should be able to create agents.

## Creating the Ticket Collection

The support ticket collection is useful for managing the support request from customers. Create a `support_ticket` collection and add the following fields:

- `ticket-id`, primary key, type: integer (auto-incrementing integer).
- `customer_name`, type: text.
- `customer_email`, type: text.
- `issue_type`, type: dropdown, options: Billing, Technical Issue, General Inquiry.
- `status`, type: dropdown, options: Open, In Progress, Resolved.
- `date_created` type: datetime.
- `date_updated` type: datetime.
- `user_updated` many-to-one to `directus_users` (id)
- `message` type: textarea
- `assigned_to` many-to-one to `agent collection` (agent_id)

Make sure to check the "required" option for critical fields like customer_email and issue_type to prevent incomplete submissions.

![fields and layout of a sample support ticket form](schema.png)

## Creating the Ticket Message Collection

The support ticket collection is useful for managing the support request from customers. Create a `support_ticket` collection and add the following fields:

The message collection is useful for follow-up messages between an assigned support agent and the customer. Create a `ticket_message` collection and add the following fields:

- `message-id`, primary key, type: UID.
- `date_created` type: datetime.
- `date_updated` type: datetime.
- `user_created` many-to-one to `directus_users` (id)
- `user_updated` many-to-one to `directus_users` (id)
- `message` type: textarea
- `relationship` many-to-one to `support_tickets` (ticket_id)

> Following all the above listed steps, you will have three collections by now. The collection for the ticket_submission, agents, and ticket message. All linked together.

## How It Works

### For Customers

- Customers submit tickets by creating a new item in the Support Tickets collection, detailing their issue or inquiry.
- They can view the status of their submitted tickets.
- They can also access a direct message with the support agent handling their case.

### For Support Agents

- Agents can view incoming tickets assigned to them.
- They update ticket statuses (e.g., from Open to In Progress or Resolved) and communicate directly with customers through the ticket's messaging system.

### For Admins

- Admins can view incoming tickets and assign the ticket to an available agent (sales, support) to act on them.
- Admins can update ticket statuses and communicate directly with customers through the ticket's messaging system.

## Configuring Roles and Permissions

Directus’s flexibility in permissions allows you to control access precisely:
![Roles and permissions tab on Directus studio](access-control.png)

1. **Navigate to Roles & Permissions** in Directus Studio and create two new roles named `Customer`, `Agent`. Agents will be assigned to review and treat tickets.
2. **Set Permissions for the `Customer` Role:** Allow `read` and `create` access to the `support_tickets` collection. It's crucial to restrict `read` access to only their tickets, ensuring privacy and security.
   ![Applying rules to what user can see](rule-setting.png)
3. **Set Permissions for the `Agent` Role:** Allow `read` and `update` access to the `support_tickets` assigned to them. Also, grant a full `create`, `read`, and `update` role to the Agent on the `ticket_message` collection.

![Applying rules to Agent, to limit what they can see](directus-agent-role.gif)

:::info Using Filters In Permissions

Directus allows you to apply filters to permissions. To ensure users can only see their own tickets, you'll apply a filter on the Read permission. According to the image above, you'll allow the user to see only the tickets they created.

:::


## Inviting Customers to Use the Form

To allow customers to submit tickets, you must add them to Directus and assign them the `Customer`/`User` role based on what the configuration is:

1. Add Users in Directus Studio: Go to the Users section and invite customers via email.
2. Ensure each customer user is assigned the `Customer` role to access and submit the support ticket form.

## Basic System Walkthrough

### Admins

1. The admin logs into the Directus platform and navigates to the available support tickets.
2. Admin can assign the ticket to an agent or resolve the ticket's issue.
3. The admin can also add a message to an open ticket as well close the ticket if it is resolved.

![Admin dashboard walkthrough](adminCollection.gif)

### Customers

1. The customer logs into the Directus platform and navigates to the support ticket form.
2. They fill out the form with their details and issue description and submit the ticket.
3. The customer can then go to their ticket overview page to see the status of their ticket and any messages from support agents.

![user dashboard walkthrough](userAccount.gif)

Of course, the customer contact details could be replaced by the `user_created` optional collection field.

### Support Agents

1. The support agent views a dashboard of all tickets, filtered by their current status.
2. Upon selecting a ticket, the agent can see all details provided by the customer and start a conversation through the messaging system within the ticket.
3. As they work on resolving the ticket, the agent updates the ticket's status and, once resolved, adds a resolution summary for the customer to view.

![Support Agent dashboard walkthrough](agentCollection.gif)

## Setting up Notifications for Updates

[Flows](https://docs.directus.io/app/flows.html) in Directus are used to automate actions within and outside of Directus itself, such as sending notifications. When configured, Directus will send the request based on your configuration.


1. To setup a flow, navigate to to the `Settings` menu.
2. In the settings, select the flow option.
3. Click on `Create Flow`.
4. Give your flow a name, set the status (active or inactive), add a description, and select `track activity & log` if you want the flow to track acitivity.
![How to create a flow](createFlow.png)
5. Click `Trigger Setup`
6. Select `Event hook`.
5. Type, Action (non blocking).
6. Scope, select `items.create` & `items.update`
6. Collections, select `Support Ticket` and `Ticket Message`.
7. Click the check mark at the top right
![Applying rules to what user can see](rule-setting.png)
8. Click the `+` sign in the flow image view
![Set up the flow](flowView.png)
9. Add a `name` the `key`
10. Choose `Webhook/Request URL`
11. Method should be set to `POST`.
12. Enter the URL where Directus should send the webhook POST request
13. Compelete this action by clicking the check mark at the top right.
![Webhook Operation](webhook.png)

## Best Practices

An effective support ticket system is an evolving tool. Regular reviews of customer feedback and ticket resolution times can highlight areas for improvement. Adjusting the ticket schema, roles, and communication flow based on this analysis ensures that the system remains efficient and responsive to customer needs.

## Summary

In this article, you have been able to create a customized support ticket system that streamlines the process of managing customer inquiries and issues for your business. This guide has outlined a specific approach to setting up such a system, emphasizing a clear communication flow and practical implementation for both customers and support agents. With Directus, you have the flexibility to tailor this system to fit your business's unique needs, enhancing your ability to provide excellent customer support.