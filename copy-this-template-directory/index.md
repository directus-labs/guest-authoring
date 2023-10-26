---
title: 'Directus and Google Calendar sync'
description: 'how to synchronize items in Directus with Google Calendar events using Directus Flows and Google Apps Script'
author:
  name: 'Yury Klyuch'
  avatar_file_name: 'image_profile3_cr.jpg'
---

## Introduction
Directus allows broad range of customization and extensibility. In this project we will create full two-way syncing between items in Directus collection and Google Calendar Events. So, when user create/update/delete item in Directus collection, corresponding event in Google Calendar will be created/updated/deleted. And vice versa, when user create/update/delete event in Google Calendar, corresponding item will be created/updated/deleted in Directus collection. On Directus side we will use Flows and on Google side we will use Google Apps Script.

```
Google Apps Script, eh?

Allow me a citation from official docs https://developers.google.com/apps-script/overview:
"Google Apps Script is a rapid application development platform that makes it fast and easy to create business applications
that integrate with Google Workspace. You write code in modern JavaScript and have access to built-in libraries for favorite
Google Workspace applications like Gmail, Calendar, Drive, and more.
There's nothing to install—we give you a code editor right in your browser, and your scripts run on Google's servers."

And my personal opinion:
It's free, it's powerfull, it's easy to use, have great docs, it have good resources quota, it have access to almost
all the things you can do in Gmail / Doc / Drive / Spreadsheet / Slide / Forms / Calendar.

```

&nbsp; 

## Before You Start
You will need a Directus project - check out [quickstart guide](https://docs.directus.io/getting-started/quickstart) if you don't already have one. You will also need Google Account.

**This project implementation limits:**
it is not supporting Directus Bulk operations. I only developed and tested it for single item being created/updated/deleted.

&nbsp; 

## Interactions Scheme
on the high level it might look simple
![](/copy-this-template-directory/directus_gcalendar_shapes_highlevel.svg "high level interactions scheme overview")

but actual implementation look a bit more complex

```
Sources of complexity:
- For proper syncing, we need to have some id that is shared between Directus item and Google Calendar event.
This will be id of google event, saved as additional field in Directus item. So, in Flows there will be actions to update this id 
with the value from Google Apps Script, and also actions to search Directus item by this id.
- it's not very straightforward to organize stream of updates from Google Calendar.
```

![](/copy-this-template-directory/directus_gcalendar_shapes__23-10-22%2019.16.17.svg "detailed interactions scheme overview")

1 - Flow set to react on create/update events in our _collection1_. Another flow set to react on delete events. Both sends signal to our Google Apps Script webapp (2).

3 - same webapp (but different function) sends signals to another Directus Flow, that creates/updates/deletes items in collection1 accordingly (4).

5 - Google Calendar can send push notification when event added/updated/deleted, those notifications can be recieved directly by Google Apps Script webapp, but webapp cannot read request headers, so proxy Directus Flow (6) is used to modify request and send to webapp with parameters in body (7).

8 - Google Apps Script have additional functions (cron to renew notifications, function to stop notifications, etc)

&nbsp; 

**Let's dive in.**

&nbsp; 

## Setup in Directus side
Directus collection (_collection1_ in this sample project) have these fields:

`calendar_event_id` - type text, where google calendar event id will be saved (automatically)

`calendar_event_start` - type timestamp, where event start date is

`calendar_event_end` - type timestamp, where event end date is

`name` - type text, where event title is

`description` - type text, where event description is

> `timestamp` type is available when field is created in Advanced Mode. Could be Datetime type used instead? It might work, but since it doesn't have timezone info, you must ensure that your Directus and Google Calendar have same timezone setting.

These fields are required, names could be changed, types should not be changed

&nbsp; 

In the Flows we will use [environment variables](https://docs.directus.io/self-hosted/config-options.html). Please add these two environment variables:

`GCALENDARHOOKSECRET=supersecretpass`

`GCALENDARHOOKURL=https://script.google.com/macros/s/xxxx/exec`

First is the secret word our Flows will use to check that incoming data is from our trusted script, and second is the actual URL of published google apps script, you will acquire it later. In order to Flows to have access to variables, they need to be listed in another variable:

`FLOWS_ENV_ALLOW_LIST=GCALENDARHOOKSECRET,GCALENDARHOOKURL`

&nbsp; 

**now a Flows.**

There are 4 Flows. For each Flow I'll show overview picture and then each node screen with config also as text, so you can copy-paste it.

&nbsp; 

### Flow "Google Calendar event Proxy"

As I mentioned - it's not very straightforward to organize stream of updates from Google Calendar with Google Apps Script.
We will use [push notifications](https://developers.google.com/calendar/api/guides/push) that will call our published Google Apps Script when there is changes in calendar. The catch though, these push notifications provide data within Header but for incoming requests Google Apps Script can't read headers, only body. That's why we will use Directus Flow as simple proxy, it's webhook url will be registered as address for push notifications, it will receive data, save header as body and sent it to our published Google Apps Script.

![whole flow](/copy-this-template-directory/directus_flow_1_full_.png "whole flow")

***

- Trigger node - webhook:

![trigger node](/copy-this-template-directory/directus_flow_1_01_.png "trigger node")


***

- Node 2 - "Webhook / Request URL"

![request node](/copy-this-template-directory/directus_flow_1_02_.png "request node")

url is `{{$env.GCALENDARHOOKURL}}` - actual value in the environment variable will be set after Google Apps Script is published.

Request body:
```js
{
  "headers": {{$trigger.headers}}
}
```

> note that {{$trigger.headers}} is not quoted!

After you save this Flow, copy resulting webhook url somewhere.

***

&nbsp; 

### Flow "Send delete event to Google Calendar"

Processing of Delete event is a bit different from processing of Create / Update (next flow). It should be set to blocking, cause we need to "intercept" delete command and read item data - we need to know id of Google Calendar event, so we can send it to Published Google Apps Script.

![whole flow](/copy-this-template-directory/directus_flow_2_full_.png "whole flow")

***

- Trigger node - Event Hook

![trigger node](/copy-this-template-directory/directus_flow_2_01_.png "trigger node")

make sure that you have same config:

Type - Filter(Blocking)

Scope - items.delete

Collections - collection of your choice

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

url is `{{$env.GCALENDARHOOKURL}}` - actual value in the environment variable will be set after Google Apps Script is published.

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
Processing of Create / Update is more complicated than Delete, cause after we sent this event info, we might receive id of Google Calendar Event that was created and we must update current Directus item with this id. This operation is not blocking. 

![whole flow](/copy-this-template-directory/directus_flow_3_full_.png "whole flow")

***

- Trigger Node - Event Hook

![trigger node](/copy-this-template-directory/directus_flow_3_01.png "trigger node")

parameters:

Type - Action (Non-Blocking)

Scope - items.create, items.update

Collections - collection of your choice


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

url is `{{$env.GCALENDARHOOKURL}}` - actual value in the environment variable will be set after Google Apps Script is published.

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

Collection - collection of your choice

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

> make sure that you are using same key (here it's "request_webhook_update") as you set in node 4

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

url is `{{$env.GCALENDARHOOKURL}}` - actual value in the environment variable will be set after Google Apps Script is published.

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

Collection - collection of your choice

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

> make sure that you are using same key (here it's "request_webhook_create") as you set in node 9

Query is empty

***

&nbsp; 

### Flow "Process events from Google Calendar"

This is a webhook, called from Google Apps script when event created / updated / deleted in Google Calendar.

Collection1 item will be Created / Updated / Deleted from incoming hook parameters.

This flow is a bit wide, so screenshot is split in two.


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
  
simple gatekeeper - check that password from incoming parameter is same as saved in Environment Variable

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

determine from incoming paramenter, if Directus item need to be created

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

Collection - collection of your choice

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

Collection - collection of your choice

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

if such item is found (then proceed to actions delete/update, oherwise - create new item (for update action))

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

determine from incoming paramenter, if Directus item need to be deleted

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

Collection - collection of your choice

IDs (edit raw value):
```js
[
    "{{item_read[0].id}}"
]
```
> make sure that you are using same key (here it's "item_read") as you set in node 5

Query is empty


***

- Node 9 - "Update Data"

![node 09](/copy-this-template-directory/directus_flow_4_09.png "node 09")

Collection - collection of your choice

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

> make sure that you are using same key (here it's "item_read") as you set in node 5

Query is empty


***

- Node 10 - "Condition"

The flow is in the branch "item not found". So, if action was to Update item, then create item is required (in next node)

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

Collection - collection of your choice

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

if there was error during operation. This node is not necessary


After you save this Flow, copy resulting webhook url somewhere.

***

&nbsp; 


## Setup in Google side

[Google Apps Script](https://developers.google.com/apps-script/overview) is usually enabled by default, but if you are using Google Workspace within organization, your Admin might disable Google Apps Script.

Google Apps Script can be created as dedicated file (Drive → New → More → Google Apps Script).

But for this project we will use Script within Spreadsheet. That way it is easier to use spreadsheet for config and for logging.

For quicker setup, please use [this spreadsheet template copy](https://docs.google.com/spreadsheets/d/1QSKNNqRbpVBW1OAYg_8kWOCbK4RAM8FwAL8D701LEdY/copy), it have script inside.

In yours spreadsheet copy, open menu Extensions → Apps Script. It will open Script Editor.

Click button "Deploy" → New Deployment

Web app

Execute as - Me (info@flowmata.com)

Who has access - Anyone

Click Deploy, then "Authorize Access". Choose your account and then click "Allow"

Some users (usually, users not within organization) will see a bit scary looking "This App is not Verified by Google" - then you should click "Continue anyway"

Web app
URL
https://script.google.com/macros/s/AKfycbxabMEaxAV5J-8JNehR-gvRQDvWtTxt-2WjkFGgZsej1tAX37lPY29YAb-eoNQUnAj2/exec
Copy

&nbsp; 

### Google Apps Script 


&nbsp; 

### Deploy Google Apps Script 


&nbsp; 

text **text**, **text**. With [link](https://www.google.com/), text

 - [x] check1
 - [x] check2
 - [x] check3
