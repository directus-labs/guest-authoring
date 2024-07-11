---
title: 'Integrating Melisearch Indexing in SolidStart and Directus'
description: 'Learn how to integrate Meilisearch indexing with a SolidStart frontend and a Directus backend. You will set up Directus, create a SolidStart project, and implement Meilisearch indexing to keep your search data up-to-date.'
author:
  name: 'Clara Ekekenta'
  avatar_file_name: './clara-ekekenta.avif'
---

In this tutorial, you will learn how to integrate Meilisearch indexing with a SolidStart frontend and a Directus backend. You'll focus on setting up Directus, creating a SolidStart project, and implementing Meilisearch indexing to keep your search data up-to-date.

## Setting Up Directus
First, ensure you have a local Directus project running. Once your project is set up, create a collection called `articles` with fields such as `title`, `content`, and `author`.

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

## Setting Up SolidStart
Create a new SolidStart project with command:
```shell
npx create-solid@latest
```

Choose the `SolidStart` option and your preferred styling method.

Install the Meilisearch client in your SolidStart project:
```shell
npm install meilisearch
```

Create a new file `src/lib/meilisearch.js` and add the following code snippets:

```javascript
import { MeiliSearch } from 'meilisearch'

export const searchClient = new MeiliSearch({
  host: import.meta.env.VITE_MEILISEARCH_HOST,
  apiKey: import.meta.env.VITE_MEILISEARCH_API_KEY
})
```
Create a `.env` file in your SolidStart project root and add your Meilisearch credentials:
```env
VITE_MEILISEARCH_HOST=your_meilisearch_host
VITE_MEILISEARCH_API_KEY=your_meilisearch_api_key
```

## Performing Search Queries
Create a new component `src/components/ArticleSearch.jsx` and add the following code:

```jsx
import { createSignal, createResource } from 'solid-js'
import { searchClient } from '../lib/meilisearch'

export default function ArticleSearch() {
  const [query, setQuery] = createSignal('')
  const [results] = createResource(query, searchArticles)

  async function searchArticles(q) {
    if (!q) return []
    const { hits } = await searchClient.index('articles').search(q)
    return hits
  }

  return (
    <div>
      <input
        type="text"
        value={query()}
        onInput={(e) => setQuery(e.target.value)}
        placeholder="Search articles..."
      />
      <ul>
        {results()?.map((article) => (
          <li key={article.id}>{article.title}</li>
        ))}
      </ul>
    </div>
  )
}
```
Now you can use this `ArticleSearch` component in your SolidStart pages to provide real-time search functionality.

## Handling Advanced Search Features
Implement advanced search features in your Meilisearch index, including filtering, faceted search, and typo tolerance.
### Implementing filtering and sorting options
Update your extension's exported functions to implement filtering and sorting:

```javascript
//...
let isIndexConfigured = false

async function configureIndex() {
  if (!isIndexConfigured) {
    await index.updateSettings({
      filterableAttributes: ['category', 'tags'],
      sortableAttributes: ['date'],
      searchableAttributes: ['title', 'content', 'author', 'category', 'tags']
    })
    isIndexConfigured = true
    console.log('Meilisearch index configured successfully')
  }
}

export default ({ action, init }) => {
  init('app.start', async () => {
    await configureIndex()
  })

  const ensureIndexConfigured = async (callback) => {
    await configureIndex()
    return callback()
  }

  action('articles.items.create', async (meta) => {
    await ensureIndexConfigured(async () => {
      await index.addDocuments([{ 
        id: meta.key, 
        ...meta.payload
      }])
    })
  })

  action('articles.items.update', async (meta) => {
    await ensureIndexConfigured(async () => {
      await Promise.all(
        meta.keys.map(async (key) => 
          await index.updateDocuments([{ 
            id: key, 
            ...meta.payload
          }])
        )
      )
    })
  })

  action('articles.items.delete', async (meta) => {
    await ensureIndexConfigured(async () => {
      await index.deleteDocuments(meta.keys)
    })
  })
}
```
In the above code:
- The `configureIndex()` function sets up the Meilisearch index with specific attributes for filtering, sorting, and searching.
- It uses a flag `isIndexConfigured` to ensure the configuration runs only once.
- The `init('app.start', ...)` hook ensures the index is configured when the app starts.
- `ensureIndexConfigured()` wraps operations to guarantee the index is set up before performing actions.

Update your `src/components/ArticleSearch.jsx` file to add the filtering and sorting to your app.

```jsx
import { createSignal, createResource, For } from 'solid-js'
import { searchClient } from '../lib/meilisearch'

export default function ArticleSearch() {
  const [query, setQuery] = createSignal('')
  const [filter, setFilter] = createSignal('')
  const [sortBy, setSortBy] = createSignal('relevance')
  const [results] = createResource(() => ({ query: query(), filter: filter(), sortBy: sortBy() }), searchArticles)

  async function searchArticles({ query, filter, sortBy }) {
    if (!query) return []
    const searchParams = {
      filter: filter || undefined,
      sort: sortBy === 'date' ? ['date:desc'] : undefined
    }
    const { hits } = await searchClient.index('articles').search(query, searchParams)
    return hits
  }

  return (
    <div>
      <input
        type="text"
        value={query()}
        onInput={(e) => setQuery(e.target.value)}
        placeholder="Search articles..."
      />
      <select value={filter()} onChange={(e) => setFilter(e.target.value)}>
        <option value="">All Categories</option>
        <option value="category = 'Technology'">Technology</option>
        <option value="category = 'Science'">Science</option>
      </select>
      <select value={sortBy()} onChange={(e) => setSortBy(e.target.value)}>
        <option value="relevance">Relevance</option>
        <option value="date">Date (Newest First)</option>
      </select>
      <ul>
        <For each={results()}>{(article) => 
          <li>{article.title} - {article.category} ({new Date(article.date).toLocaleDateString()})</li>
        }</For>
      </ul>
    </div>
  )
}
```
![Filter and sorting options](<Screenshot 2024-07-11 at 09.44.53.png>)

### Utilizing Meilisearch advanced features (faceted search)
Extend your hooks to add faceted search, typo tolerance features with following code:

```javascript
// ... (rest of the code remains the same)

action('articles.items.create', async (meta) => {
  await ensureIndexConfigured(async () => {
    await index.addDocuments([{ 
      id: meta.key, 
      ...meta.payload,
      _categories: [meta.payload.category],
      _tags: Array.isArray(meta.payload.tags) ? meta.payload.tags : meta.payload.tags.split(',')
    }])
  })
})

action('articles.items.update', async (meta) => {
  await ensureIndexConfigured(async () => {
    await Promise.all(
      meta.keys.map(async (key) => 
        await index.updateDocuments([{ 
          id: key, 
          ...meta.payload,
          _categories: meta.payload.category ? [meta.payload.category] : undefined,
          _tags: meta.payload.tags ? (Array.isArray(meta.payload.tags) ? meta.payload.tags : meta.payload.tags.split(',')) : undefined
        }])
      )
    )
  })
})

// ... (delete action remains the same)
```
The above code:
- Converts `tags` to an array if they're provided as a comma-separated string.
- Handles tags similarly to the create action, but only if they're included in the update.

Update your `src/components/ArticleSearch.jsx` file to add faceted search feature:

```javascript
import { createSignal, createResource, For } from 'solid-js'
import { searchClient } from '../lib/meilisearch'

export default function ArticleSearch() {
  const [query, setQuery] = createSignal('')
  const [selectedFacets, setSelectedFacets] = createSignal({ categories: [], tags: [] })
  const [sortBy, setSortBy] = createSignal('relevance')
  const [results] = createResource(() => ({ query: query(), facets: selectedFacets(), sortBy: sortBy() }), searchArticles)

  async function searchArticles({ query, facets, sortBy }) {
    if (!query) return { hits: [], facetDistribution: {} }
    
    let filterStr = []
    if (facets.categories.length > 0) {
      filterStr.push(`(category IN [${facets.categories.map(c => `"${c}"`).join(',')}])`)
    }
    if (facets.tags.length > 0) {
      filterStr.push(`(tags IN [${facets.tags.map(t => `"${t}"`).join(',')}])`)
    }

    const searchParams = {
      facets: ['category', 'tags'],
      filter: filterStr.length > 0 ? filterStr.join(' AND ') : undefined,
      sort: sortBy === 'date' ? ['date:desc'] : undefined
    }
    
    return await searchClient.index('articles').search(query, searchParams)
  }

  const toggleFacet = (type, value) => {
    setSelectedFacets(prev => {
      const newFacets = { ...prev }
      if (newFacets[type].includes(value)) {
        newFacets[type] = newFacets[type].filter(v => v !== value)
      } else {
        newFacets[type] = [...newFacets[type], value]
      }
      return newFacets
    })
  }

  return (
    <div>
      <input
        type="text"
        value={query()}
        onInput={(e) => setQuery(e.target.value)}
        placeholder="Search articles..."
      />
      <select value={sortBy()} onChange={(e) => setSortBy(e.target.value)}>
        <option value="relevance">Relevance</option>
        <option value="date">Date (Newest First)</option>
      </select>
      <div>
        <h3>Categories</h3>
        <For each={Object.entries(results()?.facetDistribution?.category || {})}>
          {([category, count]) => (
            <label>
              <input
                type="checkbox"
                checked={selectedFacets().categories.includes(category)}
                onChange={() => toggleFacet('categories', category)}
              />
              {category} ({count})
            </label>
          )}
        </For>
      </div>
      <div>
        <h3>Tags</h3>
        <For each={Object.entries(results()?.facetDistribution?.tags || {})}>
          {([tag, count]) => (
            <label>
              <input
                type="checkbox"
                checked={selectedFacets().tags.includes(tag)}
                onChange={() => toggleFacet('tags', tag)}
              />
              {tag} ({count})
            </label>
          )}
        </For>
      </div>
      <ul>
        <For each={results()?.hits}>{(article) => 
          <li>{article.title} - {article.category} ({new Date(article.date).toLocaleDateString()})</li>
        }</For>
      </ul>
    </div>
  )
}
```
![Faceted search feature](<Screenshot 2024-07-11 at 16.21.40.png>)

## Summary
In this tutorial, you've learned how to integrate Meilisearch indexing with Directus and SolidStart. You've built a SolidStart application that automatically indexes data created, updated, or deleted from a Directus project in Meilisearch.