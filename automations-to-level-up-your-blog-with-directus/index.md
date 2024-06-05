# 
---
title: '5 Automations to Level-Up Your Blog with Directus'
description: 'This article shows five automation strategies that will level up you blog running on Directus'
author:
  name: 'Muhammed Ali'
  avatar_file_name: 'avatar'
---

## Introduction
Maintaining a blog requires more than just compelling content; some aspect of efficiency and automation may be required. 
This article shows five automation strategies that will level up you blog running on Directus, allowing you to focus more on content creation and less on the intricacies of blog management.

## Before You Start

You will need:

- [DeepL](https://www.deepl.com/) API key for Language translation.
- [OpenAI](https://openai.com/) key for SEO summary.

In your Directus project, create a new collection called `content` and `comment` with the following fields:
For `content`:

- `SEO_summary`: Textarea, string
- `comment`: one to many, comment
- `approved`: Radio Buttons, Choices: yes, no, Default Value: no

Do not add the fields to be translated like `title` or `detail` yet, because those will be stored in a separate collection. Follow this [guide](https://docs.directus.io/guides/headless-cms/content-translations.html) on how to set up your collection for translation.
For `comment`:

- `content`: many to one, content
- `comment`: Textarea, string

## 1. Automated Translation of New Posts
![translation of new post](ai-translation.png)

Expanding your blog's reach to non-English speaking audiences can be daunting. Automated translations ensure that every new post is available in multiple languages simultaneously, reaching a global audience requires content in multiple languages.
In this section, you will learn how to automate translation from English to French, then updating the article you include the translation. 
Here we will translate the detail section of the blog, but you can also use this method to create translation for the title in another flow.

## How to Implement
First, install the AI translator marketplace extension in Directus.
Go to the **Flows** tab and create a new flow, give it any name you like. 
Create a flow to trigger translation upon creating or updating a post and select the content collection. 
The trigger will only return the `key` of the content, but the whole post is needed to send to DeepL. Create a **Read Data** operation and give it full access permissions. On the “Content Translations” collection, access the following query:

```json
{
    "filter": {
        "_or": [
            {
                "_and": [
                    {
                        "content_id": {
                            "_eq": "{{$trigger.keys[0]}}"
                        }
                    },
                    {
                        "languages_code": {
                            "_eq": "en-US"
                        }
                    }
                ]
            },
            {
                "_and": [
                    {
                        "content_id": {
                            "_eq": "{{$trigger.key}}"
                        }
                    },
                    {
                        "languages_code": {
                            "_eq": "en-US"
                        }
                    }
                ]
            }
        ]
    }
}
```

If a content is updated, the key is found at `{{$trigger.keys[0]}}` and if a content is created, the key is found at `{{$trigger.key}}`
So basically the query is saying that the operation should filter for when `content_id` in Content Translations is equal to (`_eq`) `{{$trigger.keys[0]}}` and then grab the en-US version. Or filter for when `content_id` in Content Translations is equal to (`_eq`) `{{$trigger.key}}` and then grab the en-US version.
The output will be the article you just created or updated.
Now add the **AI Translation** operation with full access permission and put in your DeepL API key and select the plan. Put `{{$last[0].detail}}` in the Text form and then select the language.

Now we can save the French output by adding the **Create Data** operation on the **Content Translations** collection with full access permission. Use the payload:

```json
{
    "content_id": "{{$trigger.keys[0]}}",
    "languages_code": "fr-FR",
    "detail": "{{$last}}"
}
```

:::info Box title

Before attaching new operations, it is best practice to check what the payload output looks like.

:::

You can test this out by creating a new content. After a few seconds, the French version will be populated. You can do the same for the title.

![AI translated content](translated-content.png)

## 2. Automate Content Publishing Workflow on Approval
![Flow to publish article](publish-article.png)


As a blog manager, you're often juggling multiple tasks. By setting up an automated publishing workflow in Directus, once a post is reviewed and approved by an editor, it gets published immediately. This automation eliminates the back-and-forth of manual scheduling and reduces the risk of human error.
Since we are not working with frontend here, we can simulate publishing by returning the title and detail as output.

###How to Implement
Create a flow to trigger translation upon updating a post and select the content collection. 
Now add a **Read Data** operation attached to the Content Translations collection with the ID `{{$trigger.keys[0]}}`.  This will return the item of the content that was just updated. If you look at the payload, this data does not come with the value for `approved`, this is because it is saved in the `content` collection. So we need to use the **Read Data** operation again, but this time we will attach to the `content` collection with the ID `{{$trigger.keys[0]}}`.
Now use the Run Script operation to run the following JavaScript code:
```javascript
module.exports = async function(data) {
    // Initialize an empty result object
    let result = {};

    // Check if "approved" is "yes"
    if (data.item_read_rif6q.approved === "yes") {
        // Publish the article title and detail
        result = {
            title: data.item_read_6grfu.title,
            detail: data.item_read_6grfu.detail
        };
    } else {
        result = {
            status: "Not Approved for Publishing"
        };
    }
    
    return result;
};
```

In the code above, `item_read_rif6q` and `item_read_6grfu` are the keys for the Read Data operations, so yours might be different since they are automatically generated.
This is what your output will look like this when a content is approved:
```json
{
    "title": "Reasons why automating delivery could be a bad idea ",
    "detail": "Lack of Human Oversight:\nAutomated systems can lack the nuanced judgment that humans provide. Critical decisions might require human intervention to account for context, unexpected situations, or complex problem-solving...."
}
```
## 3. Configuring Alerts for New Comments
![comment alert](comment-alert.png)

As your blog grows, monitoring comments manually becomes impractical. Automated alerts ensure that you and your team are notified of new comments instantly, enabling prompt responses and fostering community engagement.
In this section, you will learn how to set up a flow to send comments email to a specified email.  

:::info Box title

If you are self-hosting a Directus instance, you will need to set up the [email service configuration](https://docs.directus.io/self-hosted/config-options.html#email).

:::
### How to Implement
Create a flow to trigger email upon creating a comment and select the `comment` collection. 
Now add a **Read Data** operation attached to the Content Translations collection with the ID `{{$trigger.payload.content}}`.  This will return the comment that was just created. Now attach the **Send Email** operation. In the To section input the email addresses you want the notification to go to.
**Subject**:  Comment for on article "{{get_article_comment.title}}"
**Type**: Markdown
**Body**: 

    Hello, there is a comment for the article "{{get_article_comment.title}}"
    > {{$trigger.payload.comment}}
![email alert](email-alert.png)

**4. Automatic SEO Summary Writing**

![AI SEO summary](seo-summary.png)


Writing SEO summaries can be time-consuming and repetitive. By leveraging the AI Writer marketplace extension, you can automatically generate keyword-rich summaries, improving your blog's SEO performance and freeing up time for more creative tasks.

### How to Implement
Install the AI Writer marketplace extension in Directus.
Create a flow to trigger translation upon creating a post and select the content collection. 
The trigger will only return the `key` of the content, but the whole post is needed to send to OpenAI. Create a **Read Data** operation and give it full access permissions. On the “Content Translations” collection, access the following query:

```json
{
    "filter": {
        "_and": [
            {
                "content_id": {
                    "_eq": "{{$trigger.key}}"
                }
            },
            {
                "languages_code": {
                    "_eq": "en-US"
                }
            }
        ]
    }
}
```

Now you can pass the content to OpenAI by creating an AI writer operation, input your OpenAI key, select the **GPT model** you want to use and **Prompt** as Create SEO Description. The Text will be gotten at `{{$last[0].detail}}`.
We can now update the content collection with the newly created SEO description using Update Data operation. The ID tag will be `{{$trigger.key}}`.
Now a creation in content will result in the generation of the SEO description.


## 5. Automate Content Promotion on Socials
![social post](social-post.png)

You will need to post your article on social media platforms for promotion. In this section you will learn how to send the post from SEO summary to Zapier where you will be able to automate publishishing to the platform of your choice.
We will use [Zap’s webhook URL](https://help.zapier.com/hc/en-us/articles/8496288690317-Trigger-Zaps-from-webhooks) so get one from Zapier.
### How to Implement
Create a flow to manually trigger on the Item page in the content collection. Now, create a **Read Data** operation, on the “Content” collection, use the ID at `{{$trigger.body.keys[0]}}`.
Finally create a Webhook/Request URL operation with POST method and paste in your zapier hook URL. Request Body will be `{{item_read_cx40l.SEO_summary}}`
Replace `item_read_cx40l` with the keys of the Read Data operations.

## Conclusion
By leveraging the capabilities of Directus, from automated translations to content promotions, you have the bility to operate your blog effectively. These five automation strategies not only streamline your workflow but also ensure that you cut down the actions you need to take between the writeing and pomotion process.

