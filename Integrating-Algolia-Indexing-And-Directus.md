# Integrating Algolia Indexing and Directus

Introduction
------------

In this article, we will explore how to index data from Directus in Algolia, enabling you to track created, updated, and deleted data efficiently. This process is straightforward if you understand the [role of Extensions in Directus](https://docs.directus.io/extensions/creating-extensions.html), and this article will simplify it for you.

Getting Started
---------------

To follow along with this project, readers will need to:

1.  Install Node.js: Ensure Node v20(mandatory) is installed on your machine. You can download it from nodejs.org.

2.  Set Up a Directus Dev enviroment:

    -   Follow the [steps in this docs](https://docs.directus.io/contributing/running-locally.html) to get one up and running on your computer.

    -   Open it in your preferred code editor.

3.  Create a Directus extension

    -   To create a Directus Extension [follow the docs](https://docs.directus.io/extensions/creating-extensions.html).

    -   In your Directus project, create an extension and name it `Indexing`.

4.  Install Algolia

    -   In your terminal, Navigate into the `Indexing` extension directory, and run `npm install algoliasearch`, to install algolia.

With these prerequisites in place, you'll be ready to follow up on this article.

What is the link between Directus and Algolia?
----------------------------------------------

Directus is a headless CMS that manages our data, including blog data like title, content, and author. On the other hand, Algolia is a tool that enhances search functionality in an application, making it easier for users to find content. For example, [Dev.to ](http://dev.to)uses Algolia for their searches.

![A Diagram explaining the flow of data from Directus to Algolia](https://lh7-us.googleusercontent.com/docsz/AD_4nXcF1_wSEzJ-vh_tvsavp_qp4hrA9vXadf6KDsZPKa1qFQB82Exl2xopUhl9AS1coxdzbkJVu3-JtNNnnSk4F_w7pxaZfCniNghKHdXDE901Q4Spu-VoOo7SHI3gsyZ7XKSWPk1OME0Xwwn3_kqN7uv5dMwN?key=Ph15yUkN_4DIsNV-CkzKUw)

In this context, we want Directus to manage our blog data and use Algolia to enable a search feature for our blog. Specifically, we'll focus on indexing our created, updated, and deleted data from Directus to Algolia. 

Let's break it down:

-   Directus: Manages blog data (title, content, author).

-   Algolia: Enhances search functionality in our application.

-   Integration: We'll index data from Directus to Algolia using [Directus extensions](https://docs.directus.io/extensions/creating-extensions.html).

Settng Up Algolia
----------------

To integrate Directus and Algolia we will need our Algolia application ID and write API key. If you don't have an account already, [create one](https://www.algolia.com/users/sign_up), and you will see these credentials in your dashboard.  

  ![An image of Algolia Dashboard](https://lh7-us.googleusercontent.com/docsz/AD_4nXeSmoCJEeIFzZVdi80jhOgarVfK5eErrVHLdgIm8wnN_aUj-zkYRsiXylMUSz7Iz0A0gRdpQNqDtcH8qM4u-VLsVKE2-H72-NlP5zmkGzFLekZkCN3iguXvRkB_2b7IswuIFCsuPV35wRByUjJR0jGdEVQc?key=Ph15yUkN_4DIsNV-CkzKUw)

Now, in our `scr/index.js` file in `indexing` extension, we will paste the code below;

```js
import algoliasearch from  'algoliasearch';

// Initialize the Algolia client
const client = algoliasearch('*********', '*******************');
const index = client.initIndex('Directus_Events');

export  default ({ action }) => {
    action('posts.items.create', async (meta) => {
 console.log(meta, 'This is the meta payload');
 await index.saveObjects([{ objectID: `${meta.key}`, ...meta.payload }]);
    });

    action('posts.items.update', async (meta) => { await  Promise.all(
            meta.keys.map(async (key) => await index.partialUpdateObjects([{ objectID: `${key}`, ...meta.payload }])),
        );
    });

    action('posts.items.delete', async (meta) => {
 await index.deleteObjects(meta.keys);
    });
};
```

The code above sets up series of [action hooks](https://docs.directus.io/extensions/hooks.html#action) to be able to synchronize content changes in Directus with Algolia. After importing the Algolia client library, we initialized it using our unique application ID and admin API key, thereby creating an Algolia index named Directus_Events.

Going further, the code exports a default function that accepts an object containing an action method. This action method is used to register hooks for different Directus events: creation, update, and deletion of items in the posts collection to be created.

For item creation (posts.items.create), the code registers a hook that triggers when a new item is added to the posts collection. The item is saved with an objectID set to the item's unique identifier, ensuring it can be accurately referenced and managed in Algolia.

For item updates (posts.items.update), the code registers a hook that triggers when an item in the posts collection is updated, and lastly, for item deletions (posts.items.delete), the code registers a hook that triggers when an item is deleted from the posts collection. Go ahead to build this extension in our terminal by navigating to the extension directory and running this command `npm run build`, this will be able to build our extension properly and set to use.

Setting Up Directus Blog Content
--------------------------------

To see this work, we need to create a folder named `extensions` in our `API` directory, then in it, copy our extension which is `indexing`, and paste it in the `extensions` folder we just created. [Rename the copied extensions](https://docs.directus.io/guides/extensions/hooks-add-stripe-customer.html) to `directus-extension-indexing` to enable Directus identify it when it runs. Save and build your directus project with `pnpm --filter api dev`. Our local Directus project will be running on `http://localhost:8055/`.

Log in with your credentials, and go ahead to create your contents. Navigate to settings, select Data Model, and click the plus icon at the top right corner of the the screen to create a collection. Name the collection Blog, and that will be all. You will need to also create the following fields:

-   Title: This field will store the title of the blog post.

    -   Field Name: title

    -   Field Type: Text

-   Content: This field will contain the main content of the blog post.

    -   Field Name: content

    -   Field Type: WYSIWYG

-   Author: This field will store the name of the author of the blog post.

    -   Field Name: Author

    -   Field Type: text

![An image showing fields created in Directus](https://lh7-us.googleusercontent.com/docsz/AD_4nXeaOSTBCNTct1im_61dH3ptHdKOK1F4m_cFJWyAYHUROS1tR0t2CGQ-_ChWQ9pRfNfLENgg_QBVl6K1i2U_1uhDmYsSYIOzOpAJW1n5v6m8ALAWfEl0OjF8PSXut6Go8z8OhjIZkOjpFwUyudEAkTlHNVbA?key=Ph15yUkN_4DIsNV-CkzKUw)

When we navigate to the content, and select the Blog collection we will be able to fill in our blog post, we will do this in the next section of the article.

Testing if It Works
-------------------

To test if the integration works, we will create a blog post in Directus, using these contents:

- Blog Title: "Will AI Take My Job?"

-   Content: "In this blog post, we explore the future of AI and its implications..."

-   Author: "Marvel Ken"  

![An image of the collection](https://lh7-us.googleusercontent.com/docsz/AD_4nXeiPVIwmJRPvplFLV-9zAjsdnAqTV0t5zKHeBa753KtjZcK0ovcd02cnC5y82l_M1MCSUmHpkuzT5fqDK1_wf-nTaVDsq23x3WW668MDZJ0S7i3ys-HjwBDP4UkjN-JQ0xHgnuAsb_VxIma5V3RVdwuDww?key=Ph15yUkN_4DIsNV-CkzKUw)

To verify that the indexing process is functioning as expected, navigate to the Algolia Dashboard. Click on "Search" in the navigation menu on the left side of your screen, then select the index. You should find the index you created: Directus_Events;

![An image of the index created in Algolia](https://lh7-us.googleusercontent.com/docsz/AD_4nXcOvSPlqIgW4px0tUsYzxN7K4aOLN2Vkcx547YLVpMfag-bDEKnf17vGyGyIV_a5Ow0j6fxUQu4eUJTcJEsw-XqlHqvcp8rlobtlvobW0BdAe_XZKzhrgW31UO9DbI0eUGKNQa8y_3WsA--FQFWM9LyfnrE?key=Ph15yUkN_4DIsNV-CkzKUw)

Below you should be able to confirm that Algolia has recognized the new data:

![An image of the created blog post](https://lh7-us.googleusercontent.com/docsz/AD_4nXczHhPz-nhYkDUE0vM8Tdt-XWj6f1zlBf7CVowvIo4aTCxgufUchiXHoKPamvPfApGyEe4YTZWXnwLxsviPBGmDu64CrDMOgNCaqK549hm24rKuq8AnWn6TfkxKpM-2MZxru7fJP720R8CIek1kd79ghq8?key=Ph15yUkN_4DIsNV-CkzKUw)

To confirm if the updates are correctly reflected, we will update the blog post in Directus by adding dates in Diorectus;

![An image of the updated blog post](https://lh7-us.googleusercontent.com/docsz/AD_4nXeUnRoC7CiK6fDENUYS1-CxCX23_fQ6BCHdxkqTIDwKDrOb0GC1YWz4fQ-HO-iOtN9xsUpPmQ6meBCBLWVWV_WvpazeeWq461n0kF51YFZ8Du5w_QQUYsS14l3JMnx_-AhGNKABTH0nc-OzsNl0G5DfBOCI?key=Ph15yUkN_4DIsNV-CkzKUw)

Lastly, to confirm if the delete function works we will delete the Collection in Directus:

![An image of the deleted blog post](https://lh7-us.googleusercontent.com/docsz/AD_4nXd7zYGKzlT-zHxiFo9ltmNaZWyBXHsPR70ZJY4QE5QsyJL0PRm-l1ckkMOQye9gwBjdWq5R1JIdGkq-TWVcQUGWM2ZB2D-iRIdpzlZsa1tXKRMYTl_nIrDqpKu75nlhCFszLrZKb7kqZ8b90wyHq3-3MrVb?key=Ph15yUkN_4DIsNV-CkzKUw)

Conclusion
----------

By following this guide, you have learned how to set up extensions in Directus. You also saw how to test the integration by creating, updating, and deleting data in Directus, with changes being reflected in your Algolia index. This setup ensures that our data remains synchronized across both platforms, helping us have a good search experience. Keep coding, and keep using Directus!!