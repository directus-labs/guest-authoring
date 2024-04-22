## Data Modeling and Permissions for a Leaning Management System

Digital education platforms engage students with different courses, instructors, and quizzes, with type of user needing different access to data in the platform. This tutorial focuses on the data modeling, roles, and permissions needed to build an education platform.

## Key Concepts

This platform has several key concepts:

1. Courses are...
2. Students and Instructors are...
3. Lessons are...
4. Quizzes are...

In your Directus project, create a `courses` collection with a primary key type of UUID. Check all of the optional fields and complete setup. Your collection should look like this:
![](https://lh7-us.googleusercontent.com/gpxJbPSFhEFGIdhmX52qhRu5WCJqndzQbFvZDZjfIblIz3JjQsetsP-gIGAHfGFQUelLh-DNgEWNxSAQgpBrUDaq2SVtgy1EsKo1QiT7XJgQXxPLX_gDegu2HZs09KBeDWfuq6d3oxWsNKP1oLf-Cdc)

Create the following additional fields:

- Name: Input interface
- Description: WYSIWYG interface
- Course_Code: Input interface
- Start_Date and End_Date: DateTime interfaces
- Course_Image: Image interface
- slug: Input interface

When creating the slug, enable the slugify option to ensure the value is kept URL-safe.

Create a `lesson` collection with a primary key type of UUID. Check all of the optional fields and complete setup. Create the following additional fields:

- number: Input interface with Integer type
- content: WYSIWYG interface
- passing_score: Input interface with Integer type

### Quiz Collection

At the end of lessons we normally have quizzes or test. Let's go ahead and create a collection for this, name it Quiz to follow up in this tutorial. Our first and second fields will be name and description, go ahead and create those. Following up we will have questions, as good practice questions should have their collection, then linked to the quiz collection, this will be discussed later on in the blog.

Going further Time limit field, which will be a text, and passing score which will be text but an integer, setting the minimum value to 0 and maximum to one hundred, or ten depending on your grade, in this blog we will leave it 100.

![](https://lh7-us.googleusercontent.com/3RWuxpL3dlE8OO3y9iKCt1yJOGOTt9_XA3rILELxhjMhI3IZn50A1wCXZXBcNnEiL6pPnuLANlFz61Yn3gzzR0VGuHiZFGNVcoOTJ7BdaNIbTDpqdno2hUAF65gFTcb0YIy228LRj5lEnGQli1MwrUk)

Lastly, we will create the question collection and for the field, we will be using the WYSIWYG option.

### Instructor Collection,

Every Course is supposed to have someone with experience to teach it. We will go ahead to create its collection, we will name the collection, and select the optional field as usual. In our collection we will have the; Instructor's Name, Email, Bio, Profile Picture or Avatar, and Courses Taught which will be linked to the Course collection.

For our name and email, we will be using the Text field, for the Bio we will be using the WYSIWYG field, and for the avatar, we will be using the image field.

All of these fields have previously been demonstrated, we can go ahead to create them. 

### ![](https://lh7-us.googleusercontent.com/OQ8roD49L-w080wFaGX1OAlNPJz4pHKeE3L4VZt9ZdcxE1jXoO3JQI8kbc9TQv91WyZtax0cUvp_dyZDFrQ102PkIfPlQqp-ZrC8T04usmMkJ_AKCeXlo2JF_ZEpvb4Y4Bb-3XJMU7mz5iaaLHYVebo)

### Enrollment Collection

The enrollment would be kind of tricky as it deals with linking a few of the other collections, the fields will look like this:

-   Enrollment Date: Date Field 

-   Status: Text or Select Field 

-   Grades/Progress: Text or Numeric Field

We will go ahead to create the enrollment date, status, and grades placing them in their proper fields.

This is what the Collections look like:

![](https://lh7-us.googleusercontent.com/LwtV7Fv9iNLaFrbI2QbXvnXayspC7A7np3txo0PsMSpyG08RzEWPU6msdMuZgxOAHN87HQuI6DH8K8ZpD0G8h20K_T_la5cRoLcvEQe9Xsw3z-VZ-eFHYi57cnLFuAJykvuvUEFuKEXuhZm_Zt-uKwk)

We could give it nice icons to make them look better and interactive. To do this on any collection, click any of your choices and scroll below the fields created, you will find the Icon option, and pick anyone that fits, Directus provides us with a lot of options, we can put in random names and get random icons too as seen below.

![](https://lh7-us.googleusercontent.com/Amg-dwA9bXk2IH0GYgr3ncSC_HnUIQJZRa7GTm-_h5O75w8dNdoJacK1YGDCUYyAxC2J17nThYDWC7PD54Pp1BhMP81KuMyCgICRC9Xws8sBbkugmNyIH30Fl9dWetvz54pTaCKOiFmNvcDwhRd7wb8)

We can go ahead, and add the icons it should look like the image below.

![](https://lh7-us.googleusercontent.com/j-0FOXFC2FN0v1U7SdNwIoEv1d3QIzVtnxTpm9a3_WQUbRVN2HuaKyzRMNuBfZ3RkNAVe_pV7LmdqshdakNguHFdCIJosJHi6r1i_p0AvD9gRLWUQtrtP8d33LWU-OPX9gHmne2jBMzPjhQrG608xT0)

We can now go ahead to understand and fix the relationship between these collections because they are supposedly dependent on each other. Let's break it down a bit; 

You do have course, and each course could have multiple lessons, and each lesson could have its quiz, right? We can also say each course should have its instructor.

As we can see they are heavily connected to give optimum value. Directus has a field for these kinds of relationships, it is called the Relational field. Under this field we have one-to-many, many-to-one, and many-to-many, we need to understand which one fleshes out the described relationship better, so it can be used.

#### One-to-Many Relationship: 

In a one-to-many relationship, one record in a collection can be related to multiple records in another collection. This relationship is typically represented by adding a "Many" field in the collection where there are multiple related records.

For example, in our scenario, the "Courses" collection can have a one-to-many relationship with the "Lessons" collection. This means that one course can have multiple lessons associated with it.

In Directus, you establish this relationship by adding a "One-to-Many" field in the "Courses" collection that links back to the "Lessons" collection. Each lesson record will then reference the ID of the course it belongs to. Behind the scenes, Directus will set this relationship inside your sequel database and make sure everything plays nice with each other.

So we will navigate into courses(courses seem to be the mother collection), click the Create field, scroll down to the relational fields, and select the one-to-many:

![](https://lh7-us.googleusercontent.com/Fo6OyKHB1BcdnK8fcqNHYdtRsj5Okq_VImsW-pw4ZmipOQjtfZWaWj4B5-WzKY83ldGx_IxPnwGL2bpqhAgvasFBNW3P245knu9JxuAEWI_b75S3ai24sYga-Xtqqm6CXDhqRI0O4nGxP7_ufZw8cXw)

For our key we will call it "Lessons", the related collection will be Lesson, and the foreign key, which is our key inside the Lesson collection will be course;

![](https://lh7-us.googleusercontent.com/vlz0BeZQYgajzhOUO_3srP-lzk6ZhbeU5Ex0NwdhlKAYJH7RtbOciIJSzMnkEMdRCyVhXDVu5MxobZeNpo339d3yPJRPSqPKoJB5hLHpPCC5cpoa9zkm-7fjB5cLNxDJZolEFC_BJ0dGOoAcF54OfY4)

For our display template, we will use the name "name"

![](https://lh7-us.googleusercontent.com/jC-HlWxNsjjZFMYTTekHtok9GjkTXJUAt5czDdWOVWilH7UIQU3_wVv0yItmuJi7hBqIoN4lQzb5n42lO0-LWOEsPTBkRkBFWgssWBPtjBys79VBospZKaMIvS7XKfApVGxoxeu6RVDM1Xm3-iOwDS4)

Scroll below and enable the item link, then save.

![](https://lh7-us.googleusercontent.com/nFNE86v5Qit2Iol0OFq_ORsJvPSlRDY2F_PjrcinuyRtM6pXf_ZjWEnXKwrQJh_9_a0ZVlS9CoxL5ejAdpjU4jxTVGUdELvKA2x1MaaIZTo7pcJd8QIwfGdpvTaCanwNl2wJB7zAwN9jQ0vxrVfUH00)

Now if we go to our courses we can find lessons as one of the fields;

![](https://lh7-us.googleusercontent.com/dKheQgAF8nxtLj8-EG8ddOv9EbkNWAvjNjl3V6jTpaGeF_oTKGxBJOafVSRY2t7WJ_InbrEplvn66PjOtB9qZRJhUMQHmbSyAZ852JxlGXI1y-L_Yksu2r8XMQgy8rcnpwqh4vwgXIYn3ZmjJx82EqA)

The same will be for lessons too, we will find courses there;

![](https://lh7-us.googleusercontent.com/H0wgWBjSElWxVGTbUuTqqTdNYO6D1lD9GuSyhBapvtF84icJbmJQpauO1DOtq9GJ5_H-kzsLW_XcIfDoaZYmZoJ78aHc1zRY5-BFClqR4XO4Tlovg5UWYF7I5rJHEF2yIlBWsnSFqZ0YNrqVIIX4QU0)

We should do the same for Lesoon and the quiz, each lesson could have its quiz. If you think further you could see another good example of one-to-many, maybe you guessed rightly, yes the quiz should have many questions.

Going further we could do more of these relationships, like an instructor could take many courses, and a course could have more than one instructor, this is a good example for many to many. We will have to go to the courses or instructors collection, it could be any actually, in our case we wiil go to instructors, just because we  havent used it yet. We will use the many to many relationship, our key will be courses, our related collection will still be courses;

![](https://lh7-us.googleusercontent.com/TBYw6vogrDkTFBk6dlPUTT726VnXzpd2hB6-LgNaUCK_yT6EzgdKIf58gvX5a-Sn_mSu93ggqRTAus42z1lfFEM5mUHRmSsuu1Yh1a_uhisu-OjazkAe7pJ8DIywo9864UjQeRWcNSIklyNPGioc-jM)

If we go back to the data model Directus will do it's magic and viola we have a hidden collection, below;

![](https://lh7-us.googleusercontent.com/X4LzpJXQn9PDZeUdfYbVp9tyvHrFc0OEJnKJvfeS7BoevGZdpczYmidUcbbxdN0GjhgDyIBQkBRHN57OKxEZr3oqPn8MOCdkYzf2y7PdvkHtDPLzwzK-q5rV-BKwWQPJo2TwEtGmlDP1AH7GWYIVncw)

Create 3 courses to test with, and then test accessing it via API by navigating to `https://your-directus-url/items/courses`. You would normally see the same data you authored in the Data Studio as JSON, except you don't. Let's talk about why.

Your request to access data via the Items API provides no authentication details, meaning it is a public request. The public role has no permission to do this.

## Roles and Permissions

You can configure roles and their permissions in the Access Control area of the settings module. 

You want to be careful with those you give administrator roles to, as it has access to administer all data, and restrict the public role from certain things, like courses enrollment, because you may only want authenticated users to access them.

Each role has a set of permissions for each collection which describe the ability to see, create, edit, and delete content. Using Directus, you can also apply custom permissions which are more granular than a "yes" or "no", but perhaps only allow instructors to edit their own courses, and students to see those that they are enrolled in.

Let's create permissions for the students and instructors.

### Student Role:

Students should be permitted to;

-   View Courses: Students should be able to view all available courses. 

-   View Lessons: Students should be able to view lessons within their enrolled courses: Students should be able to view assignments within the courses they are enrolled in. 

-   View Quizzes: Students should be able to view quizzes within their enrolled courses. View Own Profile: Students should be able to view their own profile information. Restricted 

-   No creation/editing/deletion permissions: Students should not be able to create, edit, or delete courses, lessons, assignments, quizzes, or any other content.

### ![](https://lh7-us.googleusercontent.com/nO2Y5acNRKDoHRe7K6XKbccWqD2Y6sUxiSMuAHiBO7W0IHrhrnLaHdb0oJEVmDvhfxmvo8mSgRT14EZv1YvNnd_qkNQx6jESfWelh6Z6QnpkKZc7JEL5dGzPTkcT-5t9YTKxnfVvkU962eiSZe9FhNA)

### Instructor Role: 

Instructors should be permitted to;

-   Create/Edit/Delete Courses: Instructors should be able to create, edit, and delete courses. 

-   Create/Edit/Delete Lessons: Instructors should be able to create, edit, and delete lessons within the courses they are assigned to. 

-   Create/Edit/Delete Assignments: Instructors should be able to create, edit, and delete assignments within the courses they are assigned to. 

-   Create/Edit/Delete Quizzes: Instructors should be able to create, edit, and delete quizzes within the courses they are assigned to. 

-   View All Data: Instructors should have access to view all data within the platform. Restricted 

-   Limited administrative access: Instructors should not have access to manage users, roles, or other administrative settings.

![](https://lh7-us.googleusercontent.com/YGWsatZncTynyPnj8p5a9ANJzu3irWvKQ4S3btPUr6GUPJTIfHBLKYXhfSCwJuV8_kkHwNopI2mkRALEsYuik--jck8gLalu3-CGJrPFz_LGh4fqV290UCJs0TYEHFtY5ZKp0L8AB-tKJ3M2KdJOFio)

### Custom Permissions

![](https://lh7-us.googleusercontent.com/RHP_HpUinE5HzuVJOPQBYXrx1kyZJRw04Qil8wL3CmnJiuBTM_AO-lWkFtcRfpALgmhYPBoRKikjrAF6vModJfcROIuWLJjE07TXJEw3ty01X76KO-sG5VGtNRNAvg7eUIyDSDrC4IKmbDtzVRSihzE)

Now for the instructors, since they do have a little more hedge of control, we would want to insist on them having limited rights to do certain things, and there's where custom permissions come into play, Directus in its mass quality didn't leave tables empty on this one, with Directus you could easily restrict certain rights as we earlier mentioned.

We do want instructors to create a course quite alright, but we also would want them not to update other instructors' courses.

So in the instructors roles, we will go instructor-courses collection, click edit, select  custom item, go to rules, click the filter drop down, click instructor -Id then set it to $CURRENT_USER.id. This should make only instructors who created certain courses be able to edit them.

![](https://lh7-us.googleusercontent.com/eszZASHeqR49PsGeZQF7dJwKeJ-2R27676PFkVcFGlZKLbe-xRoCrgBZW3lXTDBfLjjpATpWINEYTQRdFGToRD9zmQTefOwpDufFvWVyYNgQGkQThs6F6FZPR1Sj7uba5CFA6CHWSVUJOyDAm3ddEUk)

You thought we had finished, but not quite right, we do need custom permissions for the students, as we only want students to view published courses, lessons, and quizzes.

We will open up the read permission click custom permissions, and under item permissions, we wiil create a role using a dynamic variable, by clicking the add filter drop dropdown, we are going to go to the quiz questions field, and use the ' Quiz questions to equals $CURRENT_USER.id", and now they would only be able to see the quizzes that belongs to them alone.

![](https://lh7-us.googleusercontent.com/TsilbdWhcx2I1Oy_uNME4cO-BfJMBcHWmyNCZBtj-obb5NGCnfBsrFrSsP8stVrfz12xI2jm1MXIlK4W0ZhX4nk31YJL62EFbie0f59NYZe4K659T3gi5skIkA3FoT5ZN20jtjOJM87MWAgr8-6Q-OI)

Lastly, we only want students to see published courses, or lessons; we navigate to the courses collection, the custom permissions, then item permissions, then we click the add filter dropdown under the rules, select status and let it be equal to published. And viola students can only see published courses, we will go ahead to do same for, lessons, and quiz.

![](https://lh7-us.googleusercontent.com/hZBD0NEPySG1pe8Y1hQzGAcf8h-QQJvr9qoEDiphvUaX6p0Mkag2TRs-KRPTOIVMYTlLMBjLZoUejxlv4a5SlIBPwtqLc19qXH943GJs3-bkd8ZbdggqOBi7J8K16seeEv0BLlQuoqJ3WUnqvlOPvbg)

Conclusion
----------

In this article, we have been able to go through the exercise of Data model and oermissions for Interactive Educational platforms, in directus, thank you for sticking this far, I really do appreciate. Keep using Directus!