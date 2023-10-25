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

### Flow "webhook_proxy4calendar"
proxy that was registered as Google Event Channel address, sent data to Apps Script



### Flow "collection1_delete_2GCalendar"
send data to Google Apps Script with Delete data
something something 



### Flow "after_collection1_CreateUpdate"
this is auto trigger that will call Google App Script
something something 



### Flow "webhookFromGCalendar2Coll1"
create/update/delete Collection1 item from incoming hook parameters. Called from Google Apps script
something something 





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

![This is an alt text.](/copy-this-template-directory/image_profile3_cr.jpg "This is a sample image.")
