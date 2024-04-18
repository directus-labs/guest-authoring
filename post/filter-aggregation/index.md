---
title: "Advanced Filtering: Dates, Aggregation & Grouping, and Combining Filters"
description: "120-160 characters"
author:
    name: "jay bharadia"
    avatar_file_name: "my-avatar.jpg"
---

## Introduction

Directus is a really powerful data platform that automatically generates REST and GraphQL APIs, and an SDK to make working with them more convenient.

There's a number of powerful query parameters to filter, sort, limit, and apply functions to the data returned. But with any powerful tool, complex usage can require a learning curve.

In this post, we'll cover some common and powerful advanced querying that you can use in your applications. In this example, imagine a blog with a search and some time-based filters. Note that each filter has a number of posts that match the query.

![A list of blog posts with a sidebar showing a search, a year filter with 2022 and 2023 checkboxes, and the same for months. Each year and month has a number in brackets next to it.](https://github.com/directus-community/guest-authoring/assets/1461554/e6e229bb-13e1-4a54-bfe7-623c12425bb3)

Before diving into queries, set up a new collection called `articles` with `title` and `published_date` fields. Any extra fields are fine, but we won't use them today. Then create 10 example items that we can use to test our queries.
In your Public role permissions, allow read access to the `articles` collection.

---

### Initialize the Directus SDK

-   Create a new directory and open it in both your code editor and a terminal.
-   In your terminal, run `npm init -y` to create a `package.json` file.
-   Run `npm install @directus/sdk` to install the Directus SDK.
-   Create an `index.js` file and initialize the SDK, replacing the example URL with your Directus Project URL:

```js
import { createDirectus, rest, aggregate, readItems } from "@directus/sdk";
const client = createDirectus("https://directus.example.com").with(rest());
```

### Get Data By Month

-   The following filter will return any item in the `articles` collection that is published in months 1, 2, or 3 (January, February, or March).
-   This filter uses the `_or` [filter operator](https://docs.directus.io/reference/filter-rules.html#filter-operators) which contains an array.
-   If at least one of the object conditions are `true`, it will be included.
-   `month()` is a [DateTime function](https://docs.directus.io/reference/query.html#datetime-functions) that will extract the month from a datetime, date, or timestamp field and return the month number.

```js
client.request(
    readItems("articles", {
        filter: {
            _or: [
                {
                    "month(published_date)": {
                        // January
                        _eq: 1,
                    },
                },
                {
                    "month(published_date)": {
                        // Feb
                        _eq: 2,
                    },
                },
                {
                    "month(published_date)": {
                        // March
                        _eq: 3,
                    },
                },
            ],
        },
    })
);
```

Save the file, and run `node index.js` to try it out and see an array of matching items. Of course, you require at least one item which matches the criteria to receive a non-empty array.

### Get Data By Year

The same approach can be used with any other DateTime functions, such as `year()`:

```js
client.request(
    readItems("articles", {
        filter: {
            _or: [
                {
                    "year(published_date)": {
                        _eq: 2023,
                    },
                },
                {
                    "year(published_date)": {
                        _eq: 2022,
                    },
                },
            ],
        },
    })
);
```

:::info Dynamic Variables

Directus also has a set of [dynamic variables](https://docs.directus.io/reference/filter-rules.html#dynamic-variables) that can be used in filtering, including `$NOW` and `$NOW(<adjustment>)` (for example `$NOW(-1 week)`), that can be used for relative filters.
:::

### Combining Filters: Date & Search Term

Filter syntax can get confusing as you build increasingly complex requests. However, with what we've learnt so far, we can create something very powerful.
This filter starts with an `_and` logical operator - this means that every member of the array must be true for a match, and there are three:

1. Must be published in January, February, or March.
2. Must be published in 2022 or 2023.
3. The title must contain 'Hello'.
   Notice that for both of the month-based filters, it begins with an `_or` logical operator. Only one of these must match for the whole item to count for the `_and` operator.

```js
client.request(
    readItems("articles", {
        filter: {
            _and: [
                {
                    _or: [
                        {
                            "month(published_date)": {
                                _eq: 1,
                            },
                        },
                        {
                            "month(published_date)": {
                                _eq: 2,
                            },
                        },
                        {
                            "month(published_date)": {
                                _eq: 3,
                            },
                        },
                    ],
                },
                {
                    _or: [
                        {
                            "year(published_date)": {
                                _eq: 2022,
                            },
                        },
                        {
                            "year(published_date)": {
                                _eq: 2023,
                            },
                        },
                    ],
                },
                {
                    title: {
                        _contains: "Hello",
                    },
                },
            ],
        },
    })
);
```

## Aggregation & Grouping

Let's say you want to show data by perform math operations like count. this is where aggregation and grouping is used. lets check the example

### Count Articles By Year

In this example, we will combine aggregation and grouping. Aggregation allows us to perform a count on all articles that exist. The result will then be displayed for each bucket as grouped by the year value.

```js
const result = await client.request(
	aggregate('articles', {
		aggregate: { count: '*' },
    groupBy: ['year(published_date)']
	})
);
// Response
{
  "data": [
    {
      "published_date_year": 2022,
      "count": 35
    },
    {
      "published_date_year": 2023,
      "count": 44
    }
  ]
}
```

You can use the `month()` DateTime function to perform the same logic for the month filters.

## Summary

The examples given above use hard-coded values, but with little additional work in your real-world application, you can create dynamic queries that returns just the data your users want.
By following this guide based on a real-world example, you have not only learnt how to combine filters, but also been introduced to logical operators, DateTime functions, and explored aggregation & grouping.
