---
title: 'Syncing Directus and Google Calendar with Directus Automate and Google Apps Script'
description: 'How to synchronize items in Directus with Google Calendar events using Directus Flows and Google Apps Script'
author:
---

## Introduction
In this project, we will create a two-way sync between items in Directus Collection and Google Calendar Events. So, when the user creates/updates/deletes an item in a Directus collection or Google Calendar, the corresponding entry will be altered. On the Directus side we will use Flows, and on the Google side we will use [Google Apps Script](https://developers.google.com/apps-script/overview).

&nbsp; 

## Before You Start
You will need a Directus project - check out [quickstart guide](https://docs.directus.io/getting-started/quickstart) if you don't already have one. You will also need a Google Account.

&nbsp; 

## Interactions Scheme
on the high level, it might look simple
![high level interactions scheme overview"](/copy-this-template-directory/directus_gcalendar_shapes_highlevel.svg "high level interactions scheme overview")

But the actual implementation has some hidden complexity:

- For proper syncing, we need to have some ID that is shared between Directus item and Google Calendar event.
- It's not very straightforward to organize a stream of updates from Google Calendar.

We will use the ID of Google Calendar event as the shared ID, saved as an additional field in Directus item. We can use Flows to save it, and open the ability to search by the Calendar ID within Directus.

![detailed interactions scheme overview](/copy-this-template-directory/directus_gcalendar_shapes_highlevel_23-11-20%2002.14.16.svg "detailed interactions scheme overview")

Let's describe the processes shown in the interaction scheme:

1 - Flow set to react on create/update events in our collection of items. Our project will be using a Collection named _milestones_. Another flow set to react on delete events. 

2,3 - these flows send a signal to our Google Apps Script webapp, using "Webhook / Request URL" Action.

4 - When needed (new/updated/deleted Google calendar event detected), Google Apps Script webapp sends signals to Directus Flow Webhook.

5 - as the final step of Flow in 4, this Flow creates/updates/deletes items in _milestones_ according to received parameters.

6 - Google Calendar can send push notifications when an event is added/updated/deleted, those notifications can be received directly by 
Google Apps Script webapp, but webapp cannot read request headers, so proxy Directus Flow is used to modify requests and send them to 
webapp with parameters in the body (7).

8 - Google Apps Script has additional functions (cron to renew notifications, function to stop notifications, etc).

9 - Google Apps Script webapp can create events in Google Calendar (when processes 1,2 or 1,3 are executed).

&nbsp; 

**Let's dive in.**

&nbsp; 

## Set Up Your Directus Project
Directus collection (_milestones_) has these fields:

`calendar_event_id` - type text, where Google calendar event id will be saved (automatically)

`calendar_event_start` - type timestamp, where the event start date is

`calendar_event_end` - type timestamp, where the event end date is

`name` - type text, where the event title is

`description` - type text, where event description is

:::info Timestamp Data Type

The `timestamp` type is available when the field is created in Advanced Mode. You could use `Datetime`, but since it doesn't have timezone info, you must ensure that your Directus and Google Calendar have the same timezone setting. Using `timestamp` overcomes this limitation.

:::

These fields are required, names could be changed, just ensure that you reflect these changes in the Google Spreadsheet Config

&nbsp; 

We will use [environment variables](https://docs.directus.io/self-hosted/config-options.html) in our Flows. Please add these two environment variables:

```
GCALENDARHOOKSECRET="supersecretpass"
GCALENDARHOOKURL="https://script.google.com/macros/s/xxxx/exec"
```

First is the secret word our Flows will use to validate that incoming data is from our trusted script, and the second is the URL of your published Google Apps Script, you will acquire it later so feel free to put in a placeholder value. 

In order for Flows to have access to environment variables, they need to be listed in the `FLOWS_ENV_ALLOW_LIST` environment variable. If it exists, add the two new values to it. If it does not already exist, create it:

```
FLOWS_ENV_ALLOW_LIST=GCALENDARHOOKSECRET,GCALENDARHOOKURL
```

&nbsp; 

## Create Your Flows

There are 4 Flows required for this project: 

1. Flow "Google Calendar event Proxy" - receives info about Google Calendar trigger event with info in Headers and sends it in Body to Google Apps script.

2. Flow "Send create/update event to Google Calendar" - sends data to Google Apps script about created/updated collection item.

3. Flow "Send delete event to Google Calendar" - sends data to Google Apps script about deleted collection item.

4. Flow "Process events from Google Calendar" - webhook, called from Google Apps script when an event is created / updated / deleted in Google Calendar.

&nbsp; 

### Google Calendar Event Proxy Flow

When a change is made in our calendar, a [push notification](https://developers.google.com/calendar/api/guides/push) will trigger the registered URL endpoint with data in the headers. Google Apps Script can't read headers, so this first Flow will act as a 'middleman' proxy sitting between Google Calendar and the Google Apps Script. It will receive requests, save data from the headers in the body, and then send it back to Google Apps Script. 

![whole flow](/copy-this-template-directory/directus_flow_1_full_.png "whole flow")

Create a Flow with a **Webhook** trigger. Use a POST method. 

Add a **Webhook / Request URL** operation and send a POST request to `{{$env.GCALENDARHOOKURL}}`. The actual value in the environment variable will be set after Google Apps Script is published and a URL is provisioned. Set the Request Body to: 

```js
{
  "headers": {{$trigger.headers}}
}
```

:::info Quoting

Note that `{{$trigger.headers}}` is not quoted as it will be an object.

:::

Save the Flow and take note of the Webhook URL for later.


***

&nbsp; 
 
### Send Create or Update Event to Google Calendar Flow
After we send the event information, we might receive the ID of the Google Calendar Event that was created and we must update the current Directus item with this ID. This operation is not blocking. 

![whole flow](/copy-this-template-directory/directus_flow_3_full_.png "whole flow")

***

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

Create an Operation "Webhook / Request URL" and set Method to **Post**, set Key to **request_webhook_update**, set URL to `{{$env.GCALENDARHOOKURL}}`, set Request body to:

```js
{
  "data": {{$last}},
  "action": "update",
  "pass": "{{$env.GCALENDARHOOKSECRET}}"
}

```
:::info Quoting

Note that `{{$last}}` is not quoted as it will be an object.

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

Create an Operation "Webhook / Request URL" and set Method to Post, set key to **request_webhook_create**, set URL to `{{$env.GCALENDARHOOKURL}}`, set Request body to:

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

### Send Delete Event to Google Calendar  Flow

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

Create an Operation "Webhook / Request URL", set Method to Post, set URL to `{{$env.GCALENDARHOOKURL}}`, set Request body to:

```js
{
  "data": {{$last}},
  "action": "delete",
  "pass": "{{$env.GCALENDARHOOKSECRET}}"
}

```

***

&nbsp; 

### Process events from Google Calendar  Flow

This final Flow is the entry point for the Google Apps Script to interact with Directus regardless of whether it is triggered based on a create, update, or delete operation. It will determine and execute the correct operations within your collection.


part 1

![whole flow](/copy-this-template-directory/directus_flow_4_full_1_.png "whole flow")


part 2

![whole flow](/copy-this-template-directory/directus_flow_4_full_2_.png "whole flow")

***

Create a Flow with a **Webhook** trigger. Use a POST method.

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

:::info node key

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


***

In the Reject route of the Condition operation, where you had rule "count($last): {_gte: 1}", create an Operation "Condition".

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


## Setup in Google side

[Google Apps Script](https://developers.google.com/apps-script/overview) is usually enabled by default, but if you are using Google Workspace within the organization, your Admin might disable Google Apps Script.

Google Apps Script can be created as a dedicated file (Drive → New → More → Google Apps Script).

But for this project, we will use Script within Spreadsheet. That way it is easier to use a spreadsheet for config and for logging.

For quicker setup, please use [this spreadsheet template copy](https://docs.google.com/spreadsheets/d/1mXuSVR01uueeRDTjNeSXFH2rAziWkXdrNb2Jym413Fw/copy), it has full script inside and sheet with config.

In your spreadsheet copy, open the menu Extensions → Apps Script. It will open Script Editor.

![in the top-right corner there is Deploy button](/copy-this-template-directory/GAS_04.png "in the top-right corner there is Deploy button")

Click the button `Deploy` → `New Deployment`.

When Google Apps Script is deployed as Web App, it creates a unique URL like `https://script.google.com/macros/s/xxxx/exec`.
When this URL is called with a GET request, the script function `doGet` is executed.  When this URL is called with a POST request, script function `doPost` is executed. We will use POST requests and `doPost` function.
 
 ***

![New deployment popup](/copy-this-template-directory/GAS_05.png "New deployment popup")

set type to web app. Description - anything. Execute as - Me. Who has access - anyone.
Click `Deploy`.

 ***

![popup with Authorize access button](/copy-this-template-directory/GAS_01.png "popup with Authorize access button")

 click `Authorize access`
 ***

![warning from Google about unverified app](/copy-this-template-directory/GAS_02.png "warning from Google about unverified app")

 
Choose your account and then click "Allow".

Some users (usually, users not within the organization) will see a scary-looking

"Google hasn’t verified this app
The app is requesting access to sensitive info in your Google Account. Until the developer (your email) verifies this app with Google, you shouldn't use it."

Click "Advanced"

Then click "Go to code (unsafe)"

 
 ***

![list of permissions script requires](/copy-this-template-directory/GAS_03.png "GAS 03")

click "Allow".

If all is fine, you'll see next popup with URL of your published web app.

Web app

URL

`https://script.google.com/macros/s/xxxxxxxxxxxxxxxxxxx/exec`

Copy

click `copy` to get this Web app URL (make sure that you are copying web app URL and not Library URL) and update Directus Environment Variable `GCALENDARHOOKURL` to this value


 ***

 Back to the script editor

![In the action bar at the top of the script editor there is a run button and a dropdown of function names](/copy-this-template-directory/GAS_06.png "In the action bar at the top of the script editor there is a run button and a dropdown of function names")

Run the `listCalendars` function.

The result of the function `listCalendars` will be shown. Copy the ID of the calendar that you want to track. Usually, it's the same as an account email.

&nbsp; 

### Update Spreadsheet Config

In your Spreadsheet go to the 'Config' sheet.

1. set `calendar_id` to the id of your calendar.
2. set `directus_url_proxy` to the URL of Flow "Google Calendar event Proxy"
3. set `directus_url_webhook_from_g` to the URL of Flow "Process events from Google Calendar"
4. set `pass` to the same value as in Directus Environment Variable `GCALENDARHOOKSECRET`

&nbsp; 

### One-Time Set Up Of Google Apps Script 

In the Script Editor, we must manually run a few functions once during setup.

Select `triggerResubscribeOnceWeek` from the functions dropdown and click **Run**. If the run was successful, in the `config` sheet values next to `channel_id`, `resource_id` will be filled.

Select `runManual_getSyncedEvents` from the functions dropdown and click **Run**. If the run was successful, in the `config` sheet value next to `sync_token`, `resource_id` will be filled.


> For most of the Google Workspace App, Apps Script has a specific library, like `CalendarApp` with easy-to-use functions. However, these functions don't have all the functionality available for API calls. Luckily it's possible to use Advanced Calendar Service - it's almost like calling API calls directly. Thanks to that we can subscribe to notifications and retrieve a list of new events using syncToken.

The calendar events push notifications need resubscription, its maximum time before expiration is 1 week.

***

Set Time Trigger

![script editor have Trigger button in the left pane](/copy-this-template-directory/GAS_09.png "script editor have Trigger button in the left pane")

Click `Triggers`, then `Add Trigger`

***

![Add trigger Popup](/copy-this-template-directory/GAS_10.png "Add trigger Popup")

Select function triggerResubscribeOnceWeek

Deployment - Head

Select event source - Time-driven

Select type - Week Timer

Select day of week and time - your choice

Failure notifications - Notify me immediately

Click `Save`

&nbsp; 

### Script source code - list of functions

The source code of the script is organized into several functions. In this section, I will define what each does so you can explore and expand it. 

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

&nbsp; 

#### Script update deployment

The deployed web app has versions, so if you update the `doPost` function (or something called from it) and want these changes to have an effect - a new version of the web app needs to be deployed. If you create a new version in the same deployment, the URL will stay the same. 

![Manage Deployments popup have Edit button in the top right corner](/copy-this-template-directory/GAS_07.png "Manage Deployments popup have Edit button in the top right corner")

Manage Deployments → `Edit` (pencil icon button)

![select New Version entry from versions selector](/copy-this-template-directory/GAS_08.png "select New Version entry from versions selector")

Choose `New version`, then click `Deploy`.

The URL of the web app should stay the same (if you updated the current deployment and didn't create new deployment)

***

&nbsp; 

### Summary and Next Steps

todo: summary.

Everything is set to fully automatic two-way sync. You can try it by adding items in Directus and adding events in the Calendar.

You can check logs in the log panel for each of  Directus Flows. And you can check spreadsheet `log` sheet.

This project serves as a solid base for future expansions. It effectively manages single-item operations in Directus, paving the way for integrating bulk operations. Additionally, it treats Google Calendar recurrent events as single-date occurrences, providing a foundation for enhancing recurrent event handling.
