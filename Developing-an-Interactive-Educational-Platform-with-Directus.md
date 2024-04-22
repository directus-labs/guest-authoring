## Developing an Interactive Educational Platform with Directus

Introduction
------------

Building a mini project and managing information may not dazzle you, but when it comes to large projects with extensive data from various sources, including user-generated content and administrative inputs, can be overwhelming without good organized assistance.

Interactive educational platforms, for instance, engage students with different courses, instructors, and quizzes, all of which require structured data management. Platforms like Directus remain important because they help easily manage a great amount of this data.

They provide a place where you can organize data, control who can access it, and make changes without needing technical skills. This doesn't only saves time but also simplifies collaboration on projects.

Goals
-----

This is an article that focuses on Data Modelling, Roles, and permissions in Directus. The reader would be able to harness the totality of how good Directus could be in CRUD projects like this.

Interactive Educational Platform 
---------------------------------

Interactive educational platforms are platforms that engage students with courses, great instructors, and quizzes. Interactive educational platforms have been evolving over the past few decades, gaining more engagement in recent years due to advancements in technology and the increasing demand for online learning.

One notable example is [Coursera, founded in 2012 by Stanford professors Andrew Ng and Daphne Koller.](https://en.wikipedia.org/wiki/Coursera#:~:text=6%20External%20links-,History,left%20Stanford%20to%20launch%20Coursera.) Coursera offers various courses from universities and institutions worldwide, providing interactive content, assessments, and certifications.

Another prominent platform is [edX, established in 2012 by Harvard University and MIT](https://press.edx.org/inside-the-first-year-data-from-mitx-and-harvardx#:~:text=edX%20is%20the%20global%20online,to%20meet%20every%20professional%20moment.). edX offers courses from top universities, colleges, and organizations, with a focus on interactive learning experiences and open-access education.

[Udemy, founded in 2010 by Eren Bali, Oktay Caglar, and Gagan Biyani](https://en.wikipedia.org/wiki/Udemy#:~:text=It%20was%20founded%20in%20May,Udemy%2C%20Inc.&text=San%20Francisco%2C%20California%2C%20U.S.&text=As%20of%20June%202023%2C%20the,over%20870%20million%20course%20enrollments.), is another leading platform that allows individuals to create and sell online courses on various topics. While Udemy's courses vary widely in quality and subject matter, its platform has independently helped educators by enabling anyone to become an instructor or learner. 

## Key Concepts

This platform has several key concepts:

1. Courses are...
2. Students and Instructors are...
3. Lessons are...
4. Quizzes are...

In your Directus project, create a `courses` collection with a primary key type of UUID. Check all of the optional fields and complete setup. Your collection should look like this:
![](https://lh7-us.googleusercontent.com/gpxJbPSFhEFGIdhmX52qhRu5WCJqndzQbFvZDZjfIblIz3JjQsetsP-gIGAHfGFQUelLh-DNgEWNxSAQgpBrUDaq2SVtgy1EsKo1QiT7XJgQXxPLX_gDegu2HZs09KBeDWfuq6d3oxWsNKP1oLf-Cdc)

In the page above we have our previously selected fields, and also a button to create more as we build. In our courses we will need fields such as Course Name, Description, Course Code, Start Date, End Date, Course Image, this should be enough fields for a professional educational platform Courses. We will go ahead to create them, at this point, we can click the Create Field button.

For our Course Name, we will select the input field. 

![](https://lh7-us.googleusercontent.com/_OIgdPKEiYSCMn31_EbNDqZrbUpS2chq_fChw6GkikD2dEcyvjuMKdlG9Vtt0bS8v_-jrpYg73jausOMIRuoZ9CyPIcPEkWpaRvO1h0XdySB9eUeSEY44IyKcv7pDH3LHC2I9qXzWcFHBOhz3ijLbI4)

Fill in the name as an option:

![](https://lh7-us.googleusercontent.com/8hRqFmDP274-mwGMo2rW5y7kXkWk7Jz8rX0M4CjSKmQNdegRoRJYERDPtH8gkQ4TWRylOhfjfBEPH6PS9jfqSau8SmUBv3UKItcnBo1HLXXUMb5ofzWdw6Y_5Cs7zhnRL_oDWOlXMuhEnOiqjIsEEPw)

Scroll below to save.

![](https://lh7-us.googleusercontent.com/gVtXDivgi_Yf4WyqENsgdnyQXyKcajqv_3yb-FOGk5ofLm5XEwmL7YyrMRNeklmBTi8SlG8EA0biXAoP_ww0QBlZI9XSk-Q6vUZ6bn_DW94pg62rWynBFmgBtHIwkCIaa6Yr1yP1bvgqjbSlDN0iwFI)

At this point, we just easily created our first field, in our Course collection, and we will go ahead to do the rest. Next, we will create a Course Description, we will repeat the process of creating a field, but for this field, we will be using the WYSIWYG field option.

![](https://lh7-us.googleusercontent.com/o9w-A6mNJ6q3K6xhml5qriasAXy-tI71rGbF_KC8QjTrLWfy5n1SgKMowSESgqr5fzSBZ2kuhHIB0KUsRYFjcg258lWisbzGecSy6ROF-aIevtAuCq24odRDSg9KaqvSeTg7yjS9foGpzJSqW96V5vo)

After filling in the key, we will go ahead to save it. For the Next Field which is the course code. We will be using the Input field, fill it up, and save it as seen below.

![](https://lh7-us.googleusercontent.com/BpdRFsja0EPFCoFWJ7UnkWjisPTBh2P0iVkk3SIQMw627dhKv6TsG-7JEWc1DtgiYjmCk0hOJa3n4bAWquqXFxxh3wOZiW-4iJkfTAEVvBwVuiEhH-8fr_b4GO4eLbFg0x4TrVXrRd4t8vp_R4V4Q4o)

For the Next fields, we have the start date and the end dates. For these fields, we will be using the Directus Datetime field option as seen below:

Start Date

![](https://lh7-us.googleusercontent.com/beP60zdwrugn17yUPXwEol6z6Bia8Zcp1yrQTK0-tNjZ1HN0XyAxbd1wUbLI8eJk8mT0OPm8Wx1ydlW0i_2go5-9YfbQY7L7v5GN0qYBdvtLyeFV5mbFPw5mUg-LlZF23kmeykr3qU8vLhnxpV0TZtU)

End Date

![](https://lh7-us.googleusercontent.com/FEo-UYTeMpMYiF_m4V3Tsos0nedvTklLlcEpSJdT7d5mlwEVFDbTJNv8jsglHQheHZH_vF8NyIbVcQsLIJKz9nfnfYyyYhh6k3DW2PGLweVlHP8GKDVmMX3bIcG9LhBpEFW9tasOQ-z3jbslmZX9jIM)

Lastly, for the course collection, we have, our image and our URL slug, as you can guess already Directus has an Image field option for the image, and for the slug, we will use the input option. let's create and save them as seen below.

![](https://lh7-us.googleusercontent.com/AHQxPKN5eCjslLAr0LIBVc6mY9PWRRbBCqmWhrq2fKmiT3p2l-lhCmuCsnuSfV0WAogPBKZKBsdcD7SsiygLjSp1WDXcrooPgsIdxc_u0o7ykYD4oUHHPDEXyu_FQlEGgliaUEefedZ98j4SFR0ncg0)

While for the slug we will want to make it URL safe, after creating it we will. Select it:

![](https://lh7-us.googleusercontent.com/7Gpboo0vTxeACYdLu7Su9QGe5m51cOuJ9l2So_xz2-Y0elE58_cK5RSiQGNLPtWtmJVZgp4HT0u36BLHrQXx9ie2hLddIKunBoYC1fQjS1crGd18qqWq_L4NLX-0ci_pOyvDA8GwL3EqogABY8C7x8c)

The dashboard below pops, navigate to Interface:

![](https://lh7-us.googleusercontent.com/QyWPK0PzVd1iP0Js_m5pcgQbr5rRkTGKG3T_A6v-Y9LLEB5S4acMsZP5iCjtyp9bEHfe33D9HRo989ms61D8aUgDg-N8SyxUQ0KNgYh-alOwwWJR3bgaeh1YzxnEKUpUA3MtFejg0oKSTl3ch6aD-og)

Scroll below, select Slugify, enable the "make entered value URL safe" and save with the good icon at the top right.

![](https://lh7-us.googleusercontent.com/DXnba-E_7qkJ_G05j_td3hqn3gzQpXcCQYieZZOSwPaGVzRhEXjfC-LCsovSmsaKKghIStJE_TYgXsB4oM_buSsC25TMuvVRU3eVjcR99Ey0lJhEbmKj2uP1-rEbe4sx7bMySvypiBUu9hg9N2E0m-I)

### Lesson Collection

For our lesson collection, we will add fields such as Lesson Name, Description, Content, video URL, and Lesson Number. Let's go ahead to create this collection. Navigate to the settings as seen below, That is where you will find the Data Model, and every previously created collection, click on the plus icon at the top right to create a new collection.

![](https://lh7-us.googleusercontent.com/YMWX9dqmuC9P-CJpLXyp6iwy9UM7sH2bs2TeDDonhPyCY0NQn8qhTymO9FAX4m4A9eJ4UhXjBuHqGYLStpN9cTj7117gwp-qKQMxUDReODB1Pa1AjswFZm-s9xljk1M-jUxtJ8-VvXLB_XGJXT8GPY8)

Give the collection the name "Lesson";

![](https://lh7-us.googleusercontent.com/y3mjx1SqZmw4iGp3eKRrhvAPZEtudUMD813MmT0GYD7H7gezDbimiUKCdlXEoyUmI4EXBklAT5susrvpdze0GtMlqirt8Y8-43iy5ncj5NODiDEYY9dARLE5oxper-ON4I7YkzpCJuz7dJKm98AlAQk)

Navigate to the optional field and select everything, this will be the norm for all collections to be created, these fields being selected are not necessarily mandatory but they could also come in handy in the development, so it is advisable to always select them. I may not repeat this initial process of naming collections and selecting these fields in our future collections but note these steps as they are important and should not be overlooked. 

![](https://lh7-us.googleusercontent.com/IKBxJdFT-ZN5V90IgMxfm1TDtmRTujbXQPMzT0ODQtTbRFnkICZxyropNih8v1nmwiPzSznKT8BIPVqutNBRVQXApIZ3QZaC1O237ZBWxLK0SEWwN-6C3X3Li25X1Vzz5t5syfoEAWpYxjBstFW0CZk)

We can go ahead and save it.

At this point, we will be creating the fields, we will start by creating the name and Description, the fields are the same as the course collection kindly do the same, and if you have forgotten which is normal, you can simply scroll up to revisit it.

The new field is the Content field which is used to hold lesson contents, we will be using the WYSIWYG field as seen below: 

 ![](https://lh7-us.googleusercontent.com/EiGlXJDGfZUaBvT39lxnVG-57qFFag-PnS5OIpfHV7UyHk_wKswiDa55S1BDNjIOxpHPaFGMnkWtk53-CVKvAA6BNzVXPYQUrRArTte4IE9c0j824ubK2E2AsF1t4yKMs1gpF235Ln5htpSMhM17R5Y)

We can save afterwards. For the next field, we will have the Lesson number. For this field we will use the input field and select an integer as all lessons would be whole numbers then we go ahead and save.

![](https://lh7-us.googleusercontent.com/P8UwtU5DYrLjQSY3YX_icJIZ9y-yar58hfPboB0IlUyhhvNTEiSmMfDKBc7szGU-vNRU5op0kmkbL5qcOU8w-BLE4IdGoPDWck0elCmaflUvSfSBgPk_BRaiMo6mtPuuRnpxg_fW-PyL3IwwPaDyrRw)

Lastly, most lessons are videos, and Directus allows you to manually upload videos or you could opt for a URL, which could be much easier, and for this field, we will use the input option, save afterward. This will be all for our Lesson collection.

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

If you click on it, we have this;

![](https://lh7-us.googleusercontent.com/_yIlUTgXhUC1TcOaJAXqi10CjRcCJOk2sk51bUe5bWDs1G421eAXo2ysK8F-wa4LgaFUZNmf_RJU8zBvEeFOINCTBjoyW-4hiZimLX08UdhfjtoT2m1OeseLYP3eUtx1O5aay3628MKKLwMsyCjXlxA)

At this point, we are done with our data model, let's navigate to our content;

![](https://lh7-us.googleusercontent.com/Y5LREObxI1Dug3eOClw0IqmV8RLXqnbxC_kygIfZR3pgtavzUAHIAupQNMiGoBpdURuPqZ3QBMr7gkeoXuZWOiVMOVZJ8GB_JNQCO0A_3PgU0oLmCCeKcetDQn8CG1BzKsN5-fRD6FF7qZJzDmlwF3I)

This will take you to the dashboard below;

![](https://lh7-us.googleusercontent.com/eCQn-8QraTeRhoF9EHyEomf51uLE0XWTTYDn449Kp8s9GHBtm6eMW80ZeNTYJSFaFalhiC18T0BMG9c00GqzyH1WqXHPRkV3jrNXvKYJh23RFAawjG3-8K6bj2l9kJtWNIkGeApUF4qkVriPO7cziSY)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

We can go ahead and create our items, by selecting any of the collections above. In our case we will do courses first;

We will create items in our courses;

![](https://lh7-us.googleusercontent.com/xIC1ODpTfX5iSlRi5_QD0BUjYdt2Kf0Hskrz_GTfOKy_Ixi8wEGlJirkJx5U1HKqgv_nsNRGGwjys-IAOpawuiJFlNDZvj-ZpQ5pw192tpHoekr6F-s5JtKPG3B5R71lfO5boAVLhCsP2xhmnQeRsgg)

If we navigate to: <https://interactive-educational-platform.directus.app/items/Courses>, we will see our raw data for our courses;

![](https://lh7-us.googleusercontent.com/0py2EvtrdkF6LaREhqb20F_kuZ0jaukRHzgDj9nHByPfvnX7bpk28q-_m3uFLaBsM1nKS6h5LFS6emXsTdmyaWDJviRthHd45xYekfq2VvQEBBfr9c1AcChVuQEYt_hv_eh3GOtv46k2HSsVgdDBYKM)

If we navigate to: <https://interactive-educational-platform.directus.app/items/Lesson>, we will see our raw data for our Lesson; we do have one lesson here, we could create more than one lessons if need be;

![](https://lh7-us.googleusercontent.com/c2UkDhLS2PVgBiUm0zSGLgzqRcqXgqoC-RThI4s9QBYp00wW5Jvw6fPZ0w3IyIPycKbUVVRGSea1IqAuKWfPr0vkljU20xHEcpYCCtjZnOXo0GLZuSTE-FtObR7RbeCIoeLw58ZyL1gdPKSASqpRrIo)

Replace the last words of the links above with the collection name so we can see our raw collections.  Well, the truth is you will not be able to do so, and here is why. You do not have permission to do so, that takes us to our next section, roles, and permissions. Let me break it down a bit, as of now whatever you are seeing is for the public eyes, and directus would not allow you see these contents until the right permissions are given.

Roles and Permissions
---------------------

Roles in Directus can be easily created, navigate to the settings, and select Access Control,

![](https://lh7-us.googleusercontent.com/iQrk7gZ_26xp2Q7H2AMPsUVHItEZTEg0cBX3yzh4UY1P2XJwTeTvtH0Hww-l4UOI75FJVpoiOzM8v8X7sGkAqgz7DmTw4BpQdgmVmpCw4ezd0Mq5VoTQYhhaSjz9aRx9lOiIXKfHMM9qFiYNRGndFxc)

At the top right you will find a plus icon, click it create a role, and save. In our case we will create two roles the Students and the Instructors.

![](https://lh7-us.googleusercontent.com/o_JTTKDATPEJ04qM9KEZ_J2EMu2gkOpYwsKBx6CKprmIRTflQNRgChJ5nNjDuPZNZSRfseIy-H3eA500cMPvuPAi6A8YT4I3W2sRlNovXRe4FBY12uv8PTtjQDDybu3lv3VQXb1jrSqiOU-rXhKCF3Y)

By default we do have the Public and the Admin, we do not need to set permissions for the admin as it has all permissions.

![](https://lh7-us.googleusercontent.com/BIdQat51I19KIs3vA3ahnTCxVHJcgqWvTBO9jILTB-k5dgxdghbBrRgboqA5SJB7h7gRWriBrkTCChPzLdlldrsX928dwb2WZzjf1u56KZKAoI9iSRB73VdCAulNXQ1S2subXi7pVMYaDp0-cewq9Bo)

You want to be careful with those you give administrator roles to, as it has no restricted access, well for the public, you want to restrict the public from certain things, like courses, because it most likely going to be a paid course, and even if they weren't, you would only want authenticated users to access the courses, other things we would want to restrict the public from would be everything except the enrollment, we could allow access to contact and chat interfaces if there was. 

### Permissions

In permissions, we may want to be precise as to who should see content, create content, make a change to content, and delete content(CRUD). Directus makes the work much easier. We can also apply custom permissions, this will be important in scenarios where we want instructors to be able to create and update courses, remember this is permission given to all instructors, so by doing this, instructors could as well, not just update their courses but that of others, and we do not want that. With custom permissions, we can avoid this.

We will want to go ahead to create permissions for the student, and the instructors.

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