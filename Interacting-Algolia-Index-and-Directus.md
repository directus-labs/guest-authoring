# Integrating Algolia Indexing and Directus


Introduction
------------


In this article, we will explore how to index data from Directus in Algolia, enabling you to track created, updated, and deleted data efficiently. This process is straightforward if you understand the [role of Flows in Directus](https://docs.directus.io/reference/system/roles), and this article will simplify it for you.


Getting Started
---------------


To follow along with this article, readers will need to:


1.  Install Node.js: Ensure Node.js is installed on your machine. You can download it from[  nodejs.org](https://nodejs.org/).


2.  Set Up a Next.js Project:


-   Run the following command to create a new Next.js project:\
    `npx create-next-app@latest`


-   Follow the prompts to name your project and configure additional options as needed. 


![](https://lh7-us.googleusercontent.com/docsz/AD_4nXcvBoW5GcyXS-arXk_6oB_aJf3g5RlVCBjuLfNlvSv90GIvv4AH9JCGN0qalJRfOQZSToniYpQ-FvA2DLaeYWSDplJK2JDHGK4FSN0kiK3t4Rwl38b7MCYu1lrLDXz5L5QSh1gH1CDM2bxWe3Ufxfjd4_Di?key=Ph15yUkN_4DIsNV-CkzKUw)


[Refer to Next.js documentation](http://next.js) if you need further guidance.


1.  Install Algolia:


    -   Navigate to your project directory and install Algolia by running:\
    `npm install algoliasearch`


2.  Prepare a Directus Project: Ensure you have a Directus project set up and running. This will be used to manage your blog data.


3.  Knowledge Prerequisites:


    -   Webhooks: Basic understanding of what webhooks are and how they work.


    -   Asynchronous Operations: Familiarity with handling asynchronous operations in JavaScript.


    -   Making POST Requests from a CMS: Basic knowledge of how to make POST requests to external services from a CMS.


    -   Next.js Serverless Functions: Understanding of how to create and use serverless functions in Next.js.


With these prerequisites in place, you'll be ready to follow up this article.


What is the link between Directus and Algolia?
----------------------------------------------


Directus is a headless CMS that manages our data, including blog data like title, content, and author. On the other hand, Algolia is a tool that enhances search functionality in an application, making it easier for users to find content. For example, [Dev.to ](http://dev.to)uses Algolia for their searches.


![An image explaing how data is integrated from Directus to Algolia](https://lh7-us.googleusercontent.com/docsz/AD_4nXcjadakooGcT4DuzRyD_IUVWcqfMEX0fnPAa9De7-qxDKaDgJ5jqPFa1jWh6LCmluwn8bxLAmSHG7sYMqOEqyvpcmllDhzwgXhNOzQ4BAIxQtq7FPBIZsOUtpdfv75r3ILOXd5fjKJYhQPMM_W8YzdGBfY6?key=Ph15yUkN_4DIsNV-CkzKUw)


In the context of the image above, we want Directus to manage our blog data and use Algolia to enable a search feature for our blog. Specifically, we'll focus on indexing our created and updated data from Directus to Algolia. You may ask what exactly is the Next.js job here? Next.js in this scenario is the contract agreement between Directus and Algolia, so it will be used for our backend functionalities to achieve this.


Let's break it down:


-   Directus: Manages blog data (title, content, author).


-   Algolia: Enhances search functionality in our application.


-   Integration: We'll index data from Directus to Algolia.


-   Next.js: Handles the backend processes for this integration.


Setting Up Directus
-------------------


You will [need to have an account](https://docs.directus.io/getting-started/quickstart.html) if you don't already have one, do well to launch the project afterward. In Directus, we will go ahead to create our content. Navigate to settings, select Data Model, and click the plus icon at the top right corner of the the screen to create a collection. Name the collection Blog, and that will be all. You will need to also create the following fields:


-   Title: This field will store the title of the blog post.


    -   Field Name: title


    -   Field Type: Text


-   Content: This field will contain the main content of the blog post.


    -   Field Name: content


    -   Field Type: WYSIWYG


-   Author: This field will store the name or ID of the author of the blog post.


    -   Field Name: Author


    -  Field Type: Text


![An image of the collection fields ](https://lh7-us.googleusercontent.com/docsz/AD_4nXcSkZgvEXBaTAEx5JA29NG17ZUFvJOJMDPWT-BLIMnN1dPqJ_Vtwk5Zgss552RLlSpnQ7KDKPsdKs1T3gw5qr95f2mGIbCqj4OenV2iMNWMzoeTTre9x8YW-ph0lCxlbWqLH9XrtMCd3e96uF48vtjOb_GN?key=Ph15yUkN_4DIsNV-CkzKUw)


When we navigate to the content, and select the Blog collection we will be able to fill in our blog post, we will do this later in the article.


We need to interact with an external service to either get data or post data. Naturally, we would use webhooks for this. However, Directus has replaced webhooks with "flows." To utilize this navigate to settings once more, select flows, and with the plus icon, create one.


Let's set up our Flows. This setup will ensure that any new or updated blog posts can be indexed in Algolia;


Name: We'll use "Indexing-Algolia":


![Naming the Flows](https://lh7-us.googleusercontent.com/docsz/AD_4nXc26kEr_xyAr8We8lVqWzC5GFh7piuukXumBQoujxSytlUU2RgrpGprWo73LG9M1zOJ-r8w06ZZZmJY581XZqk4IpopISZAylpbB6GOG1sZSdVs6EfMMu7QqCSFfpnpHjN4aAvSVDWIDDoryELWwGAgjS1n?key=Ph15yUkN_4DIsNV-CkzKUw)


Trigger Setup: Select Event Hook for the trigger;


![Selecting a trigger](https://lh7-us.googleusercontent.com/docsz/AD_4nXc6jXjxna9oqrpnD-wF1PNSZ8wphMFSMdK_aDwQssH39nhvYUO9mHTz_cZNGBrDThSkUauh3F0YUW09mwXTY6iyPR44FiTPTZv1O8-lSS4zjy0lxOl9BZ7xzv058bptsj35Hbku0WYS4k-jPB4dVcLZf5W6?key=Ph15yUkN_4DIsNV-CkzKUw)


Type: Choose Action (Non-blocking), and for Scope, We want the flow to trigger on both create and update events, so select both options. Choose the Blog collection. Then click the save icon at the top to apply these changes.


![picking the scope](https://lh7-us.googleusercontent.com/docsz/AD_4nXdBLnFiBOCzMRWZ_YYxMk7dBhqQWwUuBEILxVs8_Z7v_XAso4ViOnZ6VfPbXfyERoZ2cCPzu4yVA-00cFpteM1Ma0_pj743TbPhQrq_-sggNUz9qTOjJM76i3PWBJuZ3z9mcUZMvsm5lXjETjpdB0uY87-i?key=Ph15yUkN_4DIsNV-CkzKUw)


After making these edits, you should see a configuration similar to this:


-   Name: Indexing-Algolia


-   Trigger: Event Hook


-   Type: Action (Non-blocking)


-   Scope: Create, Update


-   Collection: Blog


![Flows Dashboard](https://lh7-us.googleusercontent.com/docsz/AD_4nXfU2axstzrGH9bP0YnF_vaGHtrY1GmIgfWYLTZkzJHMlusKBd8fiCDgPtwtr43EqoLShH9Fn9DyH0dIP1WPeY5pQXIQqFTGOv_KW0I_US7m-MjUzbzPjBqDSqzN-lYt3HYaGaLMO3-zqXeFnlXahMdbG-er?key=Ph15yUkN_4DIsNV-CkzKUw)


Going further, we will want to edit the Operations, this will be responsible for the connection between both Directus and Algolia. To do this:


Click the plus icon highlighted below;


![Editing the flows Operation](https://lh7-us.googleusercontent.com/docsz/AD_4nXeHnNp57uqF2VUyF2fUcDHsAv9wpAp39G-1cWmPBFDTn5U33AXp1aUhqCUFlBSLo8lTzi8_X1QimIma6pajnln9uFozWy4zIk_Xl2BfcFqcUFqtDxUqXXgUWwIJ9p-EcvURvz_H0u1O_ki65d5wlBm1oHc?key=Ph15yUkN_4DIsNV-CkzKUw)


This plus icon will create an operation, that will trigger whenever we create or update data in Directus, will use webhooks for this.  Let's make changes to it.


The name will be set automatically when you select webhook as an option, you can change it if you want to. 


![Selecting webhook](https://lh7-us.googleusercontent.com/docsz/AD_4nXe8XACWhyIbO4PU6kNmCgWkvpOH37qqgYuGwCCpxonKVNaQdMtp2kVGKpZ_IM_btWn5Hy8TGuOF5P8ZASgS2RNrkUKDIJhjrmbZbmFAGa6BXFZwMRhutwWfXux8hpIpDS6ghaf2o3n7saYCT89wrFXWjc4?key=Ph15yUkN_4DIsNV-CkzKUw)


For the method, we'll select a POST request. We'll leave the URL field open for now and come back to it later. For the request body, we'll use {{$trigger}}. This placeholder sends the entire data content to our backend, which will be specified in the URL later. This is where Next.js will come into play.


![selecting post method](https://lh7-us.googleusercontent.com/docsz/AD_4nXePdRKJcHJAUww3IQm5ZscK-0hbwIcjORnYisupORqPng2_6qtAQJ54SYjPOAvfaxJcVDtdy56U4hP3R9bpIha_IdXSAQKuSXXKexkGxjQslAi8i8lbhR428kOYV0jLz4cIQB1CGgAAmdDIszyJA-SgFS1Q?key=Ph15yUkN_4DIsNV-CkzKUw)


Here's the breakdown:


-   Method: Select POST request.


-   URL: Leave this field open for now; we'll return to it later.


-   Request Body: Enter `{{$trigger}}`.


This configuration ensures that the whole data content is sent to our backend (the URL we will define later), allowing Next.js to process it. 


![The flows dashboard](https://lh7-us.googleusercontent.com/docsz/AD_4nXcc2bsxllNgnmr7w0aV53Z99ybYMtY17eKWDbqgEqO8uPBYH4IxkI1RJrf3XlDhe2IYwSRkRt6yxl7v4l2ei8RHX3nxQSoPAPJseWakZquBoTMYUaLV7gwsyOBFDO2arNGW-UQuNZ2m75dJnKudyfe-tsY?key=Ph15yUkN_4DIsNV-CkzKUw)


Now we have our Flows set up, the next step will be using Next.js as an API handler to handle incoming requests from Directus and interact with Algolia to create, update or delete records based on the event type from Directus.


Indexing Algolia
----------------


Using Next.js as our backend simplifies everything, as we can easily integrate both frontend and backend functionalities. To set it up, we create the Necessary Folders:


-   In your project, create a folder named pages.


-   Inside the pages folder, create another folder named api.


-   In the api folder, create a file named updateAlgolia.js.


The file tree look like this:


├── pages/


│   ├── api/


│   │   └── updateAlgolia.js


By organizing your folders in this way, you can write your backend logic in Next.js, handling API requests directly. In our `UpdateAlgolia.js` file, we will create an index in Algolia manually, to do that, we will need our Algolia application ID and write API key. 


If you don't have an account already, [create one](https://www.algolia.com/users/sign_up), and you will see these credentials in your dashboard.  


  ![Algolia Dashboard](https://lh7-us.googleusercontent.com/docsz/AD_4nXdRPs48hrGCkJ2zXWyGy6ZBY3Mv2CPaOWPMyB5FJo5NDs2-zbi5yNNFQoGRgfn5RvNqT4NVTIPoHWRvSoVZUCQ9FLfEkqBXNM3GK33-Q4KAlqBE-Jawk1vUOJyHzPrpfIDyG5HpbcFF69NaQBlH0rK2e2yA?key=Ph15yUkN_4DIsNV-CkzKUw)


Now, in our `UpdateAlgolia.js` file, we will paste the code below;


```Js
// pages/api/updateAlgolia.js


import  algoliasearch  from  'algoliasearch';


// Initialize the Algolia client


const  client  =  algoliasearch('**********', '***************************');


const  index  =  client.initIndex('Directus_Events');
```


The code above initializes an Algolia client in a Next.js API route. First, it imports the Algolia search client library. Then, it creates an Algolia client using the provided Application ID and API Key. Lastly for this code, it sets up an index named `Directus_Events`, which will be used for storing and managing searchable data from Directus. This setup allows the API route to interact with Algolia for performing search and indexing operations.


Moving further we will want to process POST requests to update Algolia index based on events from Directus. After the Algolia client initialization, paste the code below;


```js

// API handler to update the Algolia client
export default async function handler(req, res) {
  if (req.method === "POST") {
    try {
      console.log(req.body, "show the body");
      if (req.body.event === "Blog.items.create") {
        await index.saveObjects([
          { objectID: `${req.body.key}`, ...req.body.payload },
        ]);
      } else if (req.body.event === "Blog.items.update") {
        console.log("show the updated data", req.body);
        await Promise.all(
          req.body.keys.map(
            async (key) =>
              await index.partialUpdateObjects([
                { objectID: `${key}`, ...req.body.payload },
              ])
          )
        );
      } else if (req.body.event === "Blog.items.delete") {
        console.log("show the deleted data", req.body);
        await index.deleteObjects(req.body.keys);
      }

      res.status(200).json({ message: "Success" });
    } catch (error) {
      res
        .status(500)
        .json({ message: "Error updating Algolia", error: error.message });
    }
  } else return res.status(400).json({ message: "Method not allowed" });
}

```


The code above is an API handler that processes `POST` requests from Directus to update an Algolia index. When a new item is created in Directus (`post.items.create`), it adds the item to Algolia using the `saveObjects` method. For updated items (`post.items.update`), it uses `partialUpdateObjects()` method to update, and for deleted items (`Blog.items.delete`) it uses `deleteobject()` to delete the existing records in Algolia.


The function checks the event type from Directus, handles the data accordingly, and logs relevant information for debugging. If the operations succeed, it returns a success message; otherwise, it catches and returns any errors encountered. By doing this we are almost done with indexing.


### Fixing the URL


Remember the URL we left empty in Directus? It was left empty because it should point to the deployed version of our Next.js API handler. Now, let's go ahead and deploy this handler.


First, push your Next.js project to a GitHub repository. Once it's on GitHub, create an account on [Vercel](https://vercel.com/) if you don't already have one. Link your Vercel account to your GitHub repositories, then select the Indexing-Algolia project and deploy it.


After deployment, your API will be accessible at a URL like https://your-project-name.vercel.app/api/updateAlgolia. Copy this URL and paste it into the URL input field we left empty in Directus. This will complete the setup, allowing Directus to send data to your deployed Next.js API handler for indexing in Algolia.


Testing if It Works
-------------------


To test the integration, we will create a blog post in Directus, using this content:


-   Blog Title: "Will AI Take My Job?"


-   Content: "In this blog post, we explore the future of AI and its implications..."


-   Author: "Marvel Ken"  


![Creating a Blog post with the collections](https://lh7-us.googleusercontent.com/docsz/AD_4nXcstK7E6DQ5l_lL2SwfVnVriZmtsvo2qzPLLdidKs0fD0VXKOE0LIuGxjdEFk5_Uqp9k371zrPVXkZ1BOtofhFkDX1KEDX1S7WFI7bv25SczVi8quXfurA_wHKLJVB8MK4i4JhGcJCovvyk9RX0Kmz29vQ?key=Ph15yUkN_4DIsNV-CkzKUw)


After creating the post, we will check two logs. The first will be Directus log and then the versel log. The Directus log will tell us if any data is properly sent. Navigate to the flow, by the right side of your screen you will see if anything was logged, open it up and check the trigger payload to see if the data was logged, and the Webhook/Request URL payload to see if there was an error;


Trigger Payload:


![Trigger payload log](https://lh7-us.googleusercontent.com/docsz/AD_4nXfp55PNzHSlr2KqGGrx-ac3whIG2OF38ngTe62J55B9qyBDtSLiRIF2sGaY_DDS0pGPwTBSLoVi1n0PqnYFMZAsjS9tz_ztm0JtD7oWk5-zIt0YTi6FTV-xszgP7vcGV89klbEqjagyYtT_KvF7cvXF712d?key=Ph15yUkN_4DIsNV-CkzKUw)


Webhook/Request URL Payload:


![Webhook/Request URL Payload log](https://lh7-us.googleusercontent.com/docsz/AD_4nXfX3wjIFHt3Ht0wXKGsaA7P2WJP_TdHlp57E7hJFE47rrK28VVnu3Lsw1cQpJcQwfrGfRKVcy-DsJEjyGLHaVOhdeK1TKH6JR3g4SbInnWW2lBlLZKakgjXKelKLuMVnd8KQjRDpCc8cszkXWmhOUa4XK0j?key=Ph15yUkN_4DIsNV-CkzKUw)


There was no error and it logged the data perfectly. We will also need to check Versel log and see if it received any data from Directus.


![Versel Log](https://lh7-us.googleusercontent.com/docsz/AD_4nXdw3j7lYDio1rLkf8_hquJmtsEX5A0HlSNwG7-kY3QvFbv7l2sk1czDseFLN00xnnc6zXO9XwC9SuYewxu0ch-gv38HHybrKsclkJkzVkzNFYwDQdQNzJ0X58-E0dUnYbOXYev1JAzCCaXS54aN4NrrtkiM?key=Ph15yUkN_4DIsNV-CkzKUw)


Yes! It received the content and it resembles ours. To verify that the indexing process is functioning as expected, navigate to the Algolia Dashboard. Click on "Search" in the navigation menu on the left side of your screen, then select the index. You should find the index you created: Directus_Events;


![Algolia index creation](https://lh7-us.googleusercontent.com/docsz/AD_4nXfxPfrdO3-szXg-Fvn_46w_Pc57iL0W3ZNrfLGVODBtEE3P3Z0GMQiFeh-7XNGgNl3gx7e1M9e-4dN4sHkkVEI7aUoAd1fpa_m6tcyDy7rUYRWgbfsLafxh713htn2bpOGJapUFmEyIpHMfInhN7ifWq-I_?key=Ph15yUkN_4DIsNV-CkzKUw)


Below you should be able to confirm that Algolia has recognized the new data:


![Algolia indexed Created Data](https://lh7-us.googleusercontent.com/docsz/AD_4nXcpEroj-g86rtnt_XfIQFCTIJ4qq7xwWMAWe9h_YvVooVzBd4uJif9agsTDsVV2hrbv11shHBeX-DRP5K8QahkwAmHtvGzX0YIo0gEQ94QpubwKkX11LoIs16gLhUUPozPqyhRbg5EtOZ1BVMIpoWtLMZA?key=Ph15yUkN_4DIsNV-CkzKUw)


To confirm if the updates are correctly reflected, we will update the blog post in Directus by adding dates;


![Algolia indexed updated Data](https://lh7-us.googleusercontent.com/docsz/AD_4nXeKVO9N1-vuwCcn0hRKtOvsW4d1bHvlz4PnzCBblPSx9DCSwZh5ATjeAvP1EdyuLbZsUkGjv3nb3wEG1ibEKX6X_TfXtFqrwgl6bP1XE0Y_sjWCwahMZVTvLxfXLqHadlqxvF2-A8sD53b61g-gzffb33Bl?key=Ph15yUkN_4DIsNV-CkzKUw)


Lastly, to confirm if the delete function works we will delete the Collection:


![Algolia indexed deleted Data](https://lh7-us.googleusercontent.com/docsz/AD_4nXcoGF1QlEJNy7xNO6K67vUKNym1CAxiK6bbQ8w16bWE9a3sMGCtN3_6hwaU5DWglbtwctRB0QLh_EC3MunUQe2KPeYXZwbhgrke6Go-OU-ynwuuSPLb8fl3OqVZ_SznAGTHnnUZo0RJNoFme9xaQFaHpAlk?key=Ph15yUkN_4DIsNV-CkzKUw)


Conclusion
----------


By following this guide, you have learned how to set up Flows in Directus, configure a Next.js API handler, and deploy it using Vercel. You also saw how to test the integration by creating, updating, and deleting data in Directus, with changes being reflected in your Algolia index. This setup ensures that our data remains synchronized across both platforms, helping us have a good search experience. Keep coding, and keep using Directus!!

