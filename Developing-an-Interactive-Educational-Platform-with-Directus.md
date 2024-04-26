## Data Modeling and Permissions for a Leaning Management System

Digital education platforms engage students with different courses, instructors, and quizzes, with type of user needing different access to data in the platform. This tutorial focuses on the data modeling, roles, and permissions needed to build an education platform.

## Key Concepts

An interactive learning platform is expected to have the following key concepts which are:

1. Students and Instructors. 
2. Courses.
3. Lessons.
4. Quiz.
5. Enrollments.

In your Directus project, we will create a `courses` collection with a primary key type of UUID. Check all of the optional fields and complete setup. Your course collection should look like the image below:

![Course collection having only the default fields](https://lh7-us.googleusercontent.com/gpxJbPSFhEFGIdhmX52qhRu5WCJqndzQbFvZDZjfIblIz3JjQsetsP-gIGAHfGFQUelLh-DNgEWNxSAQgpBrUDaq2SVtgy1EsKo1QiT7XJgQXxPLX_gDegu2HZs09KBeDWfuq6d3oxWsNKP1oLf-Cdc)

Create the following additional fields:

- name: input interface
- description: WYSIWYG interface
- course_Code: dnput interface
- start_date and end_date: dateTime interfaces
- image: image interface
- slug: input interface

When creating the slug, enable the slugify option to ensure the value is kept URL-safe.

Create a `lesson` collection with a primary key type of UUID. Check all of the optional fields and complete setup. Create the following additional fields:

- name: input interface
- description: WYSIWYG interface
- number: Input interface with Integer type
- video_url: input interface
- content: WYSIWYG interface
- passing_score: Input interface with Integer type

### Quiz Collection

At the end of lessons we normally have quizzes or test. Let's go ahead and create a collection for this, name it Quiz to follow up in this tutorial. Our first and second fields will be name and description, go ahead and create those. Following up we will have questions, as good practice questions should have their collection, then linked to the quiz collection, this will be discussed later on in the blog.

Going further Time limit field, which will be a text, and passing score which will be text-interface but an integer, setting the minimum value to 0 and maximum to one hundred, or ten depending on your grade, in this blog we will leave the option 100 as seen in the image below.

![An image ](https://lh7-us.googleusercontent.com/3RWuxpL3dlE8OO3y9iKCt1yJOGOTt9_XA3rILELxhjMhI3IZn50A1wCXZXBcNnEiL6pPnuLANlFz61Yn3gzzR0VGuHiZFGNVcoOTJ7BdaNIbTDpqdno2hUAF65gFTcb0YIy228LRj5lEnGQli1MwrUk)

Lastly, we will create the question collection and for the field, we will be using the WYSIWYG interface only.

### Instructor Collection,

Every Course is supposed to have someone with experience to teach it. We will go ahead to create its collection, we will name the collection, and select the optional field as usual. In our collection we will have the; Instructor's Name, Email, Bio, Profile Picture or Avatar, and Courses Taught which will be linked to the Course collection.

For our name and email, we will be using the Text field, for the Bio we will be using the WYSIWYG field, and for the avatar, we will be using the image field.

All of these fields have previously been demonstrated, we can go ahead to create them. Your Instructor collection, should have the following fields, as seen in the image below;

### ![](https://lh7-us.googleusercontent.com/OQ8roD49L-w080wFaGX1OAlNPJz4pHKeE3L4VZt9ZdcxE1jXoO3JQI8kbc9TQv91WyZtax0cUvp_dyZDFrQ102PkIfPlQqp-ZrC8T04usmMkJ_AKCeXlo2JF_ZEpvb4Y4Bb-3XJMU7mz5iaaLHYVebo)

### Enrollment Collection

The enrollment would be kind of tricky as it deals with linking a few of the other collections, the fields will look like this:

-   Enrollment Date: Date Field 

-   Status: Text  Field 

-   Grades/Progress: Text or Numeric Field

We will go ahead to create the enrollment date, status, and grades placing them in their proper fields.

## Creating Relationships

The `courses` collection can have a one-to-many relationship with the `lessons` collection. This means that one course can have multiple lessons associated with it.

In the `courses` collection, create a One to Many field called `lessons`. The related collection will be `lessons`, and the foreign key, which is our key inside the related collection will be `course`.

![An Image showing the selection of lessons as a related collection and courses as the key](https://lh7-us.googleusercontent.com/vlz0BeZQYgajzhOUO_3srP-lzk6ZhbeU5Ex0NwdhlKAYJH7RtbOciIJSzMnkEMdRCyVhXDVu5MxobZeNpo339d3yPJRPSqPKoJB5hLHpPCC5cpoa9zkm-7fjB5cLNxDJZolEFC_BJ0dGOoAcF54OfY4)

To make this easier to select, set the display template to the `name` variable, as seen below:

![](https://lh7-us.googleusercontent.com/jC-HlWxNsjjZFMYTTekHtok9GjkTXJUAt5czDdWOVWilH7UIQU3_wVv0yItmuJi7hBqIoN4lQzb5n42lO0-LWOEsPTBkRkBFWgssWBPtjBys79VBospZKaMIvS7XKfApVGxoxeu6RVDM1Xm3-iOwDS4)

Also create the folowing One to Many fields:

- We should do the same for Lesson and quiz. In the lesson collection create a field with the a related colelction of quiz. 
- In the quizzes collection, create a field with the a related collection of questions.

Also for Many to Many relationships:

Like an instructor could take many courses, and a course could have more than one instructor. We will have to go to the courses or instructors collection, it could be any actually, in our case we wiil go to instructors, just because we  haven't used it yet. We will use the many to many relationship, our related collection will be courses;

![Many to Many relationship between Instructors and Courses](https://lh7-us.googleusercontent.com/TBYw6vogrDkTFBk6dlPUTT726VnXzpd2hB6-LgNaUCK_yT6EzgdKIf58gvX5a-Sn_mSu93ggqRTAus42z1lfFEM5mUHRmSsuu1Yh1a_uhisu-OjazkAe7pJ8DIywo9864UjQeRWcNSIklyNPGioc-jM)


If we go back to the data model Directus will have created a junction collection on your behalf. Let's take a look at all our collections below:

![](https://lh7-us.googleusercontent.com/X4LzpJXQn9PDZeUdfYbVp9tyvHrFc0OEJnKJvfeS7BoevGZdpczYmidUcbbxdN0GjhgDyIBQkBRHN57OKxEZr3oqPn8MOCdkYzf2y7PdvkHtDPLzwzK-q5rV-BKwWQPJo2TwEtGmlDP1AH7GWYIVncw)

Create 3 courses to test with, and then test accessing it via API by navigating to `https://your-directus-url/items/courses`. You would normally see the same data you authored in the Data Studio as JSON, except you don't. Let's talk about why.

Your request to access data via the Items API provides no authentication details, meaning it is a public request. The public role has no permission to do this, now this takes us to the next section, where we will talk about roles and permissions.

## Roles and Permissions

You can configure roles and their permissions in the Access Control area of the settings module. 

You want to be careful with those you give administrator roles to, as it has access to administer all data, and you will want to restrict the public role from certain things, like courses enrollment, because you may only want authenticated users to access them.

Each role has a set of permissions for each collection which describe the ability to see, create, edit, and delete content. Using Directus, you can also apply custom permissions which are more granular than a "yes" or "no", but perhaps only allow instructors to edit their own courses, and students to see those that they are enrolled in. 

Let's create permissions for the students and instructors.

### Student Role:

### ![An image representing permissions in student role](https://lh7-us.googleusercontent.com/nO2Y5acNRKDoHRe7K6XKbccWqD2Y6sUxiSMuAHiBO7W0IHrhrnLaHdb0oJEVmDvhfxmvo8mSgRT14EZv1YvNnd_qkNQx6jESfWelh6Z6QnpkKZc7JEL5dGzPTkcT-5t9YTKxnfVvkU962eiSZe9FhNA)

AS seen in image above, Students should be permitted to;

-   View Courses: Students should be able to view all available courses. 

-   View Lessons: Students should be able to view lessons within their enrolled courses

-   View Quizzes: Students should be able to view quizzes within their enrolled courses.

- View Own Profile: Students should be able to view their own profile information. 

-   No creation/editing/deletion permissions: Students should not be able to create, edit, or delete courses, lessons, assignments, quizzes, or any other content.

- Custom permissions: Students should only  view published courses, lessons, and quizzes
    - Quiz: We will open up the read permission click custom permissions, and under item permissions, we will create a role using a dynamic variable, by clicking the add filter drop dropdown, we are going to go to the quiz questions field, and use the ' Quiz questions to equals `$CURRENT_USER.id`, and now they would only be able to see the quizzes that belongs to them alone. Kindly look at the image below, and do make sure you are in line.

![](https://lh7-us.googleusercontent.com/TsilbdWhcx2I1Oy_uNME4cO-BfJMBcHWmyNCZBtj-obb5NGCnfBsrFrSsP8stVrfz12xI2jm1MXIlK4W0ZhX4nk31YJL62EFbie0f59NYZe4K659T3gi5skIkA3FoT5ZN20jtjOJM87MWAgr8-6Q-OI)

- 
    - Courses: we navigate to the courses collection, the custom permissions, then item permissions, then we click the add filter dropdown under the rules, select status and let it be equal to published. And viola students can only see published courses, we will go ahead to do same for, lessons, and quiz.

![](https://lh7-us.googleusercontent.com/hZBD0NEPySG1pe8Y1hQzGAcf8h-QQJvr9qoEDiphvUaX6p0Mkag2TRs-KRPTOIVMYTlLMBjLZoUejxlv4a5SlIBPwtqLc19qXH943GJs3-bkd8ZbdggqOBi7J8K16seeEv0BLlQuoqJ3WUnqvlOPvbg)

### Instructor Role: 

Instructors should be permitted to;

-   Create/Edit/Delete Courses: Instructors should be able to create, edit, and delete courses. 

-   Create/Edit/Delete Lessons: Instructors should be able to create, edit, and delete lessons within the courses they are assigned to. 

-   Create/Edit/Delete Quizzes: Instructors should be able to create, edit, and delete quizzes within the courses they are assigned to. 

-   View All Data: Instructors should have access to view all data within the platform.

-   Limited administrative access: Instructors should not have access to manage users, roles, or other administrative settings.

![](https://lh7-us.googleusercontent.com/YGWsatZncTynyPnj8p5a9ANJzu3irWvKQ4S3btPUr6GUPJTIfHBLKYXhfSCwJuV8_kkHwNopI2mkRALEsYuik--jck8gLalu3-CGJrPFz_LGh4fqV290UCJs0TYEHFtY5ZKp0L8AB-tKJ3M2KdJOFio)

Now for the instructors, since they do have a little more hedge of control, we would want to insist on them having limited rights to do certain things.

We do want instructors to create a course, but we also would want them not to update other instructors' courses.

So in the instructors roles, we will go to instructor-courses collection, click edit, select  custom item, go to rules, click the filter drop down, click instructor -Id then set it to `$CURRENT_USER.id`. This should make only instructors who created certain courses be able to edit them.

![](https://lh7-us.googleusercontent.com/eszZASHeqR49PsGeZQF7dJwKeJ-2R27676PFkVcFGlZKLbe-xRoCrgBZW3lXTDBfLjjpATpWINEYTQRdFGToRD9zmQTefOwpDufFvWVyYNgQGkQThs6F6FZPR1Sj7uba5CFA6CHWSVUJOyDAm3ddEUk)


## Directus API's
Directus provides API_URL (`https://your-directus-url`) layer that can be used to build in the front end. To set up your project, you'll first need to install Directus SDK for easy querying. 

`npm install @directus/sdk`

Once your Directus instance is installed and running, you'll need to start by accessing your Directus instance through its URL. You might query your courses data via:

`https://your-directus-url/items/courses`

Below is a imaginary breakdown of how it should look, we will be considering API endpoints for students and instructors:


### Student: 
- GET `/users/:id` - Get current user information (requires authentication) 
- GET `/Courses` - Get all available courses 
- GET `/Courses/:id` - Get details of a specific course 
- GET `/Courses/Lessons` - Get all lessons for a course 
- GET `/Courses/:Courses_id/:Quiz_id` - Get details of a specific quiz 
- POST `/Courses/:id/Enrollment` - Enroll in a course (requires authentication) 


In building this API endpoints comes in handy:
- Student Login: The student logs in, and the front-end retrieves user information using the `/users/:id` endpoint.
- Check Courses: The front-end retrieves a list of available courses using the `/Courses` endpoint and displays them to the student.
- Enroll in Course: The student clicks to enroll in a course. The front-end sends a POST request to the `/Courses/Enrollment` endpoint.
- View Lessons: The student opens a course. The front-end retrieves lesson details using the `/Courses/lessons` endpoint and displays them.
- Attempt Quiz: The student starts a quiz. The front-end retrieves quiz details using the `/courses/Lessons/Quiz_id` endpoint and displays questions.

### Instructors:
Instructor API Endpoints:

-   `POST /api/Courses`: Allows instructors to create new courses(requires authentication).
-   `DELETE /api/Courses/:Course_id`: Deletes a course and associated content.
-   `GET /users/:id` - Get current user information (requires authentication)
-   `GET /Courses` - Get all courses (including unpublished ones)
-   `GET /Courses/:id` - Get details of a specific course
-   `PUT /Courses/:id` - Update an existing course (requires authentication)
-   `GET /Courses/:id/Lessons` - Get all lessons for a course (can include unpublished lessons)
-   `POST /Courses/:id/Lessons` - Create a new lesson for a course (requires authentication)
-   `PUT /Lessons/:id` - Update an existing lesson (requires authentication)
-   `DELETE /Lessons/:id` - Delete a lesson (requires authentication)
-   `GET /Courses/:id/Quiz` - Get all quizzes for a course (can include unpublished quizzes)
-   `POST /Courses/:id/Quiz` - Create a new quiz for a course (requires authentication)
-   `PUT /Quiz/:id` - Update an existing quiz (requires authentication)
-   `DELETE /Quiz/:id` - Delete a quiz (requires authentication)

### Instructor Workflow Brief Example:

-  Create Course: The instructor creates a new course using the `/Courses` endpoint.
-  Develop Course Content: The instructor creates lessons and quizzes using the respective endpoints (`/Courses/Lessons` and `/Courses/Lesson/Quiz`).
-  Manage Content: The instructor can edit the course, lessons, and quizzes using the update endpoints (PUT /Courses/:id, PUT /Lessons/:id, PUT /Quiz/:id) before publishing.


## Conclusion
In this article, we have been able to go through the exercise of Data model and permissions for Interactive Educational platforms in directus, thank you for sticking this far, I really do appreciate. For further details refer to our [docs](https://docs.directus.io/), Keep using Directus!