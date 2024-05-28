---
title: "Directus and IoT: Sensor Data with an ESP32"
description: "IoT applications can write data to Directus and have this data visualized through Directus Dashboards. This is achieved through the Directus API and an ESP32."
author:
  name: "Osinachi Chukwujama"
  avatar_file_name: "osiheadshot.png"
---

# Directus and IoT: Sensor Data with an ESP32

IoT systems will not be complete without logging their collected data to remote cloud storage. In this tutorial, you will build a simple temperature logging system using a DHT22 as a temperature and humidity module, and an ESP32 as the microcontroller board with onboard WiFi for connecting to a Directus project.

## Before You Start

You will need:

- A Directus project - [follow our quickstart guide](https://docs.directus.io/getting-started/quickstart) if you don't already have one.
- Either the list of physical components below, or a [A Wowki Club account](https://wokwi.com/club) that will allow you to simulate the hardware.

### Components List

- An [ESP32](https://www.espressif.com/en/products/socs/esp32) development board.
- A [DHT22 sensor](https://www.adafruit.com/product/385). 
- A Type B Micro USB cable.
- Three male to female jumper cables (may be optional, depending on the configuration of your DHT22).

You will also need to install the [Arduino IDE](https://www.arduino.cc/en/software) and have ESP32 board support. Follow the official [Espressif documentation](https://docs.espressif.com/projects/arduino-esp32/en/latest/installing.html) to install it in the IDE.

## Creating the `temperature_and_humidity` Collection

After setting up Directus, you must create the database table where your IoT data will be stored. Create a new `temperature_and_humidity` collection and enable the **Created On (date_created)** optional field.

Create the following additional fields:
- `temperature` - input interface - `float` type.
- `humidity` - input interface - `float` type.

## Creating a Directus Role and User

Create a new role called `esp32-writer` and give **All Access** to the `temperature_and_humidity` collection.

Create a new user in this role called "ESP32-Writer" and generate a static access token. Save this for later.

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

A DHT22 sensor can connect directly to an ESP32 using three pins. DHT22 comes in two types, 3-pin type and 4-pin type. The 3-pin type doesn't require extra configuration. You connect ground to ground, VCC to 5V output, and data to a GPIO pin, say pin 13. For the 4-pin type, ignore the 3rd pin from the left and connect as shown in the image below:

![DHT22 to ESP32](./DHT22_to_ESP32.png)

### Installing DHT22 Sensor Library

Since you will program your ESP32 using the Arduino IDE, you must install the [DHT sensor library by Adafruit](https://www.arduino.cc/reference/en/libraries/dht-sensor-library/). Search for the "DHT Sensor Library" in your library manager and install the corresponding library authored by Adafruit. Use the image below as a reference.

![Installing the DHT22 sensor library](./Installing_the_DHT22_sensor_library.png)

### Connecting the ESP32 Board to your Computer

You can see the values from the DHT22 sensor in the Arduino serial monitor. After connecting your ESP32 to your computer, choose a board and port that corresponds to your purchased board and available port on your computer.

![Board selector page](./Board_selector_page.png)

If you are using the ESP32 Wroom 32D, choose the ESP 32 DA Module and the COM port that appears after you plug in the ESP32 to your computer via the USB cable.

![Selecting board and port](./Selecting_board_and_port.png)

### Logging temperature and humidity data to Serial

You can log the values from the DHT22 to the serial monitor by defining variables for the temperature and humidity and then initializing the DHT object. Within the setup function, you must initialize the Serial logging and intialize the connection to the DHT22 module. 

Within the loop function, the sensor readings are obtained from the DHT22 and stored to the temperature and humidity variables. With all that done, these values can be logged to the serial monitor.

There's a delay of 5 seconds to ensure that the DHT22 can handle accurate readings as it has a low sampling rate. When sending your data to Directus, you will increase the delay to 30 seconds or greater. Your Serial Monitor baud rate must be set to 115200 for you to see the values being logged.

```cpp
#include <DHT.h>

float temperature, humidity;
DHT dht22_sensor(13, DHT22);

void setup() {
  Serial.begin(115200);
  dht22_sensor.begin();
}

void loop() {
  temperature = dht22_sensor.readTemperature();
  humidity = dht22_sensor.readHumidity();

  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.print("°C <-> Humidity: ");
  Serial.print(humidity);
  Serial.println("%");

  delay(5000);
}
```

## Sending the temperature and humidity data to Directus

At this point, you have your ESP32 logging data to the Serial monitor. But you actually want to send this data to Directus. You have to introduce the HTTP and WiFi libraries to achieve this. The WiFi library connects your ESP32 to the internet while the HTTP library turns your ESP32 into an HTTP agent that can make GET and POST requests among others. The script below is the complete code for logging data to Directus. Here are the things you must change for the script to work:

1. Your WiFI SSID, i.e. the name of your WiFi network, as the value of the `ssid` variable on line 6.
2. Your WiFi password on line 7.
3. Your Directus esp32-board user token that you used with cURL at the `Creating dummy temperature and humidity values` section of this article. You can still [regenerate the token if you lost it](./regenerate_lost_token.png). If you have the token, set it as the <TOKEN> placeholder value on line 8.
4. Your WiFi gateway address is defined on line 9 as `directusEndpoint`. It won't start with localhost because the ESP32 is running as a separate system. So you must check the gateway address of your local network. This address will either start with 10, 172, or 192. Follow [this blog post](https://nordvpn.com/blog/find-router-ip-address/) for instructions for checking your gateway address on different operating systems. Note that your ESP32 and your computer running Directus must be connected to the same WiFi network for this to work.

With all the changes made, you can upload your script to your ESP32 and observe it log temperature and humidity data on your Directus `temperature_and_humidity` collection.

```cpp
#include <WiFi.h>
#include <HTTPClient.h>
#include <Arduino.h>
#include <DHT.h>

const char* ssid = "<YOUR_WIFI_SSID>";
const char* password = "<YOUR_WIFI_PASSWORD>";
const char* directusToken = "Bearer <TOKEN>";
const char* directusEndpoint = "http://192.168.43.143:8055/items/temperature_and_humidity";

float temperature, humidity;
DHT dht22_sensor(13, DHT22);

void setup() {
 Serial.begin(115200);
 delay(1000);

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
 if (WiFi.status() == WL_CONNECTED) {
   temperature = dht22_sensor.readTemperature();
   humidity = dht22_sensor.readHumidity();

   if (!isnan(temperature) && !isnan(humidity)) {
     HTTPClient http;
     http.begin(directusEndpoint);
     http.addHeader("Content-Type", "application/json");
     http.addHeader("Authorization", directusToken);

     String jsonPayload = "{\"temperature\":" + String(temperature) + ",\"humidity\":" + String(humidity) + "}";

     Serial.println(jsonPayload);

     int httpResponseCode = http.POST(jsonPayload);

     if (httpResponseCode > 0) {
       Serial.print("HTTP Response code: ");
       Serial.println(httpResponseCode);

       String response = http.getString();
       Serial.println(response);
     } else {
       Serial.printf("[HTTP] POST... failed, error: %s\n", http.errorToString(httpResponseCode).c_str());
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

When you open your Directus content section, you will see the values logged so far. You can increase the delay to reduce the number of logs you get per hour. Currently, the rate is 2 logs per minute (1 log every 30 seconds), so 120 logs per hour. This may or may not work for you depending on your use case.

![Dashboard with logs](./Dashboard_with_logs.png)

## Visualizing Data in Directus Insights

Directus Insights allows you to create multiple panels in a dashboard, powered by data in your project.

To show the change over time for temperature, create a bar chart with the following settings:

1. Collection - Temperature and Humidity
2. X-Axis - Date Created
3. Y-Axis - Temperature
4. Y-Axis Function - Max
5. Value Decimals - 2
6. Color - #E35168

You can repeat this for humidity, and any other data inside of your project. 

![temperature and humidity trends over time](./temperature_and_humidity_trends_over_time.png)

## Summary

In this tutorial, you learned how to collect temperature and humidity data from a DHT22 sensor and log it to a Directus project. You also learned how to visualize how this data changes over time using Directus Insights.
