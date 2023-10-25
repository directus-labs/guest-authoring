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


## Before You Start
You will need a Directus project - check out [quickstart guide](https://docs.directus.io/getting-started/quickstart) if you don't already have one. You will also need Google Account.

**This project implementation limits:**
it is not supporting Directus Bulk operations. I only developed and tested it for single item being created/updated/deleted.


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


**Let's dive in.**


## Setup in Directus side
Directus collection (_collection1_ in this sample project) have these fields:

`calendar_event_id` - type text, where google calendar event id will be saved (automatically)

`calendar_event_start` - type timestamp, where event start date is

`calendar_event_end` - type timestamp, where event end date is

`name` - type text, where event title is

`description` - type text, where event description is

These fields are required, names could be changed, types should not be changed


In the Flows we will use environment variables.
[docs](https://docs.directus.io/self-hosted/config-options.html)

`GCALENDARHOOKSECRET=supersecretpass`

`GCALENDARHOOKURL=https://script.google.com/macros/s/xxxx/exec`

First is the secret word our Flows will use to check that incoming data is from our trusted script, and second is the actual URL of published google apps script, you will acquire it later. In order to Flows to have access to variables, they need to be listed in another variable:

`FLOWS_ENV_ALLOW_LIST=GCALENDARHOOKSECRET,GCALENDARHOOKURL`


**now a Flows.**

### Flow "Google Calendar event Proxy"
As I mentioned - it's not very straightforward to organize stream of updates from Google Calendar with Google Apps Script.
We will use [push notifications](https://developers.google.com/calendar/api/guides/push) that will call our published Google Apps Script when there is changes in calendar. The catch though, these push notifications provide data within Header but for incoming requests Google Apps Script can't read headers, only body. That's why we will use Directus Flow as simple proxy, it's webhook url will be registered as address for push notifications, it will receive data, save header as body and sent it to our published Google Apps Script.

![whole flow](/copy-this-template-directory/directus_flow_1_full_.png "whole flow")


this is the trigger node, webhook:

![trigger node](/copy-this-template-directory/directus_flow_1_01_.png "trigger node")


which goes into "Webhook / Request URL" node:

![request node](/copy-this-template-directory/directus_flow_1_02_.png "request node")

url is `{{$env.GCALENDARHOOKURL}}` - actuall value in the environment variable will be set after Google Apps Script is published.

Request body:
```js
{
  "headers": {{$trigger.headers}}
}
```

note that {{$trigger.headers}} is not quoted!

After you save this Flow, copy resulting webhook url somewhere.

|  
 
### Flow "Send delete event to Google Calendar"

Processing of Delete event is a bit different from processing of Create / Update (next flow). It should be set to blocking, cause we need to "intercept" delete command and read item data - we need to know id of Google Calendar event, so we can send it to Published Google Apps Script.

![whole flow](/copy-this-template-directory/directus_flow_2_full_.png "whole flow")

Trigger - Event Hook

![trigger node](/copy-this-template-directory/directus_flow_2_01_.png "trigger node")

make sure that you have same config:

Type - Filter(Blocking)

Scope - items.delete

Collections - collection of your choice

Response - data of last operation

which goes into "Read data" node:

![node 02](/copy-this-template-directory/directus_flow_2_02_.png "node 02")

IDs edit raw value to:
```js
[
    "{{$trigger.payload[0]}}"
]
```
Query is empty

node goes into "Webhook / Request URL" node:

![node 03](/copy-this-template-directory/directus_flow_2_03_.png "node 03")

url is `{{$env.GCALENDARHOOKURL}}` - actuall value in the environment variable will be set after Google Apps Script is published.

Method is Post

Request body:
```js
{
  "data": {{$last}},
  "action": "delete",
  "pass": "{{$env.GCALENDARHOOKSECRET}}"
}

```

note that {{$last}} is not quoted!

|
 
### Flow "Send create/update event to Google Calendar"
Processing of Create / Update is more complicated than Delete, cause after we sent this event info, we might receive id of Google Calendar Event that was created and we must update current Directus item with this id. This operation is not blocking. 

![whole flow](/copy-this-template-directory/directus_flow_3_full_.png "whole flow")

Trigger - Event Hook

![trigger node](/copy-this-template-directory/directus_flow_3_01.png "trigger node")

make sure that you have same config:

Type - Async

Scope - items.create, items.update

Collections - collection of your choice

Response - data of last operation

![node 02](/copy-this-template-directory/directus_flow_3_02.png "node 02")

node config.

it goes into Something

![node 03](/copy-this-template-directory/directus_flow_3_03.png "node 03")

node config.

it goes into Something

![node 04](/copy-this-template-directory/directus_flow_3_04.png "node 04")

node config.

it goes into Something

![node 05](/copy-this-template-directory/directus_flow_3_05.png "node 05")

node config.

it goes into Something

![node 06](/copy-this-template-directory/directus_flow_3_06.png "node 06")

node config.

it goes into Something

![node 07](/copy-this-template-directory/directus_flow_3_07.png "node 07")

node config.

it goes into Something

![node 08](/copy-this-template-directory/directus_flow_3_08.png "node 08")

node config.

it goes into Something

![node 09](/copy-this-template-directory/directus_flow_3_09.png "node 09")

node config.

it goes into Something

![node 10](/copy-this-template-directory/directus_flow_3_10.png "node 10")

node config.

it goes into Something

![node 11](/copy-this-template-directory/directus_flow_3_11.png "node 11")

node config.

it goes into Something

|

### Flow "Process events from Google Calendar"
create / update / delete Collection1 item from incoming hook parameters. Called from Google Apps script

part 1

![whole flow](/copy-this-template-directory/directus_flow_4_full_1_.png "whole flow")


part 2

![whole flow](/copy-this-template-directory/directus_flow_4_full_2_.png "whole flow")

Trigger - Event Hook

![trigger node](/copy-this-template-directory/directus_flow_4_01.png "trigger node")

make sure that you have same config:

Type - Async

Scope - items.create, items.update

Collections - collection of your choice

Response - data of last operation

![node 02](/copy-this-template-directory/directus_flow_4_02.png "node 02")

node config.

it goes into Something

![node 03](/copy-this-template-directory/directus_flow_4_03.png "node 03")

node config.

it goes into Something

![node 04](/copy-this-template-directory/directus_flow_4_04.png "node 04")

node config.

it goes into Something

![node 05](/copy-this-template-directory/directus_flow_4_05.png "node 05")

node config.

it goes into Something

![node 06](/copy-this-template-directory/directus_flow_4_06.png "node 06")

node config.

it goes into Something

![node 07](/copy-this-template-directory/directus_flow_4_07.png "node 07")

node config.

it goes into Something

![node 08](/copy-this-template-directory/directus_flow_4_08.png "node 08")

node config.

it goes into Something

![node 09](/copy-this-template-directory/directus_flow_4_09.png "node 09")

node config.

it goes into Something

![node 10](/copy-this-template-directory/directus_flow_4_10.png "node 10")

node config.

it goes into Something

![node 11](/copy-this-template-directory/directus_flow_4_11.png "node 11")

node config.

it goes into Something

![node 11](/copy-this-template-directory/directus_flow_4_12.png "node 12")

node config.


|



## Setup in Google side
something something something something
something something 


### Google Apps Script 
something something something something
something something 


### deploy Google Apps Script 
something something something something
something something 


text **text**, **text**. With [link](https://www.google.com/), text

> text **text** text.

 - line 1
 - line 2
 - line 3

 - [x] check1
 - [x] check2
 - [x] check3
