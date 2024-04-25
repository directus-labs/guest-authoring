---
title: 'Getting Started with Directus and Flutter'
description: 'Learn how to integrate Directus with Angular. You will store, retrieve, and use global metadata such as the site title, create new pages dynamically based on Directus items.'
author:
  name: 'Clara Ekekenta'
  avatar_file_name: './clara-ekekenta.avif'
---

## Introduction
Flutter is an open-source mobile app development framework created by Google. In this tutorial, you will learn how to build an application using Directus as a Headless CMS. You will store, retrieve, and use global metadata such as the site title, create new pages dynamically based on Directus items, and build a blog.

## Before You Start
You will need:

- Flutter SDK: Follow the official [Flutter installation guide](https://docs.flutter.dev/get-started/install) for your operating system (Windows, macOS, or Linux). This will also install the Dart programming language, which is required for Flutter development.
- A Directus project - [follow our quickstart guide](https://docs.directus.io/getting-started/quickstart) if you don't already have one.
-  A code editor installed.
- Knowledge of Dart.

## Initialize Project

On your terminal, navigate to the directory where you want to create your project, and run the following command:

```bash
flutter create my_directus_app
```

Navigate to the project directory, after the project has been created and run the application with the command:

```bash
cd my_directus_app && flutter run
```

This will launch the app on an emulator or connected device. If everything is set up correctly, you should see the default Flutter app running.

## Set up requests/env vars/anything needed to call Directus
Next, let's install the dependencies required to run your application. You'll need to install the following dependencies:
- **http**: A package for making HTTP requests.
- **flutter_dotenv**: A package for loading environment variables from a .env file.
- **flutter_html**: A package to convert WYSIWYG contents to HTML.
    
Add these dependencies to your `pubspec.yaml` file under the dependencies section:

```
dependencies:
  http: ^0.13.5
  flutter_dotenv: ^5.0.2
  flutter_html: ^3.0.0-alpha.6
```

Then create an `assets/.env` file in the root directory of your Flutter project. This file will store your Directus API URL and any other environment variables you might need.

```
DIRECTUS_API_URL=https://your-directus-instance.com
```

Replace `https://your-directus-instance.com` with the actual URL of your Directus instance.

Then in your `main.dart` file, import the `flutter_dotenv` package and load the environment variables:

```
import 'package:flutter_dotenv/flutter_dotenv.dart';

Future main() async {
  await dotenv.load(fileName: ".env");
  runApp(MyApp());
} 
```
    
## Using Global Metadata and Settings
Directus provides a way for you to store global metadata and settings, which can be accessed and used throughout your application. To use the global metadata and settings, navigate to **Settings -> Data Model** and create a new collection called `global`. Check the Singleton option box to 'Treat as a single object' because this collection will have just a single entry containing global application metadata and settings. Create the following fields in the your `global` collection:

- **title**: text input
- **description**: text input

    
Then navigate to the content module, select the global collection, enter information in the `title` and `description` fields, and hit save.
    
![creating a global collection](./Screenshot%202024-04-16%20at%2017.36.50.png)

Then, navigate to **Settings -> Access Control -> Public** and give public read access to the `global` collection.
    
Next, we'll setup a `DirectusService` class to retrieve all the global settings and use them in your project Create a new file named `directus_service.dart` in your `lib` directory and add the code snippets:
    
```
import 'dart:async';
import 'dart:convert';
import 'package:http/http.dart' as http;
import 'package:flutter_dotenv/flutter_dotenv.dart';

class DirectusService {
  final String _baseUrl = dotenv.env['DIRECTUS_API_URL']!;

   Future<Map<String, dynamic>> getGlobalMetadata() async {
    final response = await http.get(Uri.parse('$_baseUrl/global'));
    if (response.statusCode == 200) {
      return jsonDecode(response.body)['data'];
    } else {
      throw Exception('Failed to load global metadata');
    }
  }
}
```
    
In the above code, we created a method to fetch the global metadata settings from Directus.
    
Now import the `DirectusService` class and use it to retrieve global settings and metadata:
    
```
import 'package:flutter/material.dart';
import 'services/directus_service.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';

Future main() async {
  await dotenv.load(fileName: ".env");
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  final DirectusService _directusService = DirectusService();

  MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
      future: Future.wait([
        _directusService.getGlobalMetadata(),
      ]),
      builder: (context,
          AsyncSnapshot<List<Map<String, dynamic>>> settingsSnapshot) {
        if (settingsSnapshot.connectionState == ConnectionState.waiting) {
          return const CircularProgressIndicator();
        } else if (settingsSnapshot.hasError) {
          return Text('Error: ${settingsSnapshot.error}');
        } else {
          final metadata = settingsSnapshot.data![0];

          return MaterialApp(
            title: metadata['title'],
            theme: ThemeData(
              primarySwatch: Colors.blue,
            ),
            home: Scaffold(
              appBar: AppBar(
                title: Text(metadata['title'] ?? 'My App'),
              ),
              body: Center(
                child:
                    Text(metadata['description'] ?? 'No description provided'),
              ),
            ),
          );
        }
      },
    );
  }
}
```

In the above code, we use a `FutureBuilder` to fetch the global metadata from Directus. Once the data is loaded, we use it throughout your application for the app title, and description.
    
![Rendering the content fron Directus global collection](./Screenshot%202024-04-16%20at%2018.27.02.png)

## Creating Pages With Directus
Directus allows you to create and manage pages, which you can then use in your Flutter app. To begin, create a new collection named `pages`. The Primary ID Field should be a "Manually Entered String" named `slug`, and you should include the following fields in your pages collection:

- **title**: text input field
- **content**: WYSIWYG input field

Then in your Access Control settings, give the Public role read access to the pages collection.
    
![creating the pages collection](./Screenshot%202024-04-17%20at%2009.12.20.png)

Now add a new method to fetch pages in your `DirectusService` class from Directus in the `directus_service.dart` file:
    
```
  ...
  Future<Map<String, dynamic>>  getPages() async {
    final response = await http.get(Uri.parse('$_baseUrl/pages'));
    if (response.statusCode == 200) {
      return jsonDecode(response.body)['data'];
    } else {
      throw Exception('Failed to load pages');
    }
  }
  ...
```
Next, create a page widget to display a single page using the data returned from the page collection. Create a `screens` folder in the `lib` directory. In the screens folder, create a `home_screen.dart` file and add the following code snippet:
    
```
import 'package:flutter/material.dart';
import 'package:flutter_html/flutter_html.dart';

class PageWidget extends StatelessWidget {
  final Map<String, dynamic> page;

  const PageWidget({
    super.key,
    required this.page,
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(page['title']),
      ),
      body: SingleChildScrollView(
        child: Padding(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              const SizedBox(height: 16),
              Html(
                data: page['content'],
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```
This will render the content of your `pages` collection and use the `flutter_html` package to convert the WYSIWYG content to HTML. Update the the code in your `main.dart` file to use the page widget:

```
import 'package:flutter/material.dart';
import 'services/directus_service.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';
import 'screens/home_screen.dart';

Future main() async {
  await dotenv.load(fileName: ".env");
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  final DirectusService _directusService = DirectusService();

  MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
      future: Future.wait([
        _directusService.getGlobalMetadata(),
      ]),
      builder: (context,
          AsyncSnapshot<List<Map<String, dynamic>>> settingsSnapshot) {
        if (settingsSnapshot.connectionState == ConnectionState.waiting) {
          return const CircularProgressIndicator();
        } else if (settingsSnapshot.hasError) {
          return Text('Error: ${settingsSnapshot.error}');
        } else {
          final metadata = settingsSnapshot.data![0];

          return MaterialApp(
            title: metadata['title'],
            theme: ThemeData(
              primarySwatch: Colors.blue,
            ),
            home: FutureBuilder<Map<String, dynamic>>(
              future: _directusService.getPages(),
              builder: (context, pagesSnapshot) {
                if (pagesSnapshot.connectionState == ConnectionState.waiting) {
                  return const CircularProgressIndicator();
                } else if (pagesSnapshot.hasError) {
                  return Text('Error: ${pagesSnapshot.error}');
                } else {
                  final pages = pagesSnapshot.data!;
                  return pages.isNotEmpty
                      ? PageWidget(
                          page: pages,
                        )
                      : const Text('No pages found');
                }
              },
            ),
          );
        }
      },
    );
  }
}
```

![Showing the contents from the pages collection in flutter application](./Screenshot%202024-04-17%20at%2017.41.14.png)

  
## Creating Blog Posts With Directus

Similar to creating pages, you can also create and manage blog posts using Directus CMS. To do that, create a new collection named posts with the following fields:
  
- **slug**: Primary key field, Manually entered string
- **title**: Text input field
- **content**: WYSIWYG input field
- **status**: Dropdown filed
- **date_created**: Datetime field
    
![Creating the posts collection in Directus CMS](./Screenshot%202024-04-17%20at%2011.13.45.png)

Now add a couple of blog data to the `posts` collection.
    
![Adding entries to Directus collection](./Screenshot%202024-04-17%20at%2011.26.32.png)


## Create Blog Post Listing
Add a new method to your `DirectusService` class to fetch blog posts from Directus:
    
```
 ...
 Future<List<dynamic>> getBlogPosts() async {
    final response = await http.get(Uri.parse('$_baseUrl/posts'));
    if (response.statusCode == 200) {
      return jsonDecode(response.body)['data'];
    } else {
      throw Exception('Failed to load blog posts');
    }
  }
 ...
```
    
Then update the code in your `lib/screens/home_screen.dart` file to render the blog posts in the `PageWidget`:
    
```
import 'package:flutter/material.dart';
import 'package:flutter_html/flutter_html.dart';

class PageWidget extends StatelessWidget {
  final Map<String, dynamic> pages;
  final List<dynamic> blogPosts;

  const PageWidget({
    super.key,
    required this.pages,
    required this.blogPosts,
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(pages['title']),
      ),
      body: SingleChildScrollView(
        child: Padding(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              const SizedBox(height: 16),
              Html(
                data: pages['content'],
              ),
              const SizedBox(height: 32),
              Text(
                'Blog Posts',
                style: Theme.of(context).textTheme.headline6,
              ),
              const SizedBox(height: 16),
              ListView.builder(
                shrinkWrap: true,
                physics: const NeverScrollableScrollPhysics(),
                itemCount: blogPosts.length,
                itemBuilder: (context, index) {
                  final blogPost = blogPosts[index];
                  return BlogPostItem(blogPost: blogPost);
                },
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class BlogPostItem extends StatelessWidget {
  final dynamic blogPost;

  const BlogPostItem({
    super.key,
    required this.blogPost,
  });

  @override
  Widget build(BuildContext context) {
    return Container(
      margin: const EdgeInsets.only(bottom: 16.0),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(
            blogPost['title'],
            style: Theme.of(context).textTheme.labelLarge,
          ),
          const SizedBox(height: 8),
          Html(
            data: blogPost['content'],
          ),
        ],
      ),
    );
  }
}`
```
    
The `PageWidget` accepts `blogPost` which are the blog posts from Directus as a required parameter. So you have to update the code in your `main.dart` file to pass it from the `DirectusService` class instance:
    
```
import 'package:flutter/material.dart';
import 'services/directus_service.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';
import 'screens/home_screen.dart';

Future main() async {
  await dotenv.load(fileName: ".env");
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  final DirectusService _directusService = DirectusService();

  MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
      future: Future.wait([
        _directusService.getGlobalMetadata(),
        _directusService.getBlogPosts(),
      ]),
      builder: (context, AsyncSnapshot<List<dynamic>> settingsSnapshot) {
        if (settingsSnapshot.connectionState == ConnectionState.waiting) {
          return const CircularProgressIndicator();
        } else if (settingsSnapshot.hasError) {
          return Text('Error: ${settingsSnapshot.error}');
        } else {
          final metadata = settingsSnapshot.data![0];
          final blogPosts = settingsSnapshot.data![1];
          return MaterialApp(
            title: metadata['title'],
            theme: ThemeData(
              primarySwatch: Colors.blue,
            ),
            home: FutureBuilder<Map<String, dynamic>>(
              future: _directusService.getPages(),
              builder: (context, pagesSnapshot) {
                if (pagesSnapshot.connectionState == ConnectionState.waiting) {
                  return const CircularProgressIndicator();
                } else if (pagesSnapshot.hasError) {
                  return Text('Error: ${pagesSnapshot.error}');
                } else {
                  final pages = pagesSnapshot.data!;
                  return pages.isNotEmpty
                      ? PageWidget(pages: pages, blogPosts: blogPosts)
                      : const Text('No pages found');
                }
              },
            ),
          );
        }
      },
    );
  }
}
```
![Display the contents fron the posts collection](./Screenshot%202024-04-17%20at%2018.06.09.png)

## Create Blog Post Single
Next, create a new file called `post_single.dart` file in the `lib/screens` folder. Then create a `BlogPostWidget`:

```
import 'package:flutter/material.dart';
import 'package:flutter_html/flutter_html.dart';

class BlogPostWidget extends StatelessWidget {
  final Map<String, dynamic> post;

  const BlogPostWidget({super.key, required this.post });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(post['title']),
      ),
      body: SingleChildScrollView(
        child: Padding(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
             const SizedBox(height: 8),
            Html(
              data: post['content'],
            ),
            ],
          ),
        ),
      ),
    );
  }
}
```
    
The `BlogPostWidget` serves as the single blog post view. When a user clicks on a blog post from the listing, the app navigates to this widget, displaying the full content of the selected post.
    
    
## Add Navigation
Now update the `BlogPostItem` class in the `lib/screens/home_screen.dart` to add navigation to the project:
```
class BlogPostItem extends StatelessWidget {
  final dynamic blogPost;

  const BlogPostItem({
    super.key,
    required this.blogPost,
  });

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () {
        Navigator.push(
          context,
          MaterialPageRoute(
            builder: (context) => BlogPostWidget(post: blogPost),
          ),
        );
      },
      child: Container(
        margin: const EdgeInsets.only(bottom: 16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              blogPost['title'],
              style: Theme.of(context).textTheme.labelLarge,
            ),
          ],
        ),
      ),
    );
  }
}
```
With the above code, when the user taps on the `BlogPostItem`, it triggers the `onTap` callback function. Inside this function, we use `Navigator.push` to navigate to a new screen. We create a `MaterialPageRoute`, defining the widget to be displayed on the new screen as `BlogPostWidget`. Also, we pass the `blogPost` data as a parameter to the `BlogPostWidget` widget. This allows us to display detailed information about the selected `blog` post on the new screen.
    
![Navigating to the blog single page](./Screenshot%202024-04-17%20at%2018.27.22.png)

## Summary
Throughout this tutorial, you've learned how to build a CRUD application using Directus and Flutter. We started by creating a new project, set up environment variables and everything we need to call Directus. Then we went further to create pages, and posts collections in Directus and integrate the Flutter application with Directus by making API requests to perform CRUD operations.

Now that you've acquired this knowledge, what would you like to build next with the Directus CMS? Perhaps you can improve on this project by adding authentication to protect the application, and form validation/serialization. To learn more about Dirctus and other interesting features it provides, visit the [official documentation](https://docs.directus.io/).