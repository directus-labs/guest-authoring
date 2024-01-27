---
title: 'Using Directus with Django'
description: 'Django is one of the most popular python frameworks known for its battery included philosophy, it comes loaded with a lot of features that cuts down development time for  devs. In this tutorial, you will learn how to integrate django with Directus using a blog API. At the end of this tutorial, you will have an application that uses the django templating engine to display data from the API.
'
author:
  name: 'omu inetimi'
  avatar_file_name: 'add-to-directory'
---

## Introduction
Django is one of the most popular python frameworks known for its battery included philosophy, it comes loaded with a lot of features that cuts down development time for  devs. In this tutorial, you will learn how to integrate django with Directus using a blog API. At the end of this tutorial, you will have an application that uses the django templating engine to display data from the API.


## Before You Start
You will need:
- Python installed and a code editor on your computer.
- Directus project - Use the [quickstart guide](https://docs.directus.io/getting-started/quickstart) to create a project with the fields  title, content and author.
- Basic knowledge of python and django

## Project setup
To start, open your terminal  and create an env, this will serve as a virtual environment for all necessary dependencies for this particular project. It's recommended you use an env for every project to ensure your project is isolated from others.

```bash
python -m venv myenv
```
Then go ahead and activate the virtual environment:
- On Windows: myenv\Scripts\activate
- On macOS/Linux: source myenv/bin/activate


#### Installing Django
With the virtual environment activated, install Django within this env:

```bash
pip install django
```

This installs django within the virtual environment.

#### Initializing Django Project
Create a new Django project and move into the directory of the new project

```bash
django-admin startproject myblog
cd myblog
```

This command starts a new django project and loads up the boilerplate files within django

To fetch data from the Directus API, we will use the *requests* library. It is not included in Python’s standard library, so it needs to be installed separately:

```bash
pip install requests
```
We need it to be able to make http requests to external API’s.


#### Creating the Blog App:
Create a new app within your Django project to handle the blog functionality:
```bash
python manage.py startapp blog
```
This creates a new directory within our project containing all the necessary files for a django app


#### Updating INSTALLED_APPS:
Move to the settings.py file of your django project then add your new app 'blog' to the `INSTALLED_APPS` list.  It should look something like this:
```python
INSTALLED_APPS = [
    # ... other installed apps ...
    'blog',
]
```
###Configuring the Template Directory:
Ensure Django can locate your templates by setting up the `DIRS` in the `TEMPLATES` configuration of *settings.py*. It should include the path to your templates directory:

```python
TEMPLATES = [
    {
        # ... other settings ...
        'DIRS': [os.path.join(BASE_DIR, 'blog', 'templates')],
        # ... other settings ...
    },
]
```
That's all for the setup, let's proceed to creating the views.


## Creating Views 
Lets create a view to fetch and display blog posts.
Move to to the *blog/views.py* file and copy in this code:

```python
import requests
from django.shortcuts import render




# Create your views here.
def get_blog_posts(request):
   response = requests.get("https://django.directus.app/items/post")
   posts = response.json()['data']
   return render(request, 'blog/index.html',{'posts': posts})
```

This function fetches a list of blog posts from the directus API, it uses the *requests* library we installed earlier to make a `GET` request to our directus API endpoint ("https://django.directus.app/items/post"). It parses the json and extracts the data which it saves as the variable `posts`. it then renders the *blog/detail.html* template, passing the post data as context.

Lets now create the detailed view for each blog post:
Still in the middle *blog/views.py* file copy in this function below the `get_blog_posts` function:

```python
def get_blog_post_detail(request, post_id):
   response = requests.get(f"https://django.directus.app/items/post/{post_id}")
   post = response.json()['data']
   return render(request, 'blog/detail.html',{'post': post})

```

Just like the first function, this function makes a request to the directus API endpoint using the requests library but here the `post_id` is included in the URL so that It fetches data for a specific post. That request is saved in the `response` variable, the json is then  parsed  and saved in the `post` variable , which is rendered as context within the *detail.html* template we will create.

## URL Routing

#### Defining URL Patterns for the Views:
Let's map the urls to their respective view functions:
```python
from django.urls import path
from .views import get_blog_posts, get_blog_post_detail


urlpatterns = [
   path('', get_blog_posts, name='get_blog_posts'),
   path('<int:post_id>/', get_blog_post_detail, name='get_blog_post_detail'),
]
```
Here, we define the `urlpatterns`. The first path with an empty string ‘ ‘ corresponds to the root url for the blog app. This is connected to the `get_blog_posts function` from our views which displays a list of blog posts.

The second path (<int:post_id>) is what's called a dynamic url, it's used to capture an integer from the URL and pass it as the `post_id` argument to the `get_blog_post_detail` view function for displaying specific blog posts based on their id number.


##Creating and Configuring Templates
We’ll be creating 2 templates for the django app to display blog posts

First and foremost, in the blog directory, create a *templates* directory and within the *templates* directory create another directory named *blog*. The *blog/templates/blog* directory structure in Django is designed for template namespacing, which helps avoid naming conflicts between apps.

In the *blog/templates/blog* directory, create an *index.html* file.
Let now use django’s templating language to iterate over the posts and display each post's title.

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Blog Posts</title>
  
</head>
<body>
 <div>
    {% for post in posts %}
       <h2>
          <a href="{% url 'get_blog_post_detail' post.id %}"> {{post.title}} </a>
      </h2>

        <p>Author: {{ post.author }}</p>

<hr>
     {% endfor %}
 </div>
</body>
</html>
```
After the boilerplate html and within the div, there is a for loop that iterates over each post in the `posts` context variable. The post title is then wrapped in an anchor tag (<a>), making each title a clickable link that directs the post's detail view. The href attribute uses Django's {% url %} template tag to dynamically create the URL to the detail view.

In the same directory create the details.html page:
```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>details</title>
</head>
<body>
  <div>

         <h1>{{ post.title }}</h1>
         <p>{{ post.content }}</p>


 </div>
</body>
</html>
```
This template displays the details of a single blog post by accessing the post context variable to display specific attributes like `post.title` and `post.content`. You can also display other post details as needed but we will be limiting it to these for this tutorial.
Save all files and ensure there are no errors.


##Running and Testing
Start the Django Development Server:
In the terminal, navigate to your Django project root directory (where manage.py is located).
Run the server with:

```bash
python manage.py runserver
```
This will start  django’s development  server, usually at http://localhost:8000/

Open a web browser and go to http://localhost:8000/. This should display the list of blog posts fetched from the Directus API like so:
!image

You can now click on a blog post title. This should take you to the view of the post:


![image]

### Debugging:
- If you encounter issues, check the Django server's output in the terminal for error messages.
- Ensure that the Directus API endpoint is correct and accessible.
- Verify that your index.html and detail.html templates are correctly set up to display the data.

## Conclusion
This tutorial demonstrates how to create a blog application with Django and Directus, featuring a main page listing all posts and individual detail views for each post. This setup allows users to browse through summaries of blog posts and click on them to read more.




