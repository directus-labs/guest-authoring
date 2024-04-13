---
title: 'Auth with Directus and iOS'
description: '120-160 characters'
author:
  name: 'Harshpal Bhirth'
  avatar_file_name: 'add-to-directory'
---

## Introduction

In this tutorial, you will learn how to configure an iOS project that authenticates with Directus. You'll cover registering, logging in, viewing all posts from all users, creating a post, and editing and deleting posts from your user account.

## Before You Start

You will need:

1. To have Xcode installed on your macOS machine.
2. Knowledge of the Swift programming language.
3. A Directus project - follow our [quickstart guide](docs.directus.io/getting-started/quickstart) if you don't already have one.

## Setting Up Public Role

1. Navigate to `Settings`.
2. Select `Access Control`.
3. Click on the `Public Role`.
4. Access the `System Collection`.
5. Locate `Directus_users`.
6. Under the `Create` option, choose `Use Custom`.
7. Enable field permissions for `Email` and `Password` options.

## Creating A New Role 

1. On the Access Control page, click the plus button.
2. Name your new role, such as `iOS App User`.
3. Under the "Post" collection, enable `Create` and `Read`.
4. For `Write` and `DELETE`, select `Use Custom`.
5. Add Filter `user_created -> id Equals $CURRENT_USER.id` for both update and delete options.
6. This configuration ensures users can create and read all posts, but only update and delete their own posts.
7. Copy the Primary Key from the iOS App User role.
8. In the Public Role settings, under `Directus_users`, select `Use Custom` for the create option.
9. Click `Field Presets` and paste the Primary Key.

By following these steps, you'll set up a role in Directus that grants users of your iOS app the ability to view all posts but restricts editing and deleting permissions to posts associated with their authenticated user account.


## Content View
```swift

import SwiftUI

struct ContentView: View {
    @State private var showLoginView = false
    @State private var isLoggedIn = false
    @State private var accessToken: String?
    
    var body: some View {
        NavigationView {
            VStack {
                Spacer()
                
                if isLoggedIn {
                    NavigationLink(
                        destination: CreatePostView(accessToken: accessToken ?? ""),
                        label: {
                            Text("Create Post")
                                .font(.title)
                                .foregroundColor(.white)
                                .padding()
                                .background(Color.blue)
                                .cornerRadius(10)
                        })
                        .padding()
                    
                    NavigationLink(
                        destination: PostsView(isLoggedIn: $isLoggedIn, accessToken: $accessToken),
                        label: {
                            Text("Posts")
                                .font(.title)
                                .foregroundColor(.white)
                                .padding()
                                .background(Color.green)
                                .cornerRadius(10)
                        })
                        .padding()
                }
                
                Spacer()
                
                Button(action: {
                    showLoginView = true
                }) {
                    Text("Login")
                        .font(.title)
                        .foregroundColor(.white)
                        .padding()
                        .background(Color.green)
                        .cornerRadius(10)
                }
                .sheet(isPresented: $showLoginView) {
                    LoginView(isLoggedIn: $isLoggedIn, accessToken: $accessToken)
                }
                
                NavigationLink(
                    destination: UserRegisterView(isActive: .constant(false)),
                    label: {
                        Text("Register")
                            .font(.title)
                            .foregroundColor(.white)
                            .padding()
                            .background(Color.orange)
                            .cornerRadius(10)
                    })
                    .padding()
                
                Spacer()
            }
            .padding()
            .navigationTitle("Welcome")
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```
### ContentView Struct

Defines the main view of the application.

### State Properties:

Three `@State` properties are declared to manage the state of the view:

- **showLoginView**: Tracks whether the login view should be displayed.
- **isLoggedIn**: Tracks whether the user is logged in.
- **accessToken**: Stores the access token after successful login.

### Body Property:

The `body` property defines the view's layout and behavior.

- **NavigationView**: Wraps the content and provides navigation functionality.
- **VStack**: Vertical stack layout for arranging child views.
- **Spacer**: Flexible space to push content to the top and bottom.

### Conditional Views:

Display different views based on the user's login status:

- If logged in, show navigation links for creating posts and viewing posts.
- If not logged in, display a login button.

### Create Post NavigationLink:

Navigates to the `CreatePostView` when tapped.

### Posts NavigationLink:

Navigates to the `PostsView` when tapped.

### Login Button:

Triggers the display of the login view when tapped.

### LoginView Sheet:

Presents the login view as a modal sheet when `showLoginView` is true.

### Register NavigationLink:

Navigates to the user registration view.

### Closing VStack and NavigationView:

Ends the VStack and NavigationView sections.

## UserRegisterView
```swift
import SwiftUI

struct UserRegisterView: View {
    @Binding var isActive: Bool
    @State private var email: String = ""
    @State private var password: String = ""
    @State private var showAlert: Bool = false
    @State private var alertMessage: String = ""
    @Environment(\.presentationMode) var presentationMode
    
    var body: some View {
        VStack {
            TextField("Email", text: $email)
                .padding()
                .textFieldStyle(RoundedBorderTextFieldStyle())
            SecureField("Password", text: $password)
                .padding()
                .textFieldStyle(RoundedBorderTextFieldStyle())
            
            Button("Register") {
                registerUser()
            }
            .padding()
            .alert(isPresented: $showAlert) {
                Alert(title: Text("Error"), message: Text(alertMessage), dismissButton: .default(Text("OK")))
            }
        }
        .padding()
    }
    
    func registerUser() {
        guard !email.isEmpty, !password.isEmpty else {
            alertMessage = "Please enter both email and password"
            showAlert = true
            return
        }
        
        guard let url = URL(string: "https://directus.lws.io/users") else {
            showAlert(message: "Invalid URL")
            return
        }
        
        let body = [
            "email": email,
            "password": password
        ]
        
        guard let jsonData = try? JSONSerialization.data(withJSONObject: body) else {
            showAlert(message: "Failed to encode data")
            return
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = jsonData
        
        URLSession.shared.dataTask(with: request) { data, response, error in
            if let error = error {
                showAlert(message: error.localizedDescription)
                return
            }
            
            if let data = data {
                if let responseString = String(data: data, encoding: .utf8) {
                    print("Response: \(responseString)")
                    DispatchQueue.main.async {
                        presentationMode.wrappedValue.dismiss()
                    }
                } else {
                    showAlert(message: "Failed to parse response")
                }
            } else {
                showAlert(message: "No data received")
            }
        }.resume()
    }
    
    func showAlert(message: String) {
        alertMessage = message
        showAlert = true
    }
}

struct UserRegisterView_Previews: PreviewProvider {
    static var previews: some View {
        UserRegisterView(isActive: .constant(false))
    }
}
```
### UserRegisterView Struct:

Defines a view for user registration.

### Properties:

- **@Binding var isActive: Bool**: This is a binding variable that determines whether the view is active or not. Changes in this view propagate back to the parent.
- **@State private var email: String = ""**: State variable to hold the user's email address.
- **@State private var password: String = ""**: State variable to hold the user's password.
- **@State private var showAlert: Bool = false**: State variable to control whether to show an alert.
- **@State private var alertMessage: String = ""**: State variable to hold the message to be displayed in the alert.
- **@Environment(\.presentationMode) var presentationMode**: Environment variable to access the presentation mode, which allows the view to dismiss itself.

### Body View:

- **VStack**: A vertical stack that arranges its child views in a vertical line.
- **TextField**: Text field for entering the user's email.
- **SecureField**: Secure text field for entering the user's password.
- **Button**: Button to register the user.
- **.alert(isPresented: $showAlert)**: Modifier to present an alert when showAlert is true.

### registerUser Function:

- This function is called when the user taps the "Register" button.
- It first checks if the email and password fields are not empty. If they are empty, it sets the alertMessage and shows the alert.
- Then it constructs a URL for the user registration endpoint.
- It creates a dictionary body containing the email and password.
- Converts the body dictionary to JSON data.
- Constructs a POST request with the JSON data in the HTTP body.
- Executes the request asynchronously using URLSession.shared.dataTask.
- Handles the response or any error that occurs during the network request. If successful, it dismisses the view using the presentationMode.

### showAlert Function:

- This function sets the alertMessage and sets showAlert to true, triggering the display of the alert.


## Login
```swift
import SwiftUI

struct LoginData: Codable {
    let access_token: String
}

struct LoginResponse: Codable {
    let data: LoginData
}


struct LoginView: View {
    @State private var email: String = ""
    @State private var password: String = ""
    @State private var showAlert: Bool = false
    @State private var alertMessage: String = ""
    @Binding var isLoggedIn: Bool
    @Binding var accessToken: String?
    @Environment(\.presentationMode) var presentationMode
    
    var body: some View {
        NavigationView {
            VStack {
                TextField("Email", text: $email)
                    .padding()
                    .textFieldStyle(RoundedBorderTextFieldStyle())
                    .autocapitalization(.none)
                    .keyboardType(.emailAddress)
                
                SecureField("Password", text: $password)
                    .padding()
                    .textFieldStyle(RoundedBorderTextFieldStyle())
                
                Button(action: {
                    loginUser()
                }) {
                    Text("Login")
                        .padding()
                        .background(Color.blue)
                        .foregroundColor(.white)
                        .cornerRadius(8)
                }
            }
            .padding()
            .alert(isPresented: $showAlert) {
                Alert(title: Text("Error"), message: Text(alertMessage), dismissButton: .default(Text("OK")))
            }
        }
    }
    
    func loginUser() {
        guard let url = URL(string: "https://directus.lws.io/auth/login") else {
            showAlert = true
            alertMessage = "Invalid URL"
            return
        }
        
        let loginData = ["email": email, "password": password]
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.addValue("application/json", forHTTPHeaderField: "Content-Type")
        
        do {
            request.httpBody = try JSONSerialization.data(withJSONObject: loginData, options: [])
        } catch {
            showAlert = true
            alertMessage = "Error encoding login data"
            return
        }
        
        URLSession.shared.dataTask(with: request) { data, response, error in
            guard let data = data, let httpResponse = response as? HTTPURLResponse, error == nil else {
                showAlert = true
                alertMessage = error?.localizedDescription ?? "Unknown error"
                return
            }
            
            if (200..<300).contains(httpResponse.statusCode) {
                // Successful login
                if let loginResponse = try? JSONDecoder().decode(LoginResponse.self, from: data) {
                    accessToken = loginResponse.data.access_token
                    isLoggedIn = true
                    presentationMode.wrappedValue.dismiss()
                }
            } else {
                // Failed to login
                showAlert = true
                alertMessage = "Failed to login"
            }
        }.resume()
    }
}
```
### LoginData Struct:

Codable protocol indicates that instances of this type can be encoded and decoded, typically used for JSON encoding and decoding. 

### Properties:

- Declares a struct `LoginData` with two properties: `access_token` and `refresh_token`, both of type String.

### LoginResponse Struct:

**Purpose**: Defines a struct `LoginResponse` conforming to Codable, representing the structure of the response expected from the login API.

### Properties:

- It contains a single property `data` of type `LoginData`.

### LoginView Struct:

**Purpose**: Defines a SwiftUI view named `LoginView`.

### State Variables:

- Contains several `@State` variables to hold the user's email, password, whether to show an alert, and the alert message.
- Takes two `@Binding` variables: `isLoggedIn` to track whether the user is logged in and `accessToken` to hold the access token received upon successful login.
- Accesses the presentation mode environment variable to control the navigation flow.

### Body View:

- Sets up a `NavigationView` for navigation-related functionalities.
- Contains a `VStack` with two `TextField` views for email and password input, and a `Button` for logging in.
- The button triggers the `loginUser()` function when tapped.
- Applies a modifier to show an alert if `showAlert` is true.

### loginUser Function:

- Validates the URL for the login endpoint.
- Constructs the login data dictionary with email and password.
- Creates a POST request with JSON-encoded login data.
- Performs a data task to execute the request asynchronously.
- Checks for errors, response status, and decodes the response if the status code indicates success (200..<300).
- If successful, updates `accessToken` with the received access token, sets `isLoggedIn` to true, and dismisses the view.
- If unsuccessful, the app shows an alert with an error message.

## CreatePostView
```swift
import SwiftUI

struct CreatePostView: View {
    @State private var title: String = ""
    @State private var content: String = ""
    @State private var showAlert: Bool = false
    @State private var alertMessage: String = ""
    let accessToken: String
    
    var body: some View {
        VStack {
            TextField("Title", text: $title)
                .padding()
                .textFieldStyle(RoundedBorderTextFieldStyle())
            
            TextField("Content", text: $content)
                .padding()
                .textFieldStyle(RoundedBorderTextFieldStyle())
            
            Button(action: {
                createPost()
            }) {
                Text("Create Post")
                    .padding()
                    .background(Color.blue)
                    .foregroundColor(.white)
                    .cornerRadius(8)
            }
        }
        .padding()
        .alert(isPresented: $showAlert) {
            Alert(title: Text("Error"), message: Text(alertMessage), dismissButton: .default(Text("OK")))
        }
    }
    
    func createPost() {
        guard let url = URL(string: "https://directus.lws.io/items/posts") else {
            showAlert = true
            alertMessage = "Invalid URL"
            return
        }
        
        let postData = ["title": title, "content": content]
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.addValue("application/json", forHTTPHeaderField: "Content-Type")
        request.addValue("Bearer \(accessToken)", forHTTPHeaderField: "Authorization")
        
        do {
            request.httpBody = try JSONSerialization.data(withJSONObject: postData, options: [])
        } catch {
            showAlert = true
            alertMessage = "Error encoding post data"
            return
        }
        
        URLSession.shared.dataTask(with: request) { data, response, error in
            guard let httpResponse = response as? HTTPURLResponse, error == nil else {
                showAlert = true
                alertMessage = error?.localizedDescription ?? "Unknown error"
                return
            }
            
            if (200..<300).contains(httpResponse.statusCode) {
                print("Post created successfully")
            } else {
                showAlert = true
                alertMessage = "Failed to create post. Status code: \(httpResponse.statusCode)"
            }
        }.resume()
    }
}

struct CreatePostView_Previews: PreviewProvider {
    static var previews: some View {
        CreatePostView(accessToken: "realAccessToken")
    }
}
```
### CreatePostView Struct:

Defines a SwiftUI view named `CreatePostView`.

### State Variables:

- Contains several `@State` variables to hold the post's title, content, whether to show an alert, and the alert message.

### Parameters:

- Takes an `accessToken` parameter, which represents the access token needed to authenticate the user's request to create a post.

### Body View:

- Sets up a `VStack` containing two `TextField` views for inputting the post's title and content, respectively.
- Also contains a `Button` titled "Create Post", which triggers the `createPost()` function when tapped.
- Applies a modifier to show an alert if `showAlert` is true.

### createPost Function:

- Validates the URL for the endpoint where posts are created.
- Constructs the post data dictionary with title and content.
- Creates a POST request with JSON-encoded post data.
- Adds necessary headers, including the authorization header with the provided access token.
- Performs a data task to execute the request asynchronously.
- Checks for errors and response status.
- If the status code indicates success (between 200 and 299), it prints a success message.
- If the status code indicates a failure, it shows an alert with an appropriate error message.

## Token Manager 

``` swift 
import Foundation

struct TokenManager {
    static let accessTokenKey = "accessToken"

    static func saveToken(_ accessToken: String, expirationTime: Date) {
        UserDefaults.standard.set(accessToken, forKey: accessTokenKey)
    }
    
    static func getToken() -> (accessToken: String, expirationTime: Date)? {
        guard let accessToken = UserDefaults.standard.string(forKey: accessTokenKey) else {
            return nil
        }
        return (accessToken, Date())
    }
}
```

## TokenManager Struct:

Declares a struct named `TokenManager`, which handles operations related to access tokens.

### Properties:

- **static let accessTokenKey = "accessToken"**: Defines a static constant property `accessTokenKey` which holds the key used for storing and retrieving the access token in UserDefaults.

### Functions:

#### `saveToken` Function:

- **static func saveToken(_ accessToken: String, expirationTime: Date)**: This function is responsible for saving the access token and its expiration time.
    - It takes two parameters: `accessToken`, the access token string to be saved, and `expirationTime`, the expiration time of the token.
    - Inside the function, it uses `UserDefaults.standard.set(_:forKey:)` to save the access token to the user defaults with the key defined by `accessTokenKey`.

#### `getToken` Function:

- **static func getToken() -> (accessToken: String, expirationTime: Date)?**: This function retrieves the access token and its expiration time from UserDefaults.
    - It returns an optional tuple containing the access token and its expiration time, or nil if the token is not found in UserDefaults.
    - Inside the function, it retrieves the access token from UserDefaults using `UserDefaults.standard.string(forKey:)`. If the access token is found, it returns a tuple containing the access token and the current date as the expiration time.

### Usage:

- To save an access token, you would call `TokenManager.saveToken(_:expirationTime:)` with the access token string and its expiration time.
- To retrieve the access token, you would call `TokenManager.getToken()`, which returns an optional tuple containing the access token and its expiration time.

## PostView 
``` swift 
import SwiftUI

struct PostResponse: Codable {
    let data: [Post]
}

struct Post: Codable, Identifiable {
    let id: String
    let title: String
    let content: String
    let user_created: String
    let date_created: String
}

struct PostsView: View {
    @State private var posts: [Post] = []
    @Binding var isLoggedIn: Bool
    @Binding var accessToken: String?
    
    var body: some View {
        if isLoggedIn {
            List(posts, id: \.id) { post in
                NavigationLink(destination: PostDetailView(post: post, accessToken: accessToken)) {
                    VStack(alignment: .leading) {
                        Text(post.title)
                            .font(.headline)
                        Text(post.content)
                            .font(.subheadline)
                    }
                }
            }
            .onAppear {
                fetchPosts()
            }
        } else {
            Text("Please login to view posts")
                .onAppear {
                    fetchPosts()
                }
        }
    }
    
    func fetchPosts() {
        guard let token = accessToken, let url = URL(string: "https://directus.lws.io/items/posts") else {
            return
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = "GET"
        request.addValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        
        URLSession.shared.dataTask(with: request) { data, response, error in
            if let error = error {
                print("Error fetching posts: \(error.localizedDescription)")
                return
            }
            
            guard let data = data else {
                print("No data received")
                return
            }
            
            do {
                let decodedResponse = try JSONDecoder().decode(PostResponse.self, from: data)
                DispatchQueue.main.async {
                    self.posts = decodedResponse.data
                }
            } catch {
                print("Error decoding posts: \(error.localizedDescription)")
            }
        }.resume()
    }
}
```

### PostResponse Struct:

Declares a struct `PostResponse` conforming to `Codable`. It represents the structure of the response expected from the API when fetching posts.

### Properties:

- It contains a property `data` which is an array of `Post` objects.

### Post Struct:

Declares a struct `Post` conforming to `Codable` and `Identifiable`. It represents the structure of a post.

### Properties:

- Properties such as `id`, `title`, `content`, `user_created`, and `date_created`.

### PostsView Struct:

Defines a SwiftUI view named `PostsView`.

### State Variables:

- Contains several `@State` variables: `posts` to hold an array of posts, `isLoggedIn` to track whether the user is logged in, and `accessToken` to hold the access token.

### Body View:

- Checks if the user is logged in (`isLoggedIn`). If logged in, it displays a list of posts fetched from the server.
- Each post is displayed using a `NavigationLink`, which navigates to a `PostDetailView` when tapped.
- If not logged in, it displays a message prompting the user to log in.
- Calls `fetchPosts()` to retrieve posts when the view appears.

### fetchPosts Function:

- Fetches posts from the server using an HTTP GET request.
- It constructs a request with the provided access token in the authorization header.
- Performs a data task to execute the request asynchronously.
- Handles errors, data reception, and decoding of the response.
- If successful, it decodes the response into a `PostResponse` object and updates the `posts` array with the received posts on the main thread.

## PostDetailView 
```swift
import SwiftUI


struct PostDetailView: View {
    var post: Post
    var accessToken: String?
    @State private var showAlert = false
    @State private var isEditMode = false

    var body: some View {
        VStack {
            Text(post.title)
                .font(.title)
                .padding()
            Text(post.content)
                .padding()
            
            Button("Edit") {
                isEditMode = true
            }
            .sheet(isPresented: $isEditMode) {
                EditPostView(post: post, isEditMode: $isEditMode, accessToken: accessToken)
            }
        
            DeletePostView(postId: post.id, accessToken: accessToken, showAlert: $showAlert)
        }
    }
}
```

### PostDetailView Struct:

Defines a SwiftUI view named `PostDetailView`.

### Properties:

- **post**: Represents the post to be displayed in detail.
- **accessToken**: Optional access token for authentication purposes.
- **showAlert**: A boolean state variable to control the display of an alert.
- **isEditMode**: A boolean state variable to track whether the view is in edit mode.

### Body View:

- Displays the title and content of the post.
- Contains a "Edit" button, which toggles the `isEditMode` state when tapped.
- Utilizes a `sheet` modifier to present an `EditPostView` when `isEditMode` is true. This allows the user to edit the post.
- Renders a `DeletePostView` passing the post's ID, access token, and the `showAlert` state variable. This allows the user to delete the post.

### Button Action:

- When the "Edit" button is tapped, it sets `isEditMode` to true, triggering the presentation of the `EditPostView`.

### EditPostView:

- The `sheet` modifier presents an `EditPostView` when `isEditMode` is true. It passes the post, `isEditMode`, and `accessToken` to the `EditPostView`.

### DeletePostView:

- Renders a `DeletePostView`, passing the post's ID and access token. It also passes the `showAlert` state variable, allowing the `DeletePostView` to control the display of an alert if needed.

## EditPostView 
``` swift
import SwiftUI

struct EditPostView: View {
    var post: Post
    @Binding var isEditMode: Bool
    var accessToken: String?
    @State private var editedTitle: String
    @State private var editedContent: String
    
    init(post: Post, isEditMode: Binding<Bool>, accessToken: String?) {
        self.post = post
        _isEditMode = isEditMode
        _editedTitle = State(initialValue: post.title)
        _editedContent = State(initialValue: post.content)
        self.accessToken = accessToken
    }
    
    var body: some View {
        VStack {
            TextField("Title", text: $editedTitle)
                .padding()
                .textFieldStyle(RoundedBorderTextFieldStyle())
            
            TextField("Content", text: $editedContent)
                .padding()
                .textFieldStyle(RoundedBorderTextFieldStyle())
            
            Button("Save") {
                updatePost()
            }
        }
        .padding()
        .onAppear {
            editedTitle = post.title
            editedContent = post.content
        }
    }
    
    func updatePost() {
        guard let accessToken = accessToken else {
            print("Access token is missing")
            return
        }
        
        let postId = post.id
        
        guard let url = URL(string: "https://directus.lws.io/items/posts/\(postId)") else {
            print("Invalid URL")
            return
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = "PATCH"
        request.addValue("Bearer \(accessToken)", forHTTPHeaderField: "Authorization")
        request.addValue("application/json", forHTTPHeaderField: "Content-Type")
        
        let updateData: [String: Any] = [
            "title": editedTitle,
            "content": editedContent
        ]
        
        do {
            request.httpBody = try JSONSerialization.data(withJSONObject: updateData, options: [])
        } catch {
            print("Error encoding update data: \(error.localizedDescription)")
            return
        }
        
        URLSession.shared.dataTask(with: request) { data, response, error in
            if let error = error {
                print("Error updating post: \(error.localizedDescription)")
                return
            }
            
            if let httpResponse = response as? HTTPURLResponse {
                if httpResponse.statusCode == 200 {
                    DispatchQueue.main.async {
                        isEditMode = false
                    }
                } else {
                    print("Failed to update post. Status code: \(httpResponse.statusCode)")
                }
            }
        }.resume()
    }
}
```
### EditPostView Struct:

### Properties:

- **post**: Represents the post to be edited.
- **isEditMode**: A binding to a boolean indicating whether the view is in edit mode.
- **accessToken**: Optional access token for authentication.
- **editedTitle**: A state variable to hold the edited title of the post.
- **editedContent**: A state variable to hold the edited content of the post.

### Initializer:

- Initializes the view with the provided `post`, `isEditMode`, and `accessToken`.
- Initializes the `editedTitle` and `editedContent` state variables with the initial values of the post's title and content, respectively.

### Body View:

- Displays a `VStack` containing two `TextField` views for editing the post's title and content.
- Provides a "Save" button that triggers the `updatePost()` function when tapped.
- Uses the `onAppear` modifier to set the initial values of the `editedTitle` and `editedContent` when the view appears.

### updatePost() Function:

- Updates the post with the edited title and content.
- Constructs a PATCH request with the updated data and the access token in the authorization header.
- Performs a data task to execute the request asynchronously.
- Handles errors and response status codes.
- If successful (status code 200), sets `isEditMode` to false to exit the edit mode.

## Delete Post View 
``` swift 
import SwiftUI

struct DeletePostView: View {
    let postId: String
    let accessToken: String?
    @Binding var showAlert: Bool
    
    var body: some View {
        Button("Delete") {
            showAlert.toggle()
        }
        .padding()
        .alert(isPresented: $showAlert) {
            Alert(title: Text("Confirm"), message: Text("Are you sure you want to delete this post?"), primaryButton: .destructive(Text("Delete")) {
                deletePost()
            }, secondaryButton: .cancel())
        }
    }
    
    func deletePost() {
        print("Deleting post...")
        
        guard let accessToken = accessToken else {
            print("Access token is missing")
            return
        }
        
        guard let url = URL(string: "https://directus.lws.io/items/posts/\(postId)") else {
            print("Invalid URL")
            return
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = "DELETE"
        request.addValue("Bearer \(accessToken)", forHTTPHeaderField: "Authorization")
        
        URLSession.shared.dataTask(with: request) { data, response, error in
            if let error = error {
                print("Error deleting post: \(error.localizedDescription)")
                return
            }
            
            if let httpResponse = response as? HTTPURLResponse {
                if (200..<300).contains(httpResponse.statusCode) {
                    // Post deleted successfully
                    print("Post deleted successfully")
                } else {
                    print("Failed to delete post. Status code: \(httpResponse.statusCode)")
                }
            }
        }.resume()
    }
}
```
### DeletePostView Struct:

This is the main SwiftUI view struct. It has several properties and a body.

### Properties:

- **postId**: Represents the ID of the post to be deleted.
- **accessToken**: A string representing the access token required for authorization.
- **showAlert**: A binding to a boolean value indicating whether to show an alert for confirming the deletion.

### Body:

- This is where the layout of the view is defined using SwiftUI components.
- The body consists of a single `Button` labeled "Delete". When tapped, it toggles the `showAlert` boolean to true, which triggers the display of an alert for confirming the deletion.

### Alert:

- This is presented using the `.alert` modifier, which shows an alert when `showAlert` is true.
- The alert presents a confirmation message asking the user if they are sure they want to delete the post.
- It provides two buttons: a primary button labeled "Delete" (with a destructive style) and a secondary button labeled "Cancel".
- If the user confirms deletion by tapping "Delete", the `deletePost()` function is called.

### deletePost Function:

- This function is responsible for deleting the post from the server.
- It first prints a message indicating that the post deletion process has started.
- It checks if the access token is available.
- Then it constructs a URL for deleting the post based on the post's ID.
- It creates a `URLRequest` object with the appropriate HTTP method (DELETE) and headers (including the authorization header with the access token).
- It makes an asynchronous URLSession data task to perform the request.
- Upon receiving a response, it checks if the request was successful (HTTP status code in the range 200-299). If successful, it prints a success message. Otherwise, it prints an error message.

