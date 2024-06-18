---
title: "Building a URL Shortener with React, TypeScript, and Directus"
description: "Learn to build a link shortener using Directus and React! Define data models, customize roles, and dynamically route with ease."
author:
  name: "Onyedikachi Eni"
  avatar_file_name: "profile-photo.jpg"
---

In this tutorial, we will build a link shortener. Then, we'll create a React project that looks for a slug, queries the associated record in Directus, and redirects the user to the configured link.

## Before You Start

You will need:

1. A Directus project - follow our [quickstart guide](https://docs.directus.io/getting-started/quickstart) if you don't already have one.
2. Some knowledge of Javascript and React.js.

The complete code for this project can be found on [GitHub](https://github.com/xKachi/Link-Shortener/tree/main)

## Setting Up Your Directus Project

Create a new collection called `short_link`. Enable all optional fields, and add the following additional fields:

- `slug` (type: String, interface: input, required): The URL path that will be shared in the short URL.
- `url` (type: String, interface: input, required): The URL we want to redirect to.
- `clicks` (type: Integer, interface: input, default_value: 0): Number of times the link was clicked.

To make sure the URL that will be stored in the **URL** field is valid, we will use a validation filter.

Create a validation rule for the URL. From the field settings, ensure the URL **Matches RegExp**:

```
https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)
```

![Validation regex](<Setup a Directus Collection_3.PNG>)
Create a new `contributor` role will have the following privileges:

- Create a short_link.
- Read a short_link
- Update the `clicks` field (to avoid spamming) of short_link.

![Contributor Permissions - create and read short link are enabled. Edit is custom.](<Customizing Roles and Permissions_2.PNG>)

![Contributor Privileges](<Customizing Roles and Permissions_3.PNG>)

Finally, create some sample data in your `short_link` collection. 

| Slug       | URL                    |
| ---------- | ---------------------- |
| website    | https://directus.io/   |
| x          | https://x.com/directus |

## Setting Up Your React Project

Install React.js, set up dependencies, and run a development server:

```
npm create vite@latest link-shortener
? Select a framework: React
? Select a variant: TypeScript

npm install
npm install --save-dev @types/node
npm install react-router-dom
npm install @directus/sdk 

npm run dev
```

Now, open http://localhost:5173/ on your browser, and you should see the starter page.

Next, we will set up Vite to allow environment variables, which will be used to store our Directus credentials. In the `vite.config.ts` file, add the following code:

```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  define: {
    "process.env.VITE_DIRECTUS_API_TOKEN.": JSON.stringify(
      process.env.VITE_DIRECTUS_API_TOKEN
    ),
    "process.env.VITE_DIRECTUS_API_URL.": JSON.stringify(
      process.env.VITE_DIRECTUS_API_URL
    ),
  },
});
```

Create a `.env` file in the `src` directory, and the following variables, being sure to provide your specific static authentication token and project URL:

```
VITE_DIRECTUS_API_TOKEN = XXXXX
VITE__DIRECTUS_API_URL = https://your-amazing.directus.app/
```

In the `src` directory create a sub-directory called `utils`, and within it, a `directus.ts` file.

Add the following code to _directus.ts_:

```typescript
import { createDirectus, staticToken, rest } from "@directus/sdk";

const directusToken = import.meta.env.VITE_DIRECTUS_API_TOKEN;
const directusUrl = import.meta.env.VITE__DIRECTUS_API_URL;

if (!directusToken) {
  throw new Error("Please include a Token");
}

if (!directusUrl) {
  throw new Error("Please include a Url");
}

export const directus = createDirectus(directusUrl)
  .with(staticToken(directusToken))
  .with(rest());
```

### Creating the Dynamic Route

Let's create the route that will be used to redirect the user to the slug URL queried from Directus.

The `react-router-dom` library will be used to handle client-side routing, install the `react-router-dom` library by running the command:

```
npm install react-router-dom
```

Next, in the _App.tsx_ file add the following code:

```typescript
import { BrowserRouter, Routes, Route } from "react-router-dom";
import { useParams } from "react-router-dom";

function App() {
  return (
    <>
      <h1>Link Shortener</h1>
      <BrowserRouter>
        <Routes>
          <Route path="/:slug" element={<LinkRoute />}></Route>
        </Routes>
      </BrowserRouter>
    </>
  );
}

function LinkRoute() {
  const { slug } = useParams();

  console.log(slug);

  return (
    <>
      <h1>{slug}</h1>
    </>
  );
}

export default App;
```

Browser Output:

![Dynamic URL Output](<Creating the Dynamic Route (1).gif>)

### Querying Directus

Next, we will query the associated record for the slug in Directus and redirect the user to the route link.

Enter the following code in _App.tsx_:

```typescript
import { BrowserRouter, Routes, Route } from "react-router-dom";
import { useParams } from "react-router-dom";
import { directus } from "./util/directus";
import { readItems, updateItem } from "@directus/sdk";
import { useEffect, useState } from "react";

interface ShortLink {
  clicks: number;
  date_created?: string;
  date_updated?: string;
  id: number;
  slug: string;
  sort?: null;
  url: string;
  user_created?: string;
  user_updated?: null;
}

function App() {
  return (
    <>
      <h1>Link Shortener</h1>
      <BrowserRouter>
        <Routes>
          <Route path="/:slug" element={<LinkRoute />}></Route>
        </Routes>
      </BrowserRouter>
    </>
  );
}

function LinkRoute() {
  const { slug } = useParams();
  const [slugError, setSlugError] = useState("");

  useEffect(() => {
    // Perform data fetching based on slug
    async function fetchShortLink() {
      try {
        const data = await directus.request(readItems("short_link"));

        const slug_data = data
          .map((y) => y)
          .filter((z) => z.slug.toLowerCase().includes(slug));

        if (!slug_data || slug_data?.length === 0) {
          setSlugError(`Invalid Slug: Couldn't find the record →→ ${slug}`);
          throw new Error("Invalid Slug: Couldn't find that record");
        }
        const shortLink = slug_data[0] as ShortLink;

        await directus.request(
          updateItem("short_link", shortLink.id, {
            clicks: shortLink.clicks + 1,
          })
        );

        window.location.assign(`${shortLink.url}`);
      } catch (error) {
        console.log(error);
      }
    }

    fetchShortLink();
  }, [slug]);

  return <>{slugError && <h1 style={{ color: "red" }}>{slugError}</h1>}</>;
}

export default App;
```

In the preceding code, the single route `//:slug` matches any URL with a slug parameter, and renders the `LinkRoute` component.

The `LinkRoute` component uses the `useParams` hook to get the slug parameter from the URL. It also uses the `useState` hook to store an error message if the slug is invalid.

The `useEffect` hook is used to fetch data from the Directus API via the `fetchShortLink` effect function, which performs the following actions:

> The `ShortLink` interface defines the shape of the short link data returned by the Directus API.

1. It makes a request to the Directus API to read all short links via the `readItems` composable.
2. The response data is filtered to find the short link that matches the current slug.
3. If no matching short link is found, it sets an error message and throws an error.
4. If a matching short link is found, it updates the short links `click` property by `1` via the `updateItem` composable from Directus API.
5. Finally, the user is redirected to the URL associated with the short link using the `window.location.assign` method.

Finally, if an error occurs during the data fetching process, the error is caught and logged to the console. The `LinkRoute` component also returns the error message if the slug is invalid.

Browser Output:

![Slug URL Output](<Querying Directus.gif>)

## Conclusion

You’ve just learned how to set up the data collection process and implement dynamic routing using React and Directus. The broad process was:

- Set up Directus data model
- Customize user roles and permissions
- API endpoint to query data from Directus
- Dynamic Route to navigate to slug URL

I hope you find this tutorial useful - if you have any questions or hurdles feel free to join the Directus [official Discord server](https://discord.com/invite/directus).
