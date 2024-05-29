---
title: "Build a URL Shortener with React, TypeScript, and Directus"
description: "Learn how to build a simple link shortener with Directus."
author:
  name: "Onyedikachi Eni"
  avatar_file_name: "profile-photo.jpg"
---

In this tutorial, we will build a link shortener by defining a data model (collection) that we'll use to store information about our short links. Then, we'll create a React project that looks for a slug, queries the associated record in Directus, and redirects the user to the route link.

## Before You Start

To follow along with this tutorial, it is important to have the following:

1. A Directus [project](https://directus.cloud/login?__hstc=11036843.2cf80fd4cb0cf8697ea41e55dc8fad7e.1710666413740.1716660343293.1716897283216.19&__hssc=11036843.3.1716897283216&__hsfp=325150612)
2. Basic knowledge of Javascript and React.js

The complete code for this project can be found on _[GitHub](https://github.com/xKachi/Link-Shortener/tree/main)_

## Setting up a Directus Collection for Data Storage

First of all, log in to your Directus app and navigate to **Settings > Data Model**

![short_link collection](https://hackmd.io/_uploads/BJVasrm4C.png)

Click the + icon to create a new collection called **"short_link"**.

We will get the following fields by default from Directus

![default_fields](https://hackmd.io/_uploads/BJpiJL7VC.png)

Now, add the following custom fields to the **"short_link"** collection.

- **slug(Type: String, interface: input, required)**: To store the part of the URL we want to redirect to.
- **url(Type: String, interface: input, required)**: To store the the URL we want to redirect from.
- **clicks(Type: Integer, interface: input, default_value: 0)**: To store the amount of times the link was clicked.

To make sure the URL that will be stored in the **URL** field is valid, we will use a validation filter.

Navigate to **url > validation**, click on **URL**, and select the option **"Matches RegExp"**. Now, enter the following regular expression.

```
https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)
```

![validation_regex](https://hackmd.io/_uploads/SkmDP87NA.png)

After creating the fields our **short_link** collection will look like this:

![short_link collection2](https://hackmd.io/_uploads/Hk3QCSQE0.png)

### Customizing Roles and Permissions

Let's determine _who_ can access _what_ inside our database via roles and permissions. By default, there is already an admin with all permissions.

Navigate to **Settings > Access Control**, and create a new role called **"Contributor"**. The contributor will have access to the collection, but not admin privileges.

![Contributor_Previllege](https://hackmd.io/_uploads/r1p6nI7NC.png)

A **Contributor** will have the following privileges.

- Create a short link.
- Read a short link
- Update the **clicks** field(to avoid spamming) of a short link.

![Contributor_2](https://hackmd.io/_uploads/rJ43AUXNA.png)

To enable a Contributor to update only the **clicks** do the following:

1. Click on the **Update** icon.
2. Select the **Use Custom** option from the dropdown.
3. Select Field Permissions, and tick all options except **clicks**.

![Contributor_3](https://hackmd.io/_uploads/HJpxZw7N0.png)

### Creating Sample Short Links

To create content, navigate to **Content > short_link**, and click the **Create Item** button to create a new item.

![Item1](https://hackmd.io/_uploads/SJAw_DQER.png)

We will create two items(short links), one for the Directus website, and the other for the twitter(X) page.

| Slug       | URL                    |
| ---------- | ---------------------- |
| website    | https://directus.io/   |
| X(twitter) | https://x.com/directus |

![Item2](https://hackmd.io/_uploads/rkumivXVC.png)

## React(Vite) Project Setup with Typescript

We will be using _[vite](https://vitejs.dev/)_ to create our react app.

Install React.js by running the following commands.

```
 npm create vite@latest link-shortener

  # Select React as the framework and Typescript as a variant

 npm install
```

Also, the `types/node` package.

```
npm i --save-dev @types/node
```

Run the scaffolded React.js application with this command

```
 npm run dev
```

Now, open http://localhost:5173/ on your browser, and you should see the Vite(react) starter page.

![Vite_React](https://hackmd.io/_uploads/HJEFK_7EC.png)

Next, we will set up Vite to allow environment variables, which will be used to store our Directus credentials.

## Directus Tokens and Environment Variables

In the _vite.config.ts_ file, add the following code:

```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

// https://vitejs.dev/config/
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

Create a _.env_ file in the _src_ directory, and the following code:

```
VITE_DIRECTUS_API_TOKEN = XXXXX
VITE__DIRECTUS_API_URL = https://linksshortener.directus.app/
```

To create a Static Token for [authentication](https://docs.directus.io/reference/authentication.html) do the following:

1. On the Directus dashboard, navigate to **User Directory > Administrator**
2. Click on the **Admin User** icon.
3. Scroll down to the **Admin Options** section
4. Click on the + icon in the **Token** field to generate token.
5. Copy the generated token and paste it in `VITE_DIRECTUS_API_TOKEN`.

![Admin_Token](https://hackmd.io/_uploads/rJ6fHY7E0.png)

For the project URL, copy the URL from your browser.

![App_URL](https://hackmd.io/_uploads/H11fDKmER.png)

## Integrating the Directus SDK with React

Install the SDK using the command:

```
npm install @directus/sdk
```

### Import the SDK Composables

In the _src_ directory create a sub-directory called _utils_, in _utils_ create a _directus.ts_ file.

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

- `staticToken`: is used to authenticate the client we'll create.
- `createDirectus`: is a function that initializes a Directus client.
- `rest()`: enables the `.request(...)` method to query the collection.

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

![bandicam2024-05-2910-26-52-145-ezgif.com-video-to-gif-converter](https://hackmd.io/_uploads/B1nZIdNV0.gif)

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

![bandicam2024-05-2912-32-25-477-ezgif.com-video-to-gif-converter](https://hackmd.io/_uploads/SJTK4cN40.gif)

## Conclusion

You’ve just learned how to set up the data collection process and implement dynamic routing using React and Directus. The broad process was:

- Set up Directus data model
- Customize user roles and permissions
- API endpoint to query data from Directus
- Dynamic Route to navigate to slug URL

I hope you find this tutorial useful - if you have any questions or hurdles feel free to join the Directus [official Discord server](https://discord.com/invite/directus).
