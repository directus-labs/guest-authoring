---
title: 'Using Directus as a Baby Health Tracker'
description: 'This article highlights using Directus as a central backend for smart wearables, offering step-by-step guides for setup, data retrieval, and integration with third-party tools like OpsGenie and iPhone widgets.'
author:
  name: 'Andreas Morgner'
  avatar_file_name: 'Andreas_Morgner.jpg'
---

## Summary
In this article, we will explore Directus as a backend for a smart wearable device, and how to extend the functionality of the the apps shipped with the device.
We will cover data model configuration, how to grab data from the wearable using Directus Flows, and how to integrate Directus into 3rd party tools for reporting, such as OpsGenie and screen widgets on an iPhone.

## Before You Start
To get on common ground, let's have a look at the used tools and infrastructure first.
Next to a running Directus instance, regardless if self-hosted or within Directus Cloud. 
For Directus I expect you to have the permission to create new Collections as well as new Flows, so an admin-access level would be required. 

Next, you need a wearable with an API to consume the data. Of course, all other sensor data would work as long as we can access the data by a web request. Depending on your used sensors, the final Flow might look different. In our case, we used the Neebo tracker first and switched to Owlet. Both provide an API with some authentication set up in front. Unfortunately, there is no official documentation of the API so it was kind of reverse engineering the network traffic and digging through a couple of GitHub repositories.

Last but not least, a basic understanding of Directus and RESTful APIs and web requests is helpful, but we'll provide as many details as possible to make it as easy as possible to follow, even if you're new to Directus (or APIs).

## Implementation
### Collections
#### The data storage collection
As we want to track sensor data over time, we will frequently pull the data with some details from the external API. Create a collection called `sensor_data` with a auto-generated UUID as an ID. Allow Directus to create `sort`, `status`, and `date_created` fields as well.
For each element of the data provided by the sensor, we want to save these values. Depending on the used sensor, you may come up with a different set of fields. In our case we've created the following important additional fields:

| Field name | Interface | Comment |
| ---- | ---- | ---- |
| raw_payload | Code | We use this to store the JSON payload that we gather from the external API for debugging purposes. |
| temperature | Input, Decimal | Plain value from the sensor data. |
| oxygen | Input, Decimal | Plain value from the sensor data. |
| heart_rate | Input, Decimal | Plain value from the sensor data. |
| battery_level | Input, Integer | Plain value from the sensor data. |
| battery_status | Dropdown | To evaluate the sensor data. |
| heart_status | Dropdown | To evaluate the sensor data. |
| oxygen_state | Dropdown | To evaluate the sensor data. |
| thermal_state | Dropdown | To evaluate the sensor data. |
This collection is for integration into the OpsGenie alerting system.

| Field name | Interface | Comment |
| ---- | ---- | ---- |
| id |  |  |
| external_id |  |  |
| sort |  |  |
| user_created |  |  |
| user_updated |  |  |
| date_created |  |  |
| date_updated |  |  |
| category |  |  |
| type |  |  |
| severity |  |  |
| status |  |  |
| priority |  |  |
| message |  |  |
| alias |  |  |
| details |  |  |
| entity |  |  |
| description |  |  |
| neboo_data |  |  |
| links |  |  |
### Flows
A Flow inside Direcuts can be used to automate things. Each Flow contains of a trigger and one or more operations. A trigger could be time based, manual, or based on events coming from actions like creating or updating items. As operations we can use CRUD operations to work with items or create new items, if-else condtions, web requests, and even more operations.

#### Read Data Flow
The by far most complex Flow of this use case is the one we need to extract the data. This is due to the lack of publicly documented API of the used health-tracking gadget. 
As there are a couple of Python libraries fetching the data, it was not the easiest way, but with a basic understanding of Python and programming in general, it was possible to kind of reverse-engineer this code and translate each step into an Operation for the Flow.
The very rough workflow looks like this:
1. Define the trigger
2. Authenticate and get all the needed tokens
3. Get a list of connected devices
4. Get the data of our target device
5. Transform the data to our needs
6. Save the data
##### The Trigger
The easiest way to get regular data fetched automatically with a Flow is a scheduled trigger.
During the setup, we've set the trigger to the following settings:

Status: Active
Activity & Log tracking: Track Log & Activity 
Interval: * * * * * (which means, it runs every minute)

We highly recommend turning off the logging or at least reducing it to prevent the flow from writing too much data into the history. 
##### Authenticate and request a Token
For this specific API, the authentication is a bit more complex compared to basic auth or a simple "request a token" as we might know it from the easy-to-use-api from Directus.
The general process to get a token which can be used to access actual device data contains three steps:

1. First, we use the regular login credentials (the one I can log in within the mobile App) and pass them to an auth service provided by Google. If successfully logged in, we receive a first token.
2. With this token we can now secondly authenticate against the actual vendor server. But again we just get a second token.
3. Within the third request we pass the 2nd token to the vendor API once again. For this request, we will receive a last token that can be used to fetch actual data from the API.

!(Pasted image 20240214224605.png)

The screenshot shows the last request. As a key, we've set `token_sign_in` which can be used in the next operation. Within the payload/request body, we've specified the token to use with the dynamic code `"{{$last.data.mini_token}}"`. The object `$last` refers to the previous operation, an easy way to access the results from the last request. 

!(Pasted image 20240216105507.png)
 
##### Getting actual user data
Compared to the auth process, fetching actual data from the API is done with fewer requests – just two are needed. As the vendor supports multiple devices, we have to get a list of devices first. In theory, we can also do this in preparation and hard-code the device ID within the next operation. But again, as there is no official documentation of the API this might change or the ID (which is more of a name) might not that unique and may change within any future updates. 

So let's catch the device name, or `dsn` as it is called by the external API. Again we are using a Web request with the last token we got from the auth-process.

The GET URL is static and does not contain any dynamic value this time.

| Method | URL |
| --- | --- |
| GET | https://ads-field-eu-1a2039d9.aylanetworks.com/apiv1/devices.json |
As header parameter, we set the following:

| Header | Value |
| --- | --- |
| Authorization | `auth_token {{$last.data.access_token}}` |
As you can see, we use the `{{ … }}` to pass the access_token from the last auth request.

If everything works so far, the JSON we receive looks something like this, but for security reasons, some data was removed or masked as it also contains coordinates, MAC addresses, IP addresses and so on:
```json
{
  "status": 200,
  "statusText": "OK",
  "headers": {
    ...
  },
  "data": [
    {
      "device": {
        "dsn": "AC00***697",
        "lan_ip": "192.168.*.*",
        "lan_enabled": false,
        "has_properties": true,
        "connection_status": "Online",
        "lat": "********",
        "lng": "********",
        "device_type": "Wifi"
      }
    }
  ]
}
```

The device object has exactly one tracker and provides us with the `dsn` as well as the flag for `has_properties`. With the next operation, we will request all of these properties to extract the sensor data from the tracker.

The last Web-Request to get the properties of the device looks like this:

| Method | URL |
| --- | --- |
| GET | `https://ads-field-eu-1a2039d9.aylanetworks.com/apiv1/dsns/{{$last.data[0].device.dsn}}/properties.json` |
As a header parameter, we set the following:

| Header | Value |
| ---- | ---- |
| Authorization | `auth_token {{token_sign_in.data.access_token}}` |
As you can see, the named operation becomes handy now.

This request returns every property the device currently has. In total, we received over 1600 lines of JSON this time. The good thing, we only have a single property object here. All properties have a name attribute. A very easy way is to loop over all properties and find the one we need. To achieve this within the flow, the Script-Operation can be used.

The run operation looks like this
   ```Javascript
module.exports = async function(data) {
    //console.log(data.read_dns.data[0])
    let rtnData
	data.read_dns.data.forEach(p => {
      if (p.property.name === "REAL_TIME_VITALS") {
        //console.log(JSON.parse(p.property.value))
        rtnData = JSON.parse(p.property.value)
      }
    });
    
    return rtnData
}
```

Within `rtnData` we now store all the relevant values such as battery details, heart rate and oxygen level. As the JSON data is stored as a raw string with the value attribute, the run operation already parses the string and returns the following JSON object back to the Flow

```Json
{
  "ox": 0,
  "hr": 0,
  "mv": 0,
  "sc": 2,
  "st": 0,
  "bso": 0,
  "bat": 100,
  "btt": 0,
  "chg": 2,
  "aps": 0,
  "alrt": 0,
  "ota": 0,
  "srf": 1,
  "rsi": 18,
  "sb": 0,
  "ss": 0,
  "mvb": 0,
  "mst": 0,
  "oxta": 255,
  "onm": 0,
  "bsb": 4,
  "mrs": 0,
  "hw": "obl"
}
```

Some of the keys are self-speaking while others are not clear. Due to the lack of documentation, I can only compare the values with the data from the original app to identify what is what. Also, the mentioned GitHub projects have some details from other users that were helping to understand the keys here.

##### Saving the data within Directus
The last step within the flow is to save the gathered data within the collection we've set up already. The payload of the create operation  looks like this:

```Json
{
    "raw_payload": "{{$last}}",
    "oxygen": "{{$last.ox}}",
    "heart_rate": "{{$last.hr}}",
    "temperature": "{{$last.st}}",
    "battery_level": "{{$last.bat}}"
}
```

From here, we have all the basic data saved within Directus, if the Flow runs without any issues, don't forget to turn off the logging of the flow and maybe also the logging for the data collection to reduce the amount of overhead data within the Flow log and activity table for the collection.
## Reporting
### Insights
As the Insights module is built in Directus, you can create any report with the data just as you need it. In our case, we've created the following dashboard:
!(Pasted image 20240305062855.png)

The dashboard is split into three sections, on the left, the last 30 minutes of data are displayed. On the right, you can select the amount of hours you'd like to review. This is done by a slide that is used as a global variable for the 3 charts underneath. 
In between, some stats are displayed, in detail, in the very last dataset for the oxygen level, heart rate, temperature and battery stats. 

All the charts are using the "Time Series" chart. The left side uses the default values for the data range of 30 minutes. For each measurement, the min and max values are set to render the most common values including some border values. The dynamic charts on the right are set up similarly. 

To achieve the dynamic time window, we have to create the "Global Variable" with the name 'last_x_hours' first:
!(Pasted image 20240223104116.png)

Within the dynamic "Time Series" charts we can reuse the global variable by adding its name to the date range:
!(Pasted image 20240223104315.png)
Again we can use the `{{…}}` syntax, followed by a lower `h` to identify the value that should work as hours. If you use the same variable for all three slides, they will change altogether once you've selected a new amount of hours within the global variable slider.

For the basic Metric values, you use the following configuration:
!(Pasted image 20240223105348.png)
To make the values more self-speaking, the style and format can be adjusted e.g. by different colours based on the current values.
!(Pasted image 20240223105507.png)
### 3rd Party Integration
Last, we've added some external services to get a robust alerting system and a handy quick view of the current status directly on my mobile phone.
#### Evaluate the Data and create an alert record
For the alerting system, we're using OpsGenie, usually used for system, infrastructure or app health status. But why not for the actual health use case?
To build this, a simple flow listens to every created event within the data collection. Every new record in this collection is evaluated and under certain conditions, a new item is created within an additional collection called "OpsGenie alert". The collection looks something like this to store all relevant data that OpsGenie can use:
!(Pasted image 20240304172941.png)
The related Flow looks like this:
!(Pasted image 20240304173609.png)
Within the Run Script operation the data is evaluated and based on the met conditions, the returning JSON is passed over to the next operation.
For checking the battery level the code looks like this:
```javascript
module.exports = async function(data) {
    var rtn = {
    	"create_alert": false
    }
    if (data.$trigger.payload.battery_level < 20) {
    	rtn.create_alert = true
        rtn.alert_data = {
        	"category": "battery",
            "type": "info",
            "severity": "mid",
            "priority": "P3",
            "alias": "Low Battery",
            "message": "The battery is low. Right now it has "+data.$trigger.payload.battery_level+"%",
            "description": "The battery is low. Right now it has "+data.$trigger.payload.battery_level+"%"
        }
    }
    // ... 
	return rtn;
}
```


The next Condition Operation within the Flow checks the value of `create_alert` and either stops here if `false` or passes the data to the final Create operation if set to `true`. This can be done easily by the following condition rule:

```json
{
    "$last": {
        "create_alert": {
            "_eq": true
        }
    }
}
```
You can build in several other logic like checking if there is an open alert already and stuff like this. To keep it simple we're skipping this at the moment. (And trust OpsGenie to not raise two alerts for the same incoming data)

Within the create operation, the following payload is used to create the Alert item:
```json
{
    "category": "{{exec_alert_logic.alert_data.category}}",
    "type": "{{exec_alert_logic.alert_data.type}}",
    "severity": "{{exec_alert_logic.alert_data.severity}}",
    "priority": "{{exec_alert_logic.alert_data.priority}}",
    "alias": "{{exec_alert_logic.alert_data.alias}}",
    "message": "{{exec_alert_logic.alert_data.message}}",
    "description": "{{exec_alert_logic.alert_data.description}}"
}
```
#### Escalate the alert data to OpsGenie    
Once the alert is created within Directus, of course, it has to be created within OpsGenie as well. For this – again a new Flow is needed. This time we have to use the Timeout operation as OpsGenie queues all incoming requests and does not provide the result immediately.

!(Pasted image 20240304175218.png)
Once triggered, we read out all the data first and pass the needed data to the OpsGenie API with a user key and the JSON data from the alert collection. As the API does not return the actual result but only an ID of the incoming task, we now wait 2 seconds before requesting the current status of our queued task with the 2nd web request. The request URL gets the returning ID from the 1st web request: 
!(Pasted image 20240304175856.png)

This request will return the current status and therefore all the data (including an ID) of this alert. With the update operation, we saved the alert ID to be able to refer to the OpsGenie alert if needed.
In the moment an actual alert is created within OpsGenie, any connected system is yelling for attention, just as configured in the tool.
If Directus tries to create a new alert with the same data (in case the battery or oxygen level is still on the same level) no new alert is created by the design of OpsGenie. But it's hard to miss any alert as it sends SMS, critical push notifications or even calling a phone number if needed and no reaction is noticed. 
#### As a Widget for iOS
As a last method of reporting and integration Directus, we can create a simple iOS widget, that sits right on my lock and home screen. As this is not about iOS development we chose the easy way and used Scriptable for this. This free app can be used to run JavaScript and display the data as a widget. The only downside of these Scriptable widgets is, that iOS decides WHEN to update the content of the widget. So you can not force the widget to reload actively. This is a limitation to prevent battery drainage and high load due to the demanding JavaScript. Usually, the script runs every 1 to 15 minutes. To know how "outdated" the displayed data is, we've added the create-timestamp as part of the widgets. 
For the Directus integration this time I have created a new role that can have view access to the data collection only. A user in this role has set a static token that I can use within the widget. Using the query parameter `limit` and `sort` I can define to get only the very last entry sorted by the create date column.

```javascript
const url = "https://my-directus.app/items/neebo_data?limit=1&sort=sort,-date_created"
const token = "xy-my-static-token"
async function getData(url) {
  let request = new Request(url);
  let authHeader = "Bearer " + token
  request.headers = { "Authorization": authHeader }

  let data = await request.loadJSON();
  return data;
}
function processData(data) {
  console.log(data)
  let processedData = {
    "placement": data.data[0].placement,
    "battery": {
      "level": data.data[0].battery_level,
      "state": data.data[0].battery_state
    },
    "termal": {
      "state": "",
      "value": data.data[0].temperature
    },
    "oxygen": {
      "state": "",
      "value": data.data[0].oxygen
    },
    "heart_rate": {
      "state": "",
      "value": data.data[0].heart_rate
    },
    "reldate": new RelativeDateTimeFormatter().string(new Date(data.data[0].date_created), new Date()),
    "datecreated":
      new Date(data.data[0].date_created).getHours() + ":" +
      String(new Date(data.data[0].date_created).getMinutes()).padStart(2, "0")
  }

  return processedData;
}
function createWidget(data) {
  var widget = new ListWidget();
  console.log(data.placement);
  switch (data.placement) {
    case 4:  // Charging
      var text = widget.addText("⚡️ " + data.battery.level + "% …");
      break;
    case "2":
      break;
    default:
      var text = widget.addText("🌡️ " + data.termal.value + "°C");
      var text = widget.addText("🫧 " + data.oxygen.value + " ♥️ " + data.heart_rate.value);
      break;
  }
  var text = widget.addText(data.datecreated);
  text.font = Font.regularRoundedSystemFont(12);
  return widget;
}
let data = await getData(url);
let processedData = processData(data);
let widget = await createWidget(processedData);
Script.setWidget(widget);
Script.complete()
```

## Wrap up
And Voila, you've successfully built your monitoring system by using Direcuts. In this post, we covered how to set up the needed Collections to store data from an external API using Flows, worked with the data to further integrate the collected data into third-party tools and built our reporting Dashboard directly within Direcuts using Insights as well as display the latest data on our mobile phone.