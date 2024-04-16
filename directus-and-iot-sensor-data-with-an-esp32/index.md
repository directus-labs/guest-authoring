---
title: "Directus and IoT: Sensor Data with an ESP32"
description: "IoT applications can write data to Directus and have this data visualized through Directus Dashboards. This is achieved through the Directus API and an ESP32."
author:
  name: "Osinachi Chukwujama"
  avatar_file_name: "osiheadshot.png"
---

## Introduction

When building an IoT application, you often need to collect and analyze data to control your system.

## Before You Start

## Setting up Directus: Locally or in the Cloud

You can set up Directus locally using Docker or npm (Node.js). Using the Docker approach, you run the command below to set it up:

```
docker run \
-p 8055:8055 \
-e KEY=replace-with-random-value \
-e SECRET=replace-with-random-value \
directus/directus
```

With the Directus server up and running, you can go ahead to visit http://localhost:8085 to see the Directus dashboard.

## Creating the `temperature_and_humidity` Collection

In Directus, you can create new "tables" for your database by creating a collection. A collection serves a dual purpose. A table for interacting with Directus via API (or SDK) and a way to configure CMS operations.

On the `/admin/content` page, click on the **Create Collection** button

## Summary
