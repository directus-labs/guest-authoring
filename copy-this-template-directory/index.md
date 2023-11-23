---
title: 'Syncing Directus and Google Calendar with Directus Automate and Google Apps Script'
description: 'How to synchronize items in Directus with Google Calendar events using Directus Flows and Google Apps Script'
author:
---

In this project, we will create a two-way sync between items in Directus Collection and Google Calendar Events. So, when the user creates/updates/deletes an item in a Directus collection or Google Calendar, the corresponding entry will be altered. On the Directus side we will use Flows, and on the Google side we will use [Google Apps Script](https://developers.google.com/apps-script/overview).

&nbsp;

## Before You Start

You will need a Directus project - check out [quickstart guide](https://docs.directus.io/getting-started/quickstart) if you don't already have one. You will also need a Google Account.

## Interactions Scheme

![The system overview shows the interactions between Directus and Google Services. In Directus there is a collection, a Flow processing events from the collection, and a flow processing events from Google Apps Script. On the Google side, there is a calendar, spreadsheet, Apps Script, and a Webhook.](/copy-this-template-directory/directus_gcalendar_shapes_highlevel.svg "high level interactions scheme overview")

While the project may feel simple at first, the actual implementation has some hidden complexity:

- For proper syncing, we need to have some ID that is shared between Directus item and Google Calendar event.
- It's not very easy to organize a stream of updates from Google Calendar.

We will use the ID of Google Calendar event as the shared ID, saved as an additional field in the Directus data model. We can use Flows to save it, and open the ability to search by the Calendar ID within Directus.

![detailed interactions scheme overview](/copy-this-template-directory/directus_gcalendar_shapes_highlevel_23-11-20%2002.14.16.svg "detailed interactions scheme overview")

Let's describe the processes shown in the interaction scheme:

- 1 - A Flow is triggered when items in our collection are created/updated. Our project will be using a Collection named _milestones_. A separate flow is triggered when items are deleted.
- 2 and 3 - These flows send a signal to our Google Apps Script webapp, using "Webhook / Request URL" Action.
- 4 - When needed (new/updated/deleted Google calendar event detected), Google Apps Script webapp sends signals to Directus Flow Webhook.
- 5 - As the final step of Flow in 4, this Flow creates/updates/deletes items in _milestones_ according to received parameters.
- 6 and 7 - Google Calendar can send push notifications when an event is added/updated/deleted, those notifications can be received directly by the Google Apps Script webapp, but cannot read request headers containing required data. A proxy Directus Flow is used to modify requests and send them to
webapp with parameters in the body.
- 8 - Google Apps Script has additional functions (cron to renew notifications, function to stop notifications, etc).
- 9 - Google Apps Script webapp can create events in Google Calendar.

## Set Up Your Directus Project

Directus collection (_milestones_) has these fields:

- `calendar_event_id` - type text, where Google calendar event id will be saved (automatically)
- `calendar_event_start` - type timestamp, where the event start date is
- `calendar_event_end` - type timestamp, where the event end date is
- `name` - type text, where the event title is
- `description` - type text, where event description is

:::info Timestamp Data Type

The `timestamp` type is available when the field is created in Advanced Mode. You could use `Datetime`, but since it doesn't have timezone info, you must ensure that your Directus and Google Calendar have the same timezone setting. Using `timestamp` overcomes this limitation.

:::

These fields are required, names could be changed, just ensure that you reflect these changes in the Google Spreadsheet Config introduced later.

We will use [environment variables](https://docs.directus.io/self-hosted/config-options.html) in our Flows. Please add these two environment variables:

```
GCALENDARHOOKSECRET="supersecretpass"
GCALENDARHOOKURL="https://script.google.com/macros/s/xxxx/exec"
```

`GCALENDARHOOKSECRET` is a secret key our Flows will use to validate that incoming data is from our trusted script, and `GCALENDARHOOKURL` is the URL of your published Google Apps Script, you will acquire it later so feel free to put in a placeholder value.

In order for Flows to have access to environment variables, they need to be listed in the `FLOWS_ENV_ALLOW_LIST` environment variable. If it exists, add the two new values to it. If it does not already exist, create it:

```
FLOWS_ENV_ALLOW_LIST=GCALENDARHOOKSECRET,GCALENDARHOOKURL
```

## Create Your Flows

There are 4 Flows required for this project:

1. Flow "Google Calendar Event Proxy" - receives info about Google Calendar trigger event with info in Headers and sends it in Body to Google Apps script.
2. Flow "Send Create/Update Event to Google Calendar" - sends data to Google Apps script about created/updated collection item.
3. Flow "Send Delete Event to Google Calendar" - sends data to Google Apps script about deleted collection item.
4. Flow "Process Events from Google Calendar" - Webhook, called from Google Apps script when an event is created / updated / deleted in Google Calendar.

### Google Calendar Event Proxy Flow

When a change is made in our calendar, a [push notification](https://developers.google.com/calendar/api/guides/push) will trigger the registered URL endpoint with data in the headers. Google Apps Script can't read headers, so this first Flow will act as a 'middleman' proxy sitting between Google Calendar and the Google Apps Script. It will receive requests, save data from the headers in the body, and then send it back to Google Apps Script.

![whole flow](/copy-this-template-directory/directus_flow_1_full_.png "whole flow")

Create a Flow with a **Webhook** trigger. Use a POST method.

Add a **Webhook / Request URL** operation and send a POST request to <v-span pre>`{{$env.GCALENDARHOOKURL}}`</v-span>. The actual value in the environment variable will be set after Google Apps Script is published and a URL is provisioned. Set the Request Body to:

```js
{
  "headers": {{$trigger.headers}}
}
```

:::info Quoting

Note that <v-span pre>`{{$trigger.headers}}`</v-span> is not quoted as it will be an object.

:::

Save the Flow and take note of the Webhook URL for later.

### Send Create or Update Event to Google Calendar Flow

After we send the event information, we might receive the ID of the Google Calendar Event that was created and we must update the current Directus item with this ID. This operation is not blocking.

![whole flow](/copy-this-template-directory/directus_flow_3_full_.png "whole flow")

Create a Flow with a **Event Hook** trigger. Set Type to "Action (Non-Blocking)", scope to "items.create, items.update" and  Collections to `milestones` and set Response to "data of last operation".

Create an Operation "Condition" and set Rules to:

```js
{
    "$trigger": {
        "event": {
            "_ends_with": ".items.update"
        }
    }
}
```

For the "Resolve" route, create an Operation "Read Data" and set Collections to `milestones`, set Key to **item_read_updated**, set IDs (edit raw value) to:

```js
[
    "{{$trigger.keys[0]}}"
]
```

Create an Operation "Webhook / Request URL" and set Method to **Post**, set Key to **request_webhook_update**, set the URL to <v-span pre>`{{$env.GCALENDARHOOKURL}}`</v-span>, set the Request Body to:

```js
{
  "data": {{$last}},
  "action": "update",
  "pass": "{{$env.GCALENDARHOOKSECRET}}"
}

```

:::info Quoting

Note that <v-span pre>`{{$last}}`</v-span> is not quoted as it will be an object.

:::

Create an Operation "Condition" and set Rules to:

```js
{
    "$last": {
        "data": {
            "res": {
                "_eq": "need_update_calendar_event_id"
            }
        }
    }
}
```

Create an Operation "Update Data" and set Collection to a `milestones`, set IDs (edit raw value) to:

```js
[
    "{{$trigger.keys[0]}}"
]
```

and set Payload to:

```js
{
    "calendar_event_id": "{{request_webhook_update.data.id}}"
}
```
:::info node key

Make sure that you are using the same key (here it's "request_webhook_update") as you set in "Webhook / Request URL"

:::

In the Reject route of the very first Condition create an Operation "Condition" and set Rules to:

```js
{
    "$trigger": {
        "event": {
            "_ends_with": ".items.create"
        }
    }
}
```

Create an Operation "Run Script" and set Key to **payload_transformed**, set Code to:

```js
module.exports = async function(data) {
	let out = data.$trigger.payload;
	out["id"] = data.$trigger.key;
	return out;
}
```

Create an Operation "Webhook / Request URL" and set Method to Post, set key to **request_webhook_create**, set the URL to <v-span pre>`{{$env.GCALENDARHOOKURL}}`</span>, set the Request body to:

```js
{
  "data": {{$last}},
  "action": "create",
  "pass": "{{$env.GCALENDARHOOKSECRET}}"
}
```

Create an Operation "Condition" and set Rules to:

```js
{
    "$last": {
        "data": {
            "res": {
                "_eq": "need_update_calendar_event_id"
            }
        }
    }
}
```

Create an Operation "Update Data" and set Collection to `milestones`, set IDs (edit raw value) to:

```js
[
    "{{$trigger.key}}"
]
```

set Payload to:

```js
{
    "calendar_event_id": "{{request_webhook_create.data.id}}"
}
```

:::info node key

Make sure that you are using the same key (here it's "request_webhook_create") as you set in related "Webhook / Request URL"

:::

***

&nbsp;

### Send Delete Event to Google Calendar Flow

Processing of the Delete event is a bit different from processing of Create / Update (next flow). It should be set to blocking, cause we need to "intercept" the delete command and read item data - we need to know the id of the Google Calendar event, so we can send it to Published Google Apps Script.

![whole flow](/copy-this-template-directory/directus_flow_2_full_.png "whole flow")

***

Create a Flow with a **Event Hook** trigger. Set Type to "Filter(Blocking)", Scope to "items.delete", set Collections to `milestones` and set Response to "data of last operation".

Create an Operation "Read data", set Collections to `milestones` and set IDs (edit raw value) to:

```js
[
    "{{$trigger.payload[0]}}"
]
```

Create an Operation "Webhook / Request URL", set Method to Post, set the URL to <v-span pre>`{{$env.GCALENDARHOOKURL}}`</v-span>, set the Request body to:

```js
{
  "data": {{$last}},
  "action": "delete",
  "pass": "{{$env.GCALENDARHOOKSECRET}}"
}

```

***

&nbsp;

### Process Events from Google Calendar Flow

This final Flow is the entry point for the Google Apps Script to interact with Directus regardless of whether it is triggered based on a create, update, or delete operation. It will determine and execute the correct operations within your collection.

![](/copy-this-template-directory/directus_flow_4_full_1_.png "whole flow")

![](/copy-this-template-directory/directus_flow_4_full_2_.png "whole flow")

***

Create a Flow with a **Webhook** trigger. Use the POST method.

Create an Operation "Condition" and set Rules to:

```js
{
    "$trigger": {
        "body": {
            "pass": {
                "_eq": "{{$env.GCALENDARHOOKSECRET}}"
            }
        }
    }
}
```

This will act as a gatekeeper - check that the password from the incoming parameter is the same as saved in the Environment Variable, since we don't want some random wandering bot to trigger real actions.

The next step is to determine from the incoming parameter if a Directus item needs to be created. Create an Operation "Condition" and set Rules to:

```js
{
    "$trigger": {
        "body": {
            "action": {
                "_eq": "create"
            }
        }
    }
}
```

Create an Operation "Create Data" and set Collection `milestones`, set Permissions to "full access", set Payload to:

```js
{
    "calendar_event_id": "{{$trigger.body.data.calendar_event_id}}",
    "calendar_event_start": "{{$trigger.body.data.calendar_event_start}}",
    "calendar_event_end": "{{$trigger.body.data.calendar_event_end}}",
    "name": "{{$trigger.body.data.name}}",
    "description": "{{$trigger.body.data.description}}"
}
```

In the Reject route of the last condition create an Operation "Read Data". This node will find Directus Item with a certain `calendar_event_id`.

Set Key to **item_read**, set Permissions to "Full Access", set Collection to `milestones`, set Query to:

```js
{
    "filter": {
        "calendar_event_id": {
            "_eq": "{{$trigger.body.data.calendar_event_id}}"
        }
    }
}
```

The next node will check if such an item is found (then proceed to actions delete/update, otherwise - create a new item (for update action)). Create an Operation "Condition", and set Rules to:

```js
{
    "count($last)": {
        "_gte": 1
    }
}
```

The next node will determine from the incoming parameter if a Directus item needs to be deleted. Create an Operation "Condition", and set Rules to:

```js
{
    "$trigger": {
        "body": {
            "action": {
                "_eq": "delete"
            }
        }
    }
}
```

Create an Operation "Delete Data" and set Permissions to "Full Access", set Collection to `milestones`, and set IDs (edit raw value) to:

```js
[
    "{{item_read[0].id}}"
]
```

:::info Operation Key

Make sure that you are using the same key (here it's "item_read") as you set in the first Operation "Read Data"

:::

In the Reject route of the last Condition operation create an Operation "Update Data", set permissions to "Full Access", set Collection to `milestones`, and set IDs (edit raw value) to:

```js
[
    "{{item_read[0].id}}"
]
```

and set Payload to:

```js
{
    "calendar_event_id": "{{$trigger.body.data.calendar_event_id}}",
    "calendar_event_start": "{{$trigger.body.data.calendar_event_start}}",
    "calendar_event_end": "{{$trigger.body.data.calendar_event_end}}",
    "name": "{{$trigger.body.data.name}}",
    "description": "{{$trigger.body.data.description}}"
}
```

In the Reject route of the Condition operation, where you had rule "count($last): \{_gte: 1\}", create an Operation "Condition".

The operation is in the branch "item not found". So, if the action was to Update an item, then creating an item is required (in the next node).
Set Rules to:

```js
{
    "$trigger": {
        "body": {
            "action": {
                "_eq": "update"
            }
        }
    }
}
```

Create an Operation "Create Data" and set Permissions to "full access", set Collection to `milestones`, and set Payload to:

```js
{
    "calendar_event_id": "{{$trigger.body.data.calendar_event_id}}",
    "calendar_event_start": "{{$trigger.body.data.calendar_event_start}}",
    "calendar_event_end": "{{$trigger.body.data.calendar_event_end}}",
    "name": "{{$trigger.body.data.name}}",
    "description": "{{$trigger.body.data.description}}"
}
```

After you save this Flow, copy the resulting webhook URL somewhere.

***

&nbsp;


## Set Up Google Apps Script

[Google Apps Script](https://developers.google.com/apps-script/overview) is usually enabled by default, but if you are using Google Workspace within the organization, your Admin might disable Google Apps Script.

Google Apps Script can be created as a dedicated file, but for this project, we will use a Script within a Spreadsheet. The spreadsheet can be used to store configuration and for logging.

For quicker setup, please use [this spreadsheet template copy](https://docs.google.com/spreadsheets/d/1gdHqUyzBoLJv86KgvAx42y0cL0JHot5p9iWIezJcGHQ/copy), which has the full script and sheet with configuration settings.

In your spreadsheet copy, open the menu Extensions → Apps Script. It will open Script Editor.

![in the top-right corner there is Deploy button](/copy-this-template-directory/GAS_04.png "in the top-right corner there is Deploy button")

Click the button `Deploy` → `New Deployment`.

When Google Apps Script is deployed as Web App, it creates a unique URL like `https://script.google.com/macros/s/xxxx/exec`.

When this URL is called with a GET request, the script function `doGet` is executed.  When this URL is called with a POST request, script function `doPost` is executed. We will use POST requests and `doPost` function.

Set type to a Web app, write any comment in the Description, set "Execute as" to Me, set "Who has access" to anyone, and click `Deploy`, authorizing access to your script and going to code beyond the warning which shows during development.

If all is fine, you'll see next popup with URL of your published web app in the format `https://script.google.com/macros/s/xxxxxxxxxxxxxxxxxxx/exec`.

Copy the web app URL (make sure that you are copying web app URL and not Library URL) and update Directus Environment Variable `GCALENDARHOOKURL` to this value

![In the action bar at the top of the script editor there is a run button and a dropdown of function names](/copy-this-template-directory/GAS_06.png "In the action bar at the top of the script editor there is a run button and a dropdown of function names")

Back to the script editor, run the `listCalendars` function and the result of the function `listCalendars` will be shown. Copy the ID of the calendar that you want to track. Usually, it's the same as an account email.

### Update Spreadsheet Configuration

In your Spreadsheet go to the 'Config' sheet.

1. set `calendar_id` to the id of your calendar.
2. set `directus_url_proxy` to the URL of Flow "Google Calendar event Proxy"
3. set `directus_url_webhook_from_g` to the URL of Flow "Process events from Google Calendar"
4. set `pass` to the same value as in Directus Environment Variable `GCALENDARHOOKSECRET`

### One-Time Set Up Of Google Apps Script

In the Script Editor, we must manually run a few functions once during setup.

Select `triggerResubscribeOnceWeek` from the functions dropdown and click **Run**. If the run was successful, in the `config` sheet values next to `channel_id`, `resource_id` will be filled.

Select `runManual_getSyncedEvents` from the functions dropdown and click **Run**. If the run was successful, in the `config` sheet value next to `sync_token`, `resource_id` will be filled.

:::info Calendar Advanced service

For most of the Google Workspace App, Apps Script has a specific library, like `CalendarApp` with easy-to-use functions. However, these functions don't have all the functionality available for API calls. Luckily it's possible to use Advanced Calendar Service - it's almost like calling API calls directly. Thanks to that we can subscribe to notifications and retrieve a list of new events using syncToken.

:::

![script editor have Trigger button in the left pane](/copy-this-template-directory/GAS_09.png "script editor have Trigger button in the left pane")

We must set up a weekly Trigger to resubscribe. Click `Triggers`, then `Add Trigger`

***

![Add trigger Popup](/copy-this-template-directory/GAS_10.png "Add trigger Popup")

- Select function trigger: `ResubscribeOnceWeek`
- Deployment - Head
- Select event source - Time-driven
- Select type - Week Timer
- Select day of week and time - your choice
- Failure notifications - Notify me immediately

### Script Source Code Explanation

The source code of the script is organized into several functions:

- `writelog` - utility function to write to the spreadsheet log entry
- `listCalendars` - function for manual launch - to list available for user calendars info (IDs, description)
- `triggerResubscribeOnceWeek` - function for once-a-week time trigger. Will resubscribe to calendar push notifications (1 week is the maximum allowed time)
- `runManual_processSyncedEvents` - function for manual launch - get events stream and process it (send to Directus)
- `runManual_getSyncedEvents` - function for manual launch - set initial sync token, nothing sent to Directus
- `getConfig` - utility function for retrieving Config set in spreadsheet
- `updateConfig` - utility function for updating certain Config in the spreadsheet
- `callDirectusWebhook` - actual data sent to Directus
- `processSyncedEvents` - get events stream and process it (send to Directus)
- `doPost` - main function, special name doPost means that when the web app is published, POST requests are processed here
- `callCalendarEventsWatch_stop` - utility function for stopping push notifications
- `callCalendarEventsWatch` - utility function for registering URL for receiving push notifications
- `getSyncedEvents` - utility function for getting a list of new / updated / deleted events in Google Calendar, using special "syncToken"

#### Updating the Script

The deployed web app has versions, so if you update the `doPost` function (or something called from it) and want these changes to have an effect - a new version of the web app needs to be deployed. If you create a new version in the same deployment, the URL will stay the same.

![Manage Deployments popup have Edit button in the top right corner](/copy-this-template-directory/GAS_07.png "Manage Deployments popup have Edit button in the top right corner")

Manage Deployments → `Edit` (pencil icon button)

![select New Version entry from versions selector](/copy-this-template-directory/GAS_08.png "select New Version entry from versions selector")

Choose `New version`, then click `Deploy`.

### Summary and Next Steps

We created Automation Flows in Directus to send updates from the collection of items to Google Apps Script, which will create events in the Calendar. And vice versa, this Script will check if a new event is added or updated in the Calendar and call Directus automation Flow to reflect these changes in Directus items.

Everything is set to successfully keep the two-way sync active. You can check logs in the log panel for each of  Directus Flows. And you can check spreadsheet `log` sheet.

This project serves as a solid base for future expansions. It effectively manages single-item operations in Directus, paving the way for integrating bulk operations. Additionally, it treats Google Calendar recurrent events as single-date occurrences, providing a foundation for enhancing recurrent event handling.
