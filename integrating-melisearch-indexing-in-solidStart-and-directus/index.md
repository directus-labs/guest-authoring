---
title: 'Integrating Melisearch Indexing with Directus'
description: 'Learn how to maintain a Melisearch index from data in your Directus project by building a custom hook extension.'
author:
  name: 'Clara Ekekenta'
  avatar_file_name: './clara-ekekenta.avif'
---

In this tutorial, you will learn how to integrate Meilisearch indexing with a SolidStart frontend and a Directus backend. You'll focus on setting up Directus, creating a SolidStart project, and implementing Meilisearch indexing to keep your search data up-to-date.


## Setting Up Directus

You will need to have a [local Directus project running](https://docs.directus.io/self-hosted/quickstart) to develop extensions. 

In your new project, create a collection called `articles` with a `title`, `content`, and `author` field.

## Initializing Directus Extension
In your `docker-compose.yml`` file, add the environment variable EXTENSIONS_AUTO_RELOAD: "true"` to enable automatic extension reloading.

Navigate to your extensions directory and run:

```shell
npx create-directus-extension@latest
```
Name your extension `meilisearch-indexing`, choose the `hook` type, and create it with `JavaScript`. Allow Directus to install dependencies automatically.

## Setting Up Meilisearch
Sign up for a Meilisearch account if you haven't already. Once you have your Meilisearch instance details, you will be able to copy your credentials in your dashboard. 

![Melisearch dashboard](<Screenshot 2024-07-11 at 06.45.38.png>)

Copy the URL and API key to your `docker-compose.yml` file:

```docker-compose
MEILISEARCH_HOST=your_meilisearch_host
MEILISEARCH_API_KEY=your_meilisearch_api_key
```

Restart your Directus project to apply the new environment variables.

In your **extension** directory, install the Meilisearch client:

```
npm install meilisearch
```

Run `npm run dev` to start the automatic extension building.

In your extension's `src/index.js` file, initialize the Meilisearch client:

```javascript
import { MeiliSearch } from 'meilisearch'

const client = new MeiliSearch({
  host: process.env.MEILISEARCH_HOST,
  apiKey: process.env.MEILISEARCH_API_KEY
})
const index = client.index('articles')
```

## Writing Directus hooks to send data to Meilisearch
Use Directus hooks to automatically update your Meilisearch index when content is created, changed, or deleted in Directus.

### On data creation
Update your extension's exported function to process create events when a new `article` is added to the collection:

```javascript
export default ({ action }) => {
  action('articles.items.create', async (meta) => {
    await index.addDocuments([{ id: meta.key, ...meta.payload }])
  })
}
```
The `articles.items.create` action hook triggers after item creation. The `meta` object contains the new item's key (ID) and other fields in its payload property. By setting the `objectID` to the Directus item id, we ensure accurate referencing and management in Meilisearch.

### On data update
Add another action hook to process updates when one or more articles are modified:
```javascript
export default ({ action }) => {
  //...
  action('articles.items.update', async (meta) => {
    await Promise.all(
      meta.keys.map(async (key) => 
        await index.updateDocuments([{ id: key, ...meta.payload }])
      )
    )
  })
}
```
The `articles.items.update` action hook triggers when articles are updated. It receives `meta.keys` (an array of updated item IDs) and `meta.payload` (changed values). The hook updates each modified document in Meilisearch using `Promise.all` for efficient processing.

### On data deletion

Add an action hook to remove items from Meilisearch when they're deleted in Directus:

```javascript
export default ({ action }) => {
  //...
   action('articles.items.delete', async (meta) => {
    await index.deleteDocuments(meta.keys)
  })
}
```
The `articles.items.delete` action hook triggers when articles are deleted. It receives `meta.keys`, an array of deleted item IDs. The hook uses these keys to remove the corresponding documents from the Meilisearch index.

Now add 3 items to your articles collection.

![Melisearch with data from Directus](<Screenshot 2024-07-11 at 06.58.04.png>)


## Summary
In this tutorial, you've learned how to integrate Meilisearch indexing with Directus and SolidStart. You've learned how to setup the Directus hooks that automatically indexes data created, updated, or deleted from a Directus project in Meilisearch.