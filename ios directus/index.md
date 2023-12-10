---
title: 'Getting Started with Directus and iOS'
description: 'Integrate Directus into your iOS app. Fetch and display dynamic content with ease using SwiftUI'
author: Harshpal Bhirth
name: ''
avatar_file_name: 'add-to-directory'
---

## Introduction
In this tutorial, you will learn how to configure an iOS project that interacts with the Directus API to fetch and showcase posts. Whether you're a seasoned iOS developer or just starting your journey, this step-by-step guide will provide you with the necessary instructions to set up Directus integration your SwiftUI-based iOS app.

## Before You Start

You will need:

1. To have Xcode installed on your macOS machine.

2. Basic knowledge of Swift programming language.

3. **A Directus Project:** Prepare a local Directus project for deployment. Follow the [Directus quickstart guide][quickstart] if you need to create a new project.





## Post

Create a new file in your Xcode project and name it Post.swift 

```swift
import SwiftUI

struct Post: Codable, Identifiable {
    var id: Int
    var title: String
    var content: String
    var status: String
    var image: String?
```

In the Post.swift file in Xcode create a Swift struct named Post to represent the data structure of the posts you'll be fetching from the Directus API. This struct should conform to the Codable and Identifiable protocols. 

```swift
   var imageURL: String? {
        guard let imageUUID = image else { return nil }
        return "https://ios-author-demo.directus.app/assets/\(imageUUID)"
    }
```
imageURL computed property: This property calculates the image URL by appending the image UUID to the base URL of your Directus instance's assets.

```swift 

func stripHTML() -> String {
        return content.replacingOccurrences(of: "<[^>]+>", with: "", options: .regularExpression, range: nil)
    }
}
```
This Swift function is designed to strip HTML tags from a string, effectively removing any HTML markup and leaving only the text content.

## ContentView

Create a ContentView.swift file if you haven't got one already 


The ContentView is a SwiftUI view that serves as the main interface for displaying a list of posts. Users can interact with individual posts, view truncated content, and access detailed information about a selected post. The view leverages SwiftUI's navigation and sheet presentation capabilities to create a seamless user experience.











```swift 
import SwiftUI

struct ContentView: View {
    @State private var posts = [Post]()
    @State private var selectedPost: Post? = nil

```

`@State private var posts = [Post]()`: This is a state property that holds an array of Post objects. The @State property wrapper indicates that the value can be modified and that changes to it should trigger a re-render of the corresponding view.

`@State private var selectedPost: Post? = nil`: Another state property that represents the currently selected Post object. It is initially set to nil because no post is selected at the start.


```swift
var body: some View {
        NavigationView {
            VStack(alignment: .leading) {
                List(posts) { post in
                    VStack(alignment: .leading) {
                        Text(post.title)
                            .font(.headline)
                        Text(post.stripHTML().prefix(100) + "...")
                            .font(.body)
                            .onTapGesture {
                                selectedPost = post
                            }
                    }
                }
                .sheet(item: $selectedPost) { post in
                    PostDetailView(selectedPost: $selectedPost, fetchPost: postAPIcall)
                }
            }
            .navigationTitle("Posts")
            .task {
                await fetchPosts()
            }
        }
    }
```

Body of the View:
The `body` property is the main content of the view. In SwiftUI, views are constructed by combining smaller views.

`NavigationView`: Wraps the entire content and provides a navigation interface.

`VStack`: A vertical stack that arranges its children views in a vertical line.

`List(posts) { post in ... }`: Creates a list of `Post` objects, where each post is represented by a vertical stack containing the post's title and a truncated version of its content.

Inside the list, a Text view displays the post's title, and another `Text` view displays a truncated version of the post's content. A `onTapGesture` is used to detect when a user taps on a post, setting the `selectedPost` property to the tapped post.

```swift
     .sheet(item: $selectedPost) { post in
                    PostDetailView(selectedPost: $selectedPost, fetchPost: postAPIcall)
                }
            }
            .navigationTitle("Posts")
            .task {
                await fetchPosts()
            }
        }
    }
```
`.sheet(item: $selectedPost) { post in ... }`: This is a sheet presentation that appears when a post is selected. It presents a detailed view of the selected post using `PostDetailView`.

The `PostDetailView` is presented as a sheet when a post is selected. It is passed the selected post as a binding parameter `($selectedPost)` and a function `(fetchPost)` for fetching additional post details (not defined in the provided code).



```swift
 .navigationTitle("Posts")
            .task {
                await fetchPosts()
            }
        }
    }
```


NavigationView Styling:
The `.navigationTitle("Posts")` sets the title of the navigation bar.
The `.task { await fetchPosts() }` is an asynchronous task that fetches posts when the view is first loaded. The `await keyword is used to perform asynchronous operations within the task.


## Fetchposts Function 

This function is responsible for asynchronously fetching posts from a remote API, decoding the JSON response, and updating the `@State` property `posts` with the retrieved data. Any errors encountered during this process are printed to the console.

```swift
  func fetchPosts() async {
        guard let url = URL(string: "https://ios-author-demo.directus.app/items/posts") else {
            print("Invalid URL")
            return
        }

        do {
            let (data, _) = try await URLSession.shared.data(from: url)
            let decoder = JSONDecoder()
            let result = try decoder.decode([String: [Post]].self, from: data)

            if let posts = result["data"] {
                self.posts = posts
            }
        } catch {
            print("Error: \(error)")
        }
    }
```

**Creating URL for API endpoint:**

`guard let url = URL(string: "https://ios-author-demo.directus.app/items/posts")`: Constructs a URL object for the API endpoint that contains the posts.

**Fetching data from the specified URL:**

`let (data, _) = try await URLSession.shared.data(from: url)`: Uses `URLSession.shared.data(from:)` to asynchronously fetch data from the specified URL. The resulting data is captured in the data variable.

**Decoding the fetched JSON data using JSONDecoder:**

`let decoder = JSONDecoder()`: Creates an instance of `JSONDecoder` to decode JSON data.

`let result = try decoder.decode([String: [Post]].self, from: data)`: Attempts to decode the JSON data into a dictionary where the keys are strings and the values are arrays of `Post` objects.


**Extracting the 'data' key from the decoded JSON:**

`if let posts = result["data"]: Checks if the "data"` key exists in the decoded JSON and retrieves the corresponding array of Post objects.

**Updating the @State property 'posts' with the fetched posts:**

`self.posts = posts`: If the "data" key exists, updates the `posts` array (which is a state property) with the fetched array of `Post` objects.

**Handling errors in case of any issues during the process:**

`catch { print("Error: \(error)") }`: Catches and prints any errors that may occur during the URL creation, data fetching, or decoding process.




## PostAPIcall Function

This function encapsulates the logic for making an asynchronous API call to fetch details of a specific post. It ensures uniqueness in the request by including a generated UUID as a query parameter. The fetched data is then decoded, and if successful, the `selectedPost` property is updated with the retrieved post details.

```swift

   func postAPIcall(postId: Int) async {
        let uuid = UUID().uuidString
        var components = URLComponents(string: "https://ios-author-demo.directus.app/items/posts/\(postId)")!
        components.queryItems = [URLQueryItem(name: "uuid", value: uuid)]

        guard let url = components.url else {
            print("Invalid URL")
            return
        }

        do {
            let (data, _) = try await URLSession.shared.data(from: url)
            let decoder = JSONDecoder()

            struct ApiResponse: Decodable {
                let data: Post
            }

            let result = try decoder.decode(ApiResponse.self, from: data)

            selectedPost = result.data
        } catch {
            print("Error: \(error)")

        }
    }
}
```


**Generating a unique UUID:**

`let uuid = UUID().uuidString`: Generates a unique identifier (UUID) as a string.

**Creating URL components with query parameters:**

`var components = URLComponents(string: "https://ios-author-demo.directus.app/items/posts/\(postId)")!`: Creates URL components with the base URL and incorporates the provided `postId`.
`components.queryItems = [URLQueryItem(name: "uuid", value: uuid)]`: Adds a query parameter (UUID) to the URL.

**Constructing the final URL:**

`guard let url = components.url else { print("Invalid URL") return }`: Ensures the final URL is valid and prints an error if not. 

**Fetching data from the specified URL:**

`let (data, _) = try await URLSession.shared.data(from: url)`: Asynchronously fetches data from the specified URL.

**Decoding the fetched JSON data using JSONDecoder:**

`let decoder = JSONDecoder(): Creates an instance of `JSONDecoder`.
`struct ApiResponse: Decodable { let data: Post }`: Defines a structure to represent the expected API response.

**Decoding the JSON data into the ApiResponse structure:**

`let result = try decoder.decode(ApiResponse.self, from: data)`: Attempts to decode the JSON data into the ApiResponse structure.

**Updating the selectedPost with the fetched post data:**

`selectedPost = result.data`: If decoding is successful, updates the selectedPost with the post data from the API response.

**Handling errors in case of any issues during the process:**
`catch { print("Error: \(error)") }`: Catches and prints any errors that may occur during the URL creation, data fetching, or decoding process.





## PostDetailView

Create a new file in your Xcode project and name it PostDetailView.swift

this SwiftUI view is designed to present detailed information about a selected post. It includes the post title, image (if available), content, a dismiss button to clear the selected post, and the post status. Additionally, it initiates an asynchronous task to fetch additional post details using the fetchPost function.

```swift
import SwiftUI

struct PostDetailView: View {
    @Binding var selectedPost: Post?
    var fetchPost: (Int) async -> Void

    var body: some View {
        if let post = selectedPost {
            VStack {
                Text(post.title)
                    .font(.headline)
                    .padding()

                if let imageURL = post.imageURL {
                    AsyncImage(url: URL(string: imageURL)) { phase in
                        switch phase {
                        case .success(let image):
                            image
                                .resizable()
                                .aspectRatio(contentMode: .fit)
                                .frame(maxHeight: 200)
                        case .failure(_):
                            Text("Failed to load image")
                        case .empty:
                            Image(systemName: "photo")
                                .resizable()
                                .aspectRatio(contentMode: .fit)
                                .frame(maxHeight: 200)
                                .foregroundColor(.gray)
                        default:
                            EmptyView()
                        }
                    }
                    .padding()
                }

                Text(post.stripHTML())
                    .font(.body)
                    .padding()

                Spacer()

                Button("Dismiss") {
                    selectedPost = nil
                }

                Text("Status: \(post.status)")
                    .font(.subheadline)
                    .foregroundColor(.gray)
                    .padding()
            }
            .task {
                await fetchPost(post.id)
            }
        }
    }
}

```

**Checking if a post is selected:**

`if let post = selectedPost { ... }`: The body of the view is only displayed if there is a selected post.

**Displaying post title:**

`Text(post.title)`: Displays the title of the selected post with a headline font.

**Displaying post image if available:**

Uses `AsyncImage` to asynchronously load and display the post image. It handles different loading phases and displays a placeholder or an error message if necessary.

**Displaying post content:**

`Text(post.stripHTML())`: Displays the content of the selected post, removing HTML tags.

**Dismiss button to clear the selectedPost:**

`Button("Dismiss") { selectedPost = nil }`: Provides a button to clear the selectedPost, effectively dismissing the detailed view.


**Displaying post status:**

`Text("Status: \(post.status)")`: Displays the status of the post in a subheadline font and gray color.

**Asynchronous task to fetch additional post details:**

`.task { await fetchPost(post.id) }`: Initiates an asynchronous task to fetch additional details for the selected post. The `await` keyword is used to wait for the asynchronous operation to complete.



## Summary
- The ContentView displays a list of posts and allows users to view detailed information about each post.
- The Post struct represents the data structure of posts.
- The fetchPosts function asynchronously fetches posts from the Directus API.
- The postAPIcall function fetches additional details for a specific post.
- The PostDetailView presents detailed information about a selected post.


By following this tutorial, you've learned how to integrate Directus API calls into your SwiftUI-based iOS app, providing a foundation for building more advanced features and interactions.





[quickstart]: https://docs.directus.io/getting-started/quickstart.html 'Directus Quickstart'

