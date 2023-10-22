---
title: 'Directus and Google Calendar sync'
description: 'how to synchronize items in Directus with Google Calendar events using Directus Flows and Google Apps Script'
author:
  name: 'Yury Klyuch'
  avatar_file_name: 'image_profile3_cr.jpg'
---

## Introduction
Wouldn't it be great to be able to sync items in Directus collection with Google Calendar Events???

## Before You Start
You will need a Directus project - check out [quickstart guide](https://docs.directus.io/getting-started/quickstart) if you don't already have one. You will also need Google Account.

**This project implementation limits:**
it is not supporting Directus Bulk operations. I only developed and tested on single item being created/updated/deleted.


## Interactions Scheme
on the high level it looks simple
![](/copy-this-template-directory/directus_gcalendar_shapes_highlevel.svg "high level interactions scheme overview")

but actual implementation look a bit more complex
![](/copy-this-template-directory/directus_gcalendar_shapes__23-10-22%2019.16.17.svg "detailed interactions scheme overview")

1 - Flow set to react on create/update events in our _collection1_. Another flow set to react on delete events. Both sends signal to our Google Apps Script webapp (2).

3 - same webapp (but different function) sends signals to another Directus Flow, that creates/updates/deletes items in collection1 accordingly (4).

5 - Google Calendar can send push notification when event added/updated/deleted, those notifications can be recieved directly by Google Apps Script webapp, but webapp cannot read request headers, so proxy Directus Flow (6) is used to modify request and send to webapp with parameters in body (7).

8 - Google Apps Script have additional functions (cron to renew notifications, function to stop notifications, etc)


**Let's dive in.**


## Setup in Directus side
Directus collection (_collection1_ in this sample project) have these fields:

_calendar_event_id_ - type text, where google calendar event id will be saved (automatically)

_calendar_event_start_ - type timestamp, where event start date is

_calendar_event_end_ - type timestamp, where event end date is

_name_ - type text, where event title is

_description_ - type text, where event description is

These fields are required, names could be changed, types should not be changed


now a Flows.


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
