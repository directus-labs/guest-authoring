---
title: 'Build a Video streaming app with Directus and Sveltekit #199'
description: 'Learn how to integrate Directus with SvelteKit. You will store, and retrieve video metadata in Directus CMS and use them to build a video streaming application.'
author:
  name: 'Clara Ekekenta'
  avatar_file_name: './clara-ekekenta.avif'
---

In this tutorial, you will learn how to build an application using Directus as a backend. You will store and retrieve video metadata in Directus CMS and use them to build a video streaming application that tracks views.

## Before You Start

You will need:

- [Node.js v20.11.1](https://nodejs.org/) or later.
- A code editor on your computer.
- A Directus project - follow our [quickstart guide](https://docs.directus.io/getting-started/quickstart) if you don't already have one.
- Some knowledge of SvelteKit.

You can find the code for this tutorial on my [GitHub repository](https://github.com/Claradev32/directus_sveltekit).

## Creating a Svelte Project

Setting up a new Sveltekit project and install the required dependencies including the Directus SDK:

```
npm create svelte@latest video-streaming-app #
cd video-streaming-app
npm install
npm install @directus/sdk svelte-video-player
```

In your `src/libs` folder, create a new `directus.ts` file to create a Directus SDK instance helper function:

```js
import { createDirectus, rest } from "@directus/sdk";
export const DIRECTUS_API_URL = import.meta.env.VITE_DIRECTUS_URL;
function getDirectusClient() {
  const directus = createDirectus(DIRECTUS_API_URL).with(rest());
  return directus;
};
export default getDirectusClient;
```

Create a `.env` file in the root folder of your project and add your Directus API URL:

```
VITE_DIRECTUS_URL='https://directus.example.com';
```

## Creating a Directus Collection

Create a new `videos` collection with the following fields:

- `id` (Primary Key Field, Type: Manually entered string)
- `title` (Type: String, Interface: Input)
- `description` (Type: String, Interface: Input)
- `thumbnail` (Type: Image Field)
- `video_file` (Type: File Field)
- `tags`  (Type: Tags Field)
- `views` (Type: String, Interface: Input)
- `upload_date` (Type: Datetime Field)

Give the Public role read access on the `videos` and `directus_files` collections.

Create 3 videos in the collection to test with.

## Creating the Video Listing Page

In your SvelteKit application, create a `types.ts` file in the `src/lib` folder to define the structure of your video data and ensure type safety throughout your application.

```ts
export interface Video {
  id: string;
  title: string;
  description: string;
  video_file:  { id: string };
  thumbnail: { id: string };
  tags: string[];
  duration: number;
  views: number;
  upload_date: string;
}
```

Create a new `components` folder inside the `src/libs` folder. Within this new folder, create two component files: `VideoCard.svelte` and `VideoGrid.svelte`. Add the following code snippet to the `VideoCard.svelte` file:

```svelte
<script lang="ts">
  import type { Video } from "$lib/types";
  export let video: Video;
</script>

<a href="/video/{video.id}" class="video-card">
  <img
    src={`${import.meta.env.VITE_DIRECTUS_URL}/assets/${video.thumbnail.id}`}
    alt={video.title}
  />
  <h3>{video.title}</h3>
  <p>
    {video.views} views • {new Date(video.upload_date).toLocaleDateString()}
  </p>
</a>

<style>
  .video-card { display: block; text-decoration: none; color: inherit; }
  img { width: 100%; height: auto; }
</style>
```
Then add the following code to the `VideoGrid.svelte` file:

```svelte
<script lang="ts">
  import type { Video } from "$lib/types";
  import VideoCard from "./VideoCard.svelte";
  export let videos: Video[];
</script>

<div class="video-grid">
  {#each videos as video}
    <VideoCard {video} />
  {/each}
</div>

<style>
  .video-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 1rem; }
</style>
```

Those two files are reusable components created to render your video data and organize the videos in a grid layout.

## Fetching video data
In your `src/libs folder`, create a new folder named services. Inside this folder, create a file named `index.ts`. Add the following code to this file to use the Directus helper function to fetch all the videos from your Directus `videos` collection:

```ts
import getDirectusClient from "$lib/directus";
import { readItems } from "@directus/sdk";
import type { Video } from "$lib/types";

export async function getVideos(params = {}): Promise<Video[]> {
  const directus = getDirectusClient();
  const response = await directus.request(readItems("videos", params));
  return response as Video[];
}
```

## Displaying thumbnails and titles
Update the your `routes/+page.svelte` file to use the `getVideos` function to fetch video data and display it using the `VideoGrid` component. This will display the thumbnails, titles, views and dates of the videos.
```svelte
+
<script lang="ts">
  import { onMount } from "svelte";
  import { getVideos } from "$lib/services/index";
  import VideoGrid from "$lib/components/VideoGrid.svelte";
  import type { Video } from "$lib/types";

  let videos: Video[] = [];

  onMount(async () => {
    try {
      videos = await getVideos({
        sort: ["-upload_date"],
        limit: 20,
        fields: ["*", "thumbnail.*", "video_file.*"],
      });
    } catch (error) {
      console.error("Error fetching videos:", error);
    }
  });
</script>

<h1>Stream your favourite vidoes</h1>


{#if videos.length > 0}
  <VideoGrid {videos} />
{:else}
  <p>Loading videos...</p>
{/if}
```
Directus stores file metadata in the `directus_files` collection. When using file fields in other collections, Directus creates a one-to-many relationship. To include file data when fetching items, you use dot notation in the fields parameter, like `thumbnail.*` and `video_file.*`. This tells Directus to populate the file information from the `directus_files` collection.

![Video Listing](<Screenshot 2024-07-04 at 11.53.25.png>)

## Video Player Page
Install the Svelte video player component to play the videos with the command.

```shell
npm install svelte-video-player
```

Update your `services/index.ts` file to add new functions that will fetch a video by its ID and update the `videos` collection to increment the video's views field.
```ts
+
// your other imports
import { readItems, readItem, updateItem } from "@directus/sdk";

export async function getVideo(id: string): Promise<Video> {
  const directus = getDirectusClient();
  const response = await directus.request(
    readItem("videos", id, {
      fields: ["*", "thumbnail.*", "video_file.*"],
    })
  );
  return response as Video;
}

export async function incrementViews(id: string) {
  const directus = getDirectusClient();
  const video = await directus.request(readItem("videos", id));
  await directus.request(
    updateItem("videos", id, { views: parseInt(video.views || 0) + 1 })
  );
}
```

Create a nested route in your **routes** folder in the format `video/[id]/+page.svelte` to create a page to play selected videos. Update this file with the following code:

```svelte
<script lang="ts">
  import { page } from "$app/stores";
  import { getVideo, incrementViews } from "$lib/services";
  import VideoPlayer from "svelte-video-player";
  import type { Video } from "$lib/types";

  let video: Video | null = null;

  $: id = $page.params.id;
  $: if (id) {
    getVideo(id)
      .then((v: Video) => {
        video = v;
        incrementViews(id);
      })
      .catch((error) => {
        console.error("Error fetching video:", error);
      });
  }
</script>

{#if video}
  <h1>{video.title}</h1>
  <p>
    {video.views} views • {new Date(video.upload_date).toLocaleDateString()}
  </p>
  <VideoPlayer
    poster={`${import.meta.env.VITE_DIRECTUS_URL}/assets/${video.thumbnail.id}`}
    source={`${import.meta.env.VITE_DIRECTUS_URL}/assets/${video.video_file.id}`}
  />
  <h2>Description</h2>
  <p>{video.description}</p>
  <h2>Tags</h2>
  <div class="tags">
    {#each video.tags as tag}
      <span class="tag">{tag}</span>
    {/each}
  </div>
{:else}
  <p>Loading...</p>
{/if}

<style>
  .tags {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
  }
  .tag {
    background-color: #eee;
    padding: 0.25rem 0.5rem;
    border-radius: 0.25rem;
  }
</style>
```
Now click on any of the videos to stream it. 

![Video Player](<Screenshot 2024-07-04 at 12.27.56.png>)

## Search Functionality
In your `services/index.ts`, add a new funtion that implements search functionality to find videos by title or description.

```ts
+
export async function searchVideos(query: string): Promise<Video[]> {
  const directus = getDirectusClient();
  const response = await directus.request(
    readItems("videos", {
      search: query,
      fields: ["*", "thumbnail.*", "video_file.*"],
    })
  );
  return response as Video[];
}
```
This function uses the `search` parameter from Directus to perform a search on `videos` collection.

Update the code in your `routes/+page.svelte` file to use the `searchVideos` function to add search functionality to your page.

```svelte
+
<script lang="ts">
  import { onMount } from "svelte";
  import { getVideos, searchVideos } from "$lib/services/index";
  import VideoGrid from "$lib/components/VideoGrid.svelte";
  import type { Video } from "$lib/types";

  let videos: Video[] = [];
  let searchQuery = "";
  let searchResults: Video[] = [];
  let isSearching = false;

  onMount(async () => {
    await loadLatestVideos();
  });

  async function loadLatestVideos() {
    try {
      videos = (await getVideos({
        sort: ["-upload_date"],
        limit: 20,
        fields: ["*", "thumbnail.*", "video_file.*"],
      })) as Video[];
    } catch (error) {
      console.error("Error fetching videos:", error);
    }
  }

  async function handleSearch() {
    if (searchQuery.trim()) {
      isSearching = true;
      try {
        const response = await searchVideos(searchQuery);
        searchResults = response as Video[];
      } catch (error) {
        console.error("Error searching videos:", error);
      } finally {
        isSearching = false;
      }
    } else {
      searchResults = [];
    }
  }
</script>

<h1>Stream your favourite vidoes</h1>

<form on:submit|preventDefault={handleSearch}>
  <input type="text" bind:value={searchQuery} placeholder="Search for videos" />
  <button type="submit">Search</button>
</form>

{#if isSearching}
  <p>Searching...</p>
{:else if searchResults.length > 0}
  <h2>Search Results</h2>
  <VideoGrid videos={searchResults} />
{:else if searchQuery}
  <p>No results found.</p>
{:else}
  <h2>Latest Videos</h2>
  {#if videos.length > 0}
    <VideoGrid {videos} />
  {:else}
    <p>Loading videos...</p>
  {/if}
{/if}
```

You can now search and stream any video of your choice.
![Search videos](<Screenshot 2024-07-04 at 12.48.17.png>)

## Summary
In this tutorial, you've learned how to build a Sveltekit video streaming application that uses data from a Directus project. You started by creating a new project, set up environment variables and everything you need to call Directus. You then created a videos collections in Directus and integrated them with the your Sveltekit project.

