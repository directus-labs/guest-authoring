---
title: "Directus and IoT: Sensor Data with an ESP32"
description: "IoT applications can write data to Directus and have this data visualized through Directus Dashboards. This is achieved through the Directus API and an ESP32."
author:
  name: "Osinachi Chukwujama"
  avatar_file_name: "osiheadshot.png"
---

# Directus and IoT: Sensor Data with an ESP32

IoT systems will not be complete without logging their collected data to remote cloud storage. You can choose to build a complete backend system with HTTP, authentication, and persistent storage or use a ready-made system known as Directus. Directus offers a complete Backend as a Service (BaaS) platform that you can use for your IoT projects. It gives you an easy way to connect to databases, create tables, and have these tables exposed as HTTP APIs. To demonstrate the power of Directus in IoT, you will build a simple temperature logging system using a DHT22 as a temperature and humidity module, and an ESP32 as the microcontroller board with onboard WiFi for connecting to a local or remote Directus instance.

As a bonus, you will see how your temperature and humidity data changes in real time using Directus Dashboards. Directus Dashboards is an insights service packaged with Directus to help teams gain insight into their data. So on the insights page, you will create different charts that update at an interval and shows how your logged data changes with time.

This article presents several approaches to achieving the end results (IoT system to Directus). The default approach is using local physical components that can be purchased from a hardware store to collect and log the data. The data is then logged to a local instance of Directus, created via Docker. The alternative is using a simulation environment known as [Wokwi](https://wokwi.com/) to simulate the temperature and humidity data logging. In this alternative case, you can use a local or cloud instance of Directus.

## Prerequisites

This section lists the prerequisites depending on the approach you take. There are two major requirements, the IoT requirements and the Directus requirements. Pick and choose the approach that best suits you.

### IoT Component Requirements

For the IoT components you either use physical components or the simulator environment.

#### Using Physical Components

1. An [ESP32](https://www.espressif.com/en/products/socs/esp32) development board. You can use an existing board or purchase the Wroom 32D model from your local vendor. Here are links for [Amazon](https://www.amazon.com/HiLetgo-ESP-WROOM-32-Development-Microcontroller-Integrated/dp/B0718T232Z/), [AliExpress](https://www.aliexpress.com/item/1005006018016908.html), and [Hub360](https://hub360.com.ng/product/esp32-devkitc-core-board-esp32-development-board-esp32wroom-32d/)
2. A [DHT22 sensor](https://www.adafruit.com/product/385). You can reuse the one you own, purchase from a local vendor, or buy online from [Amazon](https://www.amazon.com/Teyleten-Robot-Digital-Temperature-Humidity/dp/B0CPHQC9SF/), [AliExpress](https://www.aliexpress.com/item/1005005545184001.html), and [Hub360](https://hub360.com.ng/product/dht22-temperature-and-humidity-sensor/)
3. Type-B micro USB cable
4. Three (3) male to female jumper cables (may be optional, depending on the configuration of your DHT22)
5. The latest Arduino IDE which can be installed for [Windows](https://downloads.arduino.cc/arduino-ide/arduino-ide_latest_Windows_64bit.exe), [MacOS](https://downloads.arduino.cc/arduino-ide/arduino-ide_latest_macOS_64bit.dmg), and [Linux](https://downloads.arduino.cc/arduino-ide/arduino-ide_latest_Linux_64bit.AppImage)
6. The ESP32 board support for Arduino IDE. Follow the official [Espressif documentation](https://docs.espressif.com/projects/arduino-esp32/en/latest/installing.html) to install it for your IDE

### Using the Online Simulator Environment (Wokwi)

1. A Wowki Club account 

### Directus set up requirements

For setting up Directus, you either use a local instance of Directus using Docker or you use a cloud instance. You should prefer the local instance because it doesn't require you to pay and you can experiment all you want. The cloud instance should be used when you are moving to production and are not ready to self-host.

#### Using the local setup

1. Docker installecd locally
2. Node.js if you cannot install Docker

#### Using the cloud setup

1. A [Directus Cloud](https://directus.cloud) account

## Setting up Directus: Locally or in the Cloud

Before proceeding with the component connections, you need to set up Directus to be ready to receive your sensor data. You can set it up locally or through the Directus cloud. The local approach frees you from a free trial or associated costs, but it means you have to purchase and manage your infrastructure when taking it to production. The cloud approach gives you a production-ready environment right from the start. It also eases the setup stress. Choose whichever approach best suits your requirements.

### Setting up Directus Locally

You can set up Directus locally using Docker or npm (Node.js). The Docker approach is simpler because you only run a single command to get Directus running. The command below pulls the Directus Docker image and starts a Directus Docker container on port 8055. The admin details (email and password) alongside the key and secret that Directus uses internally are provided as environment variables. A Docker volume is also mapped from the directory where this command is run to the `/directus/database` directory that contains the SQLite database file that will power Directus locally. So the Directory you run this from will have a `database.sqlite` file after you run the command. Go ahead and run the command below:

```
docker run \
-p 8055:8055 \
-e KEY=replace-with-random-value \
-e SECRET=replace-with-random-value \
-e ADMIN_EMAIL="admin@example.com" \
-e ADMIN_PASSWORD="password" \
-v "$(pwd)":/directus/database \
directus/directus
```

With the Directus server up and running, you should visit http://localhost:8085 to see the Directus dashboard.

![Directus Dashboard](https://github.com/vicradon/directus-guest-authoring/assets/40396070/b92ae6b9-89e8-4f4f-a869-1def773f9cbc)

If you do not want to set up locally, you can easily set up on [Directus cloud](https://directus.cloud/). You can sign in using your Github account or via email. Follow this [guide to set up Directus cloud](https://docs.directus.io/getting-started/quickstart.html).

## Creating the `temperature_and_humidity` Collection

After setting up Directus, you must create the database table where your IoT data will be stored. In Directus, you can create new "tables" for your database by creating a collection. A collection serves a dual purpose. A table for interacting with Directus via API (or SDK) and a way to configure CMS operations.

On the `/admin/content` page, click on the **Create Collection** button that is at the center of the page. This operation opens up a sidebar with a menu for inputting details about the collection.

Set the new collection’s name as `temperature_and_humidity`, then click the `right arrow` button to proceed.

On the next page, you will have the option to select pre-defined fields you would like to include in your collection. Only select the **Created On (date_created)** field, then click the `right arrow` button to finish the setup.

With all this done, you will now have a database table that can be queried via the API. But to collect usable data, you first have to create the columns that will contain the temperature and humidity data. Directus refers to columns are fields because they are more than database columns. It allows you to define how the CMS displays input for them, hence, they are referred to as **field**. So on the temperature_and_humidity collection’s data model (/admin/settings/data-model/temperature_and_humidity), click on the **Create Field** button.

Clicking the **Create Field** button opens up a sidebar with a form containing the details of the new field. Select **Input** as the field’s input type, then set its **Key** as **temperature** and the **type (datatype)** as **Float**.

Scroll through the sidebar until you see a save button, then click on it. 

Repeat the process for humidity, selecting the field type as **Input**, the key as **Humidity**, and the type (datatype) as **Float**. After both fields are created, you will have your collection ready to receive data from your IoT setup!

## Creating a Directus Role and User

You need a role that has permission to create and read records on the temperature_and_humidity table. When you create this role, you'll need to create a user and assign this role to them. To create the role, navigate to settings > access control page. On this page, click on the **Create Role** button that resides at the top right corner.

Clicking on this button opens up a modal where you put in the details of the role. Set the role name as esp32-writer and ensure that **App Access** is enabled. Proceed by clicking the **Save** button.

Clicking **Save** takes you to the next page where you can configure the details of this role. On the role configuration page, Give the **ESP32-Writer** permission to create new items in the temperature_and_humidity table. So under the plus icon, click on the red **No Access** button, and change the permission to **All Access**. The changes are saved automatically so you don’t need to save anything.

Now that this role has been created, create a user that will be assigned the role by scrolling to the end of the **ESP32-Writer** role page and clicking on the **Create New** button under the **Users in Role** section.

Clicking the **Create New** button opens a sidebar with a form containing the new user’s details. Set the First Name as **ESP32-Board*

Before saving the user’s details, scroll to the end of the sidebar to find the **Generate Token** button under admin options. Clicking on this button generates an authentication token you can use to create new data in your collection in an authorized manner.

Proceed by clicking on the **Generate Token** button, copying the generated token, and saving everything using the save checkmark button at the top right corner. You should save the generated token in a safe place, like a temporary notepad. Finally, complete the role and user creation by clicking the Save button at the top right corner.

With that done, you should have a new role, ESP32-Writer, with a single user.

## Creating dummy temperature and humidity values

You can immediately populate your temperature_and_humidity table by making a POST request. This POST request will have an Authorization header with the token inputed as a Bearer token. The snippet below shows an example post request using cURL. To run this request, replace the `<YOUR_TOKEN>` with the token generated in the previous section and then run it in a UNIX shell or Powershell.

```
curl --location 'http://localhost:8055/items/temperature_and_humidity' \
--header 'Authorization: Bearer Vi8m1wdXTEv0rhQtteUWaZKfHTqCOwDx' \
--header 'Content-Type: application/json' \
--data '{"temperature": 33.34,"humidity": 80.42}'
```

After running the curl command above, you will see a new value in the collection on your Directus dashboard (/admin/content/temperature_and_humidity). 


You can run the cURL command with different values of temperature and humidity to see more data in your collection, but first, take a moment to look at the URL used in the cURL command above. It is your Directus root URL at localhost:8055 with a path /items/temperature_and_humidity. Directus gives a straightforward way to interact with collections via a REST API by appending /items to the root URL and then the collection name. You can perform API operations using the standard REST principles such as:

- GET /items/temperature_and_humidity
- POST /items/temperature_and_humidity
- PATCH /items/temperature_and_humidity
- DELETE /items/temperature_and_humidity

## Connecting the ESP32 and the DHT22 with Physical Components

A DHT22 sensor can connect directly to an ESP32 using three pins. DHT22 comes in two types, 3-pin type and 4-pin type. The 3-pin type doesn't require extra configuration. You simply connect ground to ground, VCC to 5V output, and data to a GPIO pin, say pin 13. For the 4-pin type, simply ignore the 3rd pin from the left and connect as shown in the image below:

![DHT22 to ESP32](https://github.com/vicradon/directus-guest-authoring/assets/40396070/f60a19c3-c728-4a59-ba1d-7e4a1f6a5865)




You can program your ESP32 using the Arudino IDE. You can follow the steps in this article.

## Installing DHT22 sensor library

You can use the DHT library from Adafruit to interface your DHT22 with ESP32. Simply search it up on the libraries section and install.

### Logging temperature and humidity data to Serial
You can see the values from the DHT22 sensor in the Arduino serial monitor. 

## Sending the temperature and humidity data to Directus

#include <WiFi.h>
#include <HTTPClient.h>


const char* ssid = "Vicradon-";
const char* password = "bosslady";
const char* directusToken = "Bearer Vi8m1wdXTEv0rhQtteUWaZKfHTqCOwDx";
const char* directusEndpoint = "http://192.168.43.143:8055/items/temperature_and_humidity";
// const char* directusEndpoint = "https://webhook.site/a004f461-5f53-450c-8d86-bc39d3ecb721";


float temperature, humidity;






void setup() {
 Serial.begin(115200);
 delay(4000);


 WiFi.mode(WIFI_STA);
 WiFi.begin(ssid, password);
 Serial.println("\nConnecting to WiFi");
 while (WiFi.status() != WL_CONNECTED) {
   delay(500);
   Serial.print(".");
 }
 Serial.println("Connected to WiFi");
}


void loop() {
 if (WiFi.status() == WL_CONNECTED) {  //Check WiFi connection status


   temperature = 30.4;
   humidity = 76.2;


   HTTPClient http;
   http.begin(directusEndpoint);
   http.addHeader("Content-Type", "application/json");
   http.addHeader("Authorization", directusToken);


   String jsonPayload = "{\"temperature\":\"" + String(temperature) + "\",\"humidity\":\"" + String(humidity) + "\"}";


   // String jsonPayload = "{\"temperature\": 23.4, \"humidity\": 34.2}";




   Serial.println(jsonPayload);
   // Send POST request
   int httpResponseCode = http.POST(jsonPayload);


   if (httpResponseCode > 0) {
     Serial.print("HTTP Response code: ");
     Serial.println(httpResponseCode);


     String response = http.getString();
     Serial.println(response);
   } else {
     Serial.print("Error in HTTP POST request: ");
     Serial.println(httpResponseCode);
   }


   http.end();




   Serial.print("Temperature: ");
   Serial.print(temperature);
   Serial.print("°C <-> Humidity: ");
   Serial.print(humidity);
   Serial.println("%");


 } else {
   Serial.println("Error in WiFi connection");
 }


 delay(30000);
}
```


Here’s the complete code for your use:

```
#include <WiFi.h>
#include <HTTPClient.h>
#include <Arduino.h>
#include <DHT.h>


#define USE_SERIAL Serial




const char* ssid = "Vicradon-Android";
const char* password = "bosslady";
const char* directusToken = "Bearer HKoo58BuCS2CwcttuKSYodkeqcY4beHn";
const char* directusEndpoint = "http://192.168.43.143:8055/items/temperature_and_humidity";


float temperature, humidity;


DHT dht22_sensor(13, DHT22);


void setup() {
 Serial.begin(115200);
 delay(4000);


 WiFi.mode(WIFI_STA);
 WiFi.begin(ssid, password);
 Serial.println("\nConnecting to WiFi");
 while (WiFi.status() != WL_CONNECTED) {
   delay(500);
   Serial.print(".");
 }
 Serial.println("Connected to WiFi");


 dht22_sensor.begin();
}


void loop() {
 if (WiFi.status() == WL_CONNECTED) {  //Check WiFi connection status


   temperature = dht22_sensor.readTemperature();
   humidity = dht22_sensor.readHumidity();


   if (!isnan(temperature) && !isnan(humidity)) {
     HTTPClient http;
     http.begin(directusEndpoint);
     http.addHeader("Content-Type", "application/json");
     http.addHeader("Authorization", directusToken);


     String jsonPayload = "{\"temperature\":" + String(temperature) + ",\"humidity\":" + String(humidity) + "}";


     Serial.println(jsonPayload);
     // Send POST request
     int httpResponseCode = http.POST(jsonPayload);


     if (httpResponseCode > 0) {
       Serial.print("HTTP Response code: ");
       Serial.println(httpResponseCode);


       String response = http.getString();
       Serial.println(response);
     } else {
       USE_SERIAL.printf("[HTTP] POST... failed, error: %s\n", http.errorToString(httpResponseCode).c_str());
     }


     http.end();




     Serial.print("Temperature: ");
     Serial.print(temperature);
     Serial.print("°C <-> Humidity: ");
     Serial.print(humidity);
     Serial.println("%");


   } else {
     Serial.println("Error reading sensor data");
   }


 } else {
   Serial.println("Error in WiFi connection");
 }


 delay(30000);
}
```

## Connecting the ESP32 to DHT22 on Wokwi
To use the virtual simulation mode to carry out this tutorial, you need a Wokwi club account. This is necessary because the Wifi feature is a premium feature on Wokwi.

## Sending temperature and humidity data to Directus from Wokwi

## Conclusion
In this tutorial you learned how to collect temperature and humidity data from a DHT22 sensor and log it to a database with the aid of Directus. You learned how to visualize how this data changes over time using Directus dashboards.

Directus presents a complete BaaS solution allowing you to build content-focused and traditional web applications. It offers a plethora of services including database mirroring, user authentication, OAuth2, and HTTP APIs over data (REST and GraphQL). You can self-host Directus or use their cloud API with reasonable pricing. [Try Directus today](https://directus.cloud/)!
