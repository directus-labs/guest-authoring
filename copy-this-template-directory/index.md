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

1 - Flow set to react on create/update events in our _collection1_. Another flow set to react on delete events. 

2,3 - these flows send a signal to our Google Apps Script webapp, using "Webhook / Request URL" Action.

4 - When needed (new/updated/deleted Google calendar event detected), Google Apps Script webapp sends signals to Directus Flow Webhook.

5 - as the final step of Flow in 4, this Flow creates/updates/deletes items in collection1 according to received parameters.

6 - Google Calendar can send push notifications when an event is added/updated/deleted, those notifications can be received directly by 
Google Apps Script webapp, but webapp cannot read request headers, so proxy Directus Flow is used to modify requests and send them to 
webapp with parameters in the body (7).

8 - Google Apps Script has additional functions (cron to renew notifications, function to stop notifications, etc).

9 - Google Apps Script webapp can create events in Google Calendar (when processes 1,2 or 1,3 are executed).

&nbsp; 

**Let's dive in.**

&nbsp; 

## Set Up Your Directus Project
Directus collection (_collection1_ in this sample project) has these fields:

`calendar_event_id` - type text, where Google calendar event id will be saved (automatically)

`calendar_event_start` - type timestamp, where the event start date is

`calendar_event_end` - type timestamp, where the event end date is

`name` - type text, where the event title is

`description` - type text, where event description is

> `timestamp` type is available when the field is created in Advanced Mode. Could be Datetime type used instead? It might work, but since it doesn't have timezone info, you must ensure that your Directus and Google Calendar have the same timezone setting.

These fields are required, names could be changed, just ensure that you reflect these changes in the Google Spreadsheet Config

&nbsp; 

In the Flows, we will use [environment variables](https://docs.directus.io/self-hosted/config-options.html). Please add these two environment variables:

`GCALENDARHOOKSECRET=supersecretpass`

`GCALENDARHOOKURL=https://script.google.com/macros/s/xxxx/exec`

First is the secret word our Flows will use to check that incoming data is from our trusted script, and the second is the actual URL of the published google apps script, you will acquire it later. In order for Flows to have access to variables, they need to be listed in another variable:

`FLOWS_ENV_ALLOW_LIST=GCALENDARHOOKSECRET,GCALENDARHOOKURL`

&nbsp; 

**Now a Flows.**

There are 4 Flows. For each Flow I'll show an overview picture and then each node screen with config also as text, so you can copy-paste it.

&nbsp; 

### Flow "Google Calendar event Proxy"

As I mentioned - it's not very straightforward to organize a stream of updates from Google Calendar with Google Apps Script.
We will use [push notifications](https://developers.google.com/calendar/api/guides/push) that will call our published Google Apps Script when there are changes in the calendar. The catch though, these push notifications provide data within Header but for incoming requests, Google Apps Script can't read headers, only the body. That's why we will use Directus Flow as a simple proxy, it's webhook URL will be registered as the address for push notifications, it will receive data, save the header as a body, and send it to our published Google Apps Script.

![whole flow](/copy-this-template-directory/directus_flow_1_full_.png "whole flow")

***

- Trigger node - webhook:

![trigger node](/copy-this-template-directory/directus_flow_1_01_.png "trigger node")


***

- Node 2 - "Webhook / Request URL"

![request node](/copy-this-template-directory/directus_flow_1_02_.png "request node")

URL is `{{$env.GCALENDARHOOKURL}}` - the actual value in the environment variable will be set after Google Apps Script is published.

Request body:
```js
{
  "headers": {{$trigger.headers}}
}
```

> note that {{$trigger.headers}} is not quoted!

After you save this Flow, copy the resulting webhook URL somewhere (the URL can be copied from the trigger node, when in view mode, not in edit mode).

***

&nbsp; 

### Flow "Send delete event to Google Calendar"

Processing of the Delete event is a bit different from processing of Create / Update (next flow). It should be set to blocking, cause we need to "intercept" the delete command and read item data - we need to know the id of the Google Calendar event, so we can send it to Published Google Apps Script.

![whole flow](/copy-this-template-directory/directus_flow_2_full_.png "whole flow")

***

- Trigger node - Event Hook

![trigger node](/copy-this-template-directory/directus_flow_2_01_.png "trigger node")

make sure that you have the same config:

Type - Filter(Blocking)

Scope - items.delete

Collections - a collection of your choice

Response - data of last operation


***

- Node 2 - "Read data"

![node 02](/copy-this-template-directory/directus_flow_2_02_.png "node 02")

IDs edit raw value to:
```js
[
    "{{$trigger.payload[0]}}"
]
```
Query is empty


***

- Node 3 - "Webhook / Request URL"

![node 03](/copy-this-template-directory/directus_flow_2_03_.png "node 03")

URL is `{{$env.GCALENDARHOOKURL}}` - the actual value in the environment variable will be set after Google Apps Script is published.

Method is Post

Request body:
```js
{
  "data": {{$last}},
  "action": "delete",
  "pass": "{{$env.GCALENDARHOOKSECRET}}"
}

```

> note that {{$last}} is not quoted!

***

&nbsp; 
 
### Flow "Send create/update event to Google Calendar"
Processing of Create / Update is more complicated than Delete, cause after we send this event info, we might receive the ID of the Google Calendar Event that was created and we must update the current Directus item with this ID. This operation is not blocking. 

![whole flow](/copy-this-template-directory/directus_flow_3_full_.png "whole flow")

***

- Trigger Node - Event Hook

![trigger node](/copy-this-template-directory/directus_flow_3_01.png "trigger node")

parameters:

Type - Action (Non-Blocking)

Scope - items.create, items.update

Collections - a collection of your choice


***

- Node 2 - "Condition"
  
![node 02](/copy-this-template-directory/directus_flow_3_02.png "node 02")

Rules:

```js
{
    "$trigger": {
        "event": {
            "_ends_with": ".items.update"
        }
    }
}
```


***

- Node 3 - "Read Data"

![node 03](/copy-this-template-directory/directus_flow_3_03.png "node 03")

Key - **item_read_updated**

IDs (edit raw value):
```js
[
    "{{$trigger.keys[0]}}"
]
```
Query is empty

***

- Node 4 - "Webhook / Request URL"

![node 04](/copy-this-template-directory/directus_flow_3_04.png "node 04")

key - **request_webhook_update**

URL is `{{$env.GCALENDARHOOKURL}}` - the actual value in the environment variable will be set after Google Apps Script is published.

Method is Post

Request body:
```js
{
  "data": {{$last}},
  "action": "update",
  "pass": "{{$env.GCALENDARHOOKSECRET}}"
}

```

> note that {{$last}} is not quoted!

***

- Node 5 - "Condition"
  
![node 05](/copy-this-template-directory/directus_flow_3_05.png "node 05")

Rules:

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



***

- Node 6 - "Update Data"

![node 06](/copy-this-template-directory/directus_flow_3_06.png "node 06")

Collection - a collection of your choice

IDs (edit raw value):
```js
[
    "{{$trigger.keys[0]}}"
]
```

Payload:
```js
{
    "calendar_event_id": "{{request_webhook_update.data.id}}"
}
```

> Make sure that you are using the same key (here it's "request_webhook_update") as you set in node 4

Query is empty

***

- Node 7 - "Condition"

![node 07](/copy-this-template-directory/directus_flow_3_07.png "node 07")

Rules:

```js
{
    "$trigger": {
        "event": {
            "_ends_with": ".items.create"
        }
    }
}
```


***

- Node 8 - "Run Script"

![node 08](/copy-this-template-directory/directus_flow_3_08.png "node 08")

Key - **"payload_transformed"**

Code
```js
module.exports = async function(data) {
	let out = data.$trigger.payload;
	out["id"] = data.$trigger.key;
	return out;
}
```


***

- Node 9 - "Webhook / Request URL"

![node 09](/copy-this-template-directory/directus_flow_3_09.png "node 09")

key - **request_webhook_create**

URL is `{{$env.GCALENDARHOOKURL}}` - the actual value in the environment variable will be set after Google Apps Script is published.

Method is Post

Request body:
```js
{
  "data": {{$last}},
  "action": "create",
  "pass": "{{$env.GCALENDARHOOKSECRET}}"
}
```

> note that {{$last}} is not quoted!
  

***

- Node 10 - "Condition"
  
![node 10](/copy-this-template-directory/directus_flow_3_10.png "node 10")

Rules:

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



***

- Node 11 - "Update Data"

![node 11](/copy-this-template-directory/directus_flow_3_11.png "node 11")

Collection - a collection of your choice

IDs (edit raw value):
```js
[
    "{{$trigger.key}}"
]
```

Payload:
```js
{
    "calendar_event_id": "{{request_webhook_create.data.id}}"
}
```

> Make sure that you are using the same key (here it's "request_webhook_create") as you set in node 9

Query is empty

***

&nbsp; 

### Flow "Process events from Google Calendar"

This is a webhook, called from Google Apps script when an event is created / updated / deleted in Google Calendar.

Collection1 item will be Created / Updated / Deleted from incoming hook parameters.

This flow is a bit wide, so the screenshot is split in two.


part 1

![whole flow](/copy-this-template-directory/directus_flow_4_full_1_.png "whole flow")


part 2

![whole flow](/copy-this-template-directory/directus_flow_4_full_2_.png "whole flow")

***

- Trigger Node - Webhook

![trigger node](/copy-this-template-directory/directus_flow_4_01.png "trigger node")

config:

Method - Post

Response Body - Data of Last Operation


***

- Node 2 - "Condition"
  
simple gatekeeper - check that the password from the incoming parameter is the same as saved in the Environment Variable

![node 02](/copy-this-template-directory/directus_flow_4_02.png "node 02")

Rules:

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


***

- Node 3 - "Condition"

determine from the incoming parameter, if a Directus item needs to be created

![node 03](/copy-this-template-directory/directus_flow_4_03.png "node 03")

Rules:

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


***

- Node 4 - "Create Data"

![node 04](/copy-this-template-directory/directus_flow_4_04.png "node 04")

Collection - a collection of your choice

Permissions - full access

Payload:
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

- Node 5 - "Read Data"

find Directus Item with certain `calendar_event_id`

![node 05](/copy-this-template-directory/directus_flow_4_05.png "node 05")

Key - **item_read**

Permisssions - Full Access 

Collection - a collection of your choice

IDs is empty

Query:
```js
{
    "filter": {
        "calendar_event_id": {
            "_eq": "{{$trigger.body.data.calendar_event_id}}"
        }
    }
}
```


***

- Node 6 - "Condition"

if such an item is found (then proceed to actions delete/update, otherwise - create a new item (for update action))

![node 06](/copy-this-template-directory/directus_flow_4_06.png "node 06")

Rules:
```js
{
    "count($last)": {
        "_gte": 1
    }
}
```


***

- Node 7 - "Condition"

determine from the incoming parameter, if a Directus item needs to be deleted

![node 07](/copy-this-template-directory/directus_flow_4_07.png "node 07")

Rules:
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


***

- Node 8 - "Delete Data"

![node 08](/copy-this-template-directory/directus_flow_4_08.png "node 08")

Permisssions - Full Access 

Collection - a collection of your choice

IDs (edit raw value):
```js
[
    "{{item_read[0].id}}"
]
```
> Make sure that you are using the same key (here it's "item_read") as you set in node 5

Query is empty


***

- Node 9 - "Update Data"

![node 09](/copy-this-template-directory/directus_flow_4_09.png "node 09")

Collection - a collection of your choice

Permissions - Full Access

IDs (edit raw value):
```js
[
    "{{item_read[0].id}}"
]
```

Payload:
```js
{
    "calendar_event_id": "{{$trigger.body.data.calendar_event_id}}",
    "calendar_event_start": "{{$trigger.body.data.calendar_event_start}}",
    "calendar_event_end": "{{$trigger.body.data.calendar_event_end}}",
    "name": "{{$trigger.body.data.name}}",
    "description": "{{$trigger.body.data.description}}"
}
```

> Make sure that you are using the same key (here it's "item_read") as you set in node 5

Query is empty


***

- Node 10 - "Condition"

The flow is in the branch "item not found". So, if the action was to Update an item, then creating item is required (in the next node)

![node 10](/copy-this-template-directory/directus_flow_4_10.png "node 10")

Rules:
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


***

- Node 11 - "Create Data"

![node 11](/copy-this-template-directory/directus_flow_4_11.png "node 11")

Collection - a collection of your choice

Permissions - full access

Payload:
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

- Node 12 - "Transform Payload"

![node 12](/copy-this-template-directory/directus_flow_4_12.png "node 12")

if there was an error during operation. This node is not necessary


After you save this Flow, copy the resulting webhook URL somewhere.

***

&nbsp; 


## Setup in Google side

[Google Apps Script](https://developers.google.com/apps-script/overview) is usually enabled by default, but if you are using Google Workspace within the organization, your Admin might disable Google Apps Script.

Google Apps Script can be created as a dedicated file (Drive → New → More → Google Apps Script).

But for this project, we will use Script within Spreadsheet. That way it is easier to use a spreadsheet for config and for logging.

For quicker setup, please use [this spreadsheet template copy](https://docs.google.com/spreadsheets/d/1mXuSVR01uueeRDTjNeSXFH2rAziWkXdrNb2Jym413Fw/copy), it has full script inside and sheet with config.

In your spreadsheet copy, open the menu Extensions → Apps Script. It will open Script Editor.

![GAS 04](/copy-this-template-directory/GAS_04.png "GAS 04")

Click the button `Deploy` → `New Deployment`.

When Google Apps Script is deployed as Web App, it creates a unique URL like `https://script.google.com/macros/s/xxxx/exec`.
When this URL is called with a GET request, the script function `doGet` is executed.  When this URL is called with a POST request, script function `doPost` is executed. We will use POST requests and doPost function.
 
 ***

![GAS 05](/copy-this-template-directory/GAS_05.png "GAS 05")

set type to web app. Description - anything. Execute as - Me. Who has access - anyone.
Click `Deploy`.

 ***

![GAS 01](/copy-this-template-directory/GAS_01.png "GAS 01")

 click `Authorize access`
 ***

![GAS 02](/copy-this-template-directory/GAS_02.png "GAS 02")

 
Choose your account and then click "Allow".

Some users (usually, users not within the organization) will see a scary-looking

"Google hasn’t verified this app
The app is requesting access to sensitive info in your Google Account. Until the developer (your email) verifies this app with Google, you shouldn't use it."

Click "Advanced"

Then click "Go to code (unsafe)"

 
 ***

![GAS 03](/copy-this-template-directory/GAS_03.png "GAS 03")

click "Allow".

If all is fine, you'll see next popup with URL of your published web app.

Web app

URL

`https://script.google.com/macros/s/xxxxxxxxxxxxxxxxxxx/exec`

Copy

click `copy` to get this Web app URL (make sure that you are copying web app URL and not Library URL) and update Directus Environment Variable `GCALENDARHOOKURL` to this value


 ***

 Back to the script editor

![GAS 06](/copy-this-template-directory/GAS_06.png "GAS 06")

as shown by the arrow `1`, select function `listCalendars`. Click Run (as shown by the arrow `2`)

you'll see the result of the function listCalendars run. Copy the ID of the calendar that you want to track. Usually, it's the same as an account email.

&nbsp; 

### Update Spreadsheet Config

In your Spreadsheet go to the 'Config' sheet.

set `calendar_id` to the id of your calendar.

set `directus_url_proxy` to the URL of Flow "Google Calendar event Proxy"

set `directus_url_webhook_from_g` to the URL of Flow "Process events from Google Calendar"

set `pass` to the same value as in Directus Environment Variable `GCALENDARHOOKSECRET`

&nbsp; 

### Google Apps Script 

Back to the script editor. We need to run manually couple of functions.

select function `triggerResubscribeOnceWeek` (just like you selected listCalendars earlier). Click `Run`.

If the run was successful, in the `config` sheet values next to `channel_id`, `resource_id` will be filled.

select function `runManual_getSyncedEvents`. Click `Run`.

If the run was successful, in the `config` sheet value next to `sync_token`, `resource_id` will be filled.


> For most of the Google Workspace App, Apps Script has a specific library, like `CalendarApp` with easy-to-use functions. However, these functions don't have all the functionality available for API calls. Luckily it's possible to use Advanced Calendar Service - it's almost like calling API calls directly. Thanks to that we can subscribe to notifications and retrieve a list of new events using syncToken.

The calendar events push notifications need resubscription, its maximum time before expiration is 1 week.

***

Set Time Trigger

![GAS 09](/copy-this-template-directory/GAS_09.png "GAS 09")

Click `Triggers`, then `Add Trigger`

***

![GAS 10](/copy-this-template-directory/GAS_10.png "GAS 10")

Select function triggerResubscribeOnceWeek

Deployment - Head

Select event source - Time-driven

Select type - Week Timer

Select day of week and time - your choice

Failure notifications - Notify me immediately

Click `Save`

&nbsp; 

### Script source code - list of functions

The source code of the script is organized into several functions (but there is a lot of space for refactoring and improvements)

`writelog` - utility function to write to the spreadsheet log entry

`listCalendars` - function for manual launch - to list available for user calendars info (IDs, description)

`triggerResubscribeOnceWeek` - function for once-a-week time trigger. Will resubscribe to calendar push notifications (1 week is the maximum allowed time)

`runManual_processSyncedEvents` - function for manual launch - get events stream and process it (send to Directus)

`runManual_getSyncedEvents` - function for manual launch - set initial sync token, nothing sent to Directus

`getConfig` - utility function for retrieving Config set in spreadsheet

`updateConfig` - utility function for updating certain Config in the spreadsheet

`callDirectusWebhook` - actual data sent to Directus

`processSyncedEvents` - get events stream and process it (send to Directus)

`doPost` - main function, special name doPost means that when the web app is published, POST requests are processed here

`callCalendarEventsWatch_stop` - utility function for stopping push notifications

`callCalendarEventsWatch` - utility function for registering URL for receiving push notifications

`getSyncedEvents` - utility function for getting a list of new / updated / deleted events in Google Calendar, using special "syncToken"

&nbsp; 

#### Script update deployment

The deployed web app has versions, so if you update doPost function (or something called from it) and want these changes to have an effect - a new version of the web app needs to be deployed.

![GAS 07](/copy-this-template-directory/GAS_07.png "GAS 07")

Manage Deployments → `Edit` (pencil icon button)

![GAS 08](/copy-this-template-directory/GAS_08.png "GAS 08")

Choose `New version`, then click `Deploy`.

The URL of the web app should stay the same (if you updated the current deployment and didn't create new deployment)

***

&nbsp; 

### Ready to roll

I hope you are not exhausted after reading this post and repeating all the actions required for this project to run.

Everything is set to fully automatic two-way sync. You can try it by adding items in Directus and adding events in the Calendar.

You can check logs in the log panel for each of  Directus Flows. And you can check spreadsheet `log` sheet.

Feel free to send me a message if you know how to improve this project.

