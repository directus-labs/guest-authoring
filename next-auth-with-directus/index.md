---
title: 'Using Directus Auth with NextAuth.js'
description: 'Learn how to build an authentication system in a Next.js application using NextAuth.js and Directus Auth '
author:
  name: 'Trust Jamin'
  avatar_file_name: 'jamin.png'
---

## Introduction

Authentication is an important aspect of web applications, as it provides the ability for users with the right permissions to access certain resources or perform specific actions.

In this article, you'll build an authentication system for a Next.js application using NextAuth.js, and Directus Auth.

![Directus Auth with NextAuth.js Demo](demo.gif)

## Prerequisites

You will need the following knowledge and tools:

- [`Node.js`](https://nodejs.org/en/download/) installed on your computer to build the Next.js application
- A Directus project - you can use [Directus Cloud or run it yourself](https://docs.directus.io/getting-started/quickstart.html).
- A basic knowledge of terminal/CLI commands

## Setting up the Backend (Directus)

### Create a New Customer Role in Directus

Login with your admin credentials and head over to Settings > Access Control (`your-directus-project-url/admin/settings/roles`), and click on the `+` icon on the top right to create a new role for users that can access your Directus app.

Name this role `Customer`; also uncheck the app access checkbox as you do not want users to access your Directus app here but to login from the Next.js application and update the description to be `Users from Next.js application.`

 ![Customer Role in Directus](customer-role.png)

### Setting Permissions for Customer Role

When you navigate to the User Directory (`your-directus-project-url/admin/users`), you should find two categories of users: Administrator and Customer.

To set specific permissions and restrictions to ensure your users with the `customer` role do not access unauthorized information, and new visitors on your frontend application can only sign up as a customer, navigate to the Access Control settings. Click on the Customers role, the info icon in the sidebar, and take note of the role `id`.

![Customer role ID](customer-role-key.png)

Navigate to the role settings here and configure permissions to only allow user creation using the customer role `id`.

In the public role settings, in the `directus_users` system collection, configure custom create permissions.

![Set Customer ID Role to Public](use-custom.png)

Navigate to the `Field Validation` tab and create a new rule:
role Equals `REPLACE_WITH_CUSTOMER_ROLE_ID`

This will ensure that users created from your frontend application will permanently be assigned the role of a customer.

## Building the Next.js Application

### Installing Next.js

Run the following command to initialize a Next.js project:

```bash
npx create-next-app@latest next-directus
```

During installation, when prompted, choose the following configurations:

```bash
✔ Would you like to use TypeScript?  Yes
✔ Would you like to use ESLint?  Yes
✔ Would you like to use Tailwind CSS? No
✔ Would you like to use `src/` directory?  No
✔ Would you like to use App Router? (recommended)  Yes
✔ Would you like to customize the default import alias (@/*)? Yes
✔ What import alias would you like configured? @/*
```

This will install Next.js with Typescript.

Using the command below, start the development server:

```bash
npm run dev
```

You should have a Next.js application running in <http://localhost:3000/>

### Installing the required dependencies

For the frontend of the application, you need the following dependencies:

- [NextAuth.js](https://next-auth.js.org/) for creating the authentication system for Next.js
- [Directus JavaScript SDK](https://docs.directus.io/guides/sdk/getting-started.html) for communicating with your Directus backend.

Install the dependencies using the command:

```bash
npm i next-auth @directus/sdk
```

### Configuring the Dependencies

Create an `.env.local` with the following contents:

```bash
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=YOUR_NEXT_RANDOM_SECRET
USER_ROLE=THE_ROLE_OF_YOUR_CUSTOMER_FROM_DIRECTUS
BACKEND_URL=YOUR_DIRECTUS_URL
NEXT_PUBLIC_URL=http://localhost:3000

```

### Configuring the Directus SDK

Create a new directory called `lib`. Inside it, create `directus.ts` with the following contents to initialize a new Directus SDK instance:

```ts
import { createDirectus, rest, authentication } from '@directus/sdk';

const directus = createDirectus(BACKEND_URL).with(authentication("cookie", {credentials: "include", autoRefresh: true})).with(rest());

export default directus;
```

Here, you are creating a new Directus API instance with enabled `authentication` and `rest` API features.

### Creating the AuthForm Component

Open the `next-directus` project in a code editor. Create a `components` directory and an `AuthForm` subdirectory. Inside `AuthForm` create `index.tsx`:

```tsx
import Link from "next/link";
import { FormEvent, useState } from "react";

interface Data {
  first_name?: string;
  last_name?: string;
  email: string;
  password: string;
}
interface AuthFormProps {
  title: string;
  buttonText: string;
  onSubmit: (data: Data) => void;
  linkText: string;
  linkDescription: string;
  linkHref: string;
  isFullForm?: boolean;
}

export default function AuthForm({
  title,
  buttonText,
  onSubmit,
  linkText,
  linkHref,
  linkDescription,
  isFullForm = true,
}: AuthFormProps) {
  const [formData, setFormData] = useState({
    first_name: "",
    last_name: "",
    email: "",
    password: "",
  });

  const handleFormSubmit = async (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    onSubmit(formData);
  };

  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setFormData({
      ...formData,
      [e.target.name]: e.target.value,
    });
  };

  return (
    <form onSubmit={handleFormSubmit}>
      <h1>{title}</h1>
      {isFullForm && (
        <>
          <input
            type="text"
            placeholder="First Name"
            name="first_name"
            value={formData.first_name}
            onChange={handleInputChange}
            required
          />
          <input
            type="text"
            placeholder="Last Name"
            name="last_name"
            value={formData.last_name}
            onChange={handleInputChange}
            required
          />
        </>
      )}
      <input
        type="email"
        placeholder="Email Address"
        name="email"
        value={formData.email}
        onChange={handleInputChange}
        requir
      />
      <input
        type="password"
        placeholder="Enter your Password"
        name="password"
        value={formData.password}
        required
        onChange={handleInputChang
      />
      <button>
        {buttonText}
      </button>
      <p>
        {linkDescription}
        <Link
          href={linkHref}
        >
          {linkText}
        </Link>
      </p>
    </form>
  );
}
```

This form will serve as the `AuthForm` for both the registration and login pages.

### Implementing Registration

To create the registration functionality for new users to register on the platform, you first need to create a Next API that the UI of the registration page will consume.

#### Creating the Registration API

Open the `app` directory and create a new `api` directory with an `auth` subdirectory and inside of this `auth` directory, create a `register` directory  with the file `route.ts` with the content:

`api/auth/register/route.ts`

```tsx
import { NextResponse } from 'next/server';
import { createUser } from '@directus/sdk';
import directus from "@/lib/directus";


export async function POST(request: Request) {
  try {
    const { first_name, last_name, email, password } = await request.json();
    const result = await directus.request(
      createUser({
        first_name,
        last_name,
        email,
        password,
        role: process.env.USER_ROLE,
      })
    );
    return NextResponse.json({ message: "Account Created!" }, { status: 201 });
  } catch (e: any) {
    console.log(e);
    const code = e.errors[0].extensions.code
    if (code === 'RECORD_NOT_UNIQUE') {
      return NextResponse.json({ message: "This user already exist" }, { status: 409 });
    }

    return NextResponse.json({ message: "An unexpected error occurred, please try again" }, { status: 500 });
  }

}
```

The code above does the following:

1. Gets the `request` data that will be coming from the frontend `{ first_name, last_name, email, password }`
2. Uses the `directus` SDK to send a request to the backend server to create a new user with a `role` field that is set to a customer `role` ID
3. Respond to the frontend of the application if the request was successful or not

#### Creating the Registration UI

In the `app` directory, create a new directory called `register`; inside of this directory, create two new files, `form.tsx` and `page.tsx`.

The `form.tsx` will contain the registration form and the `page.tsx` will serve as the page rendered on the browser.
Add the following content to `form.tsx`:

`app/register/form.tsx`

```tsx
'use client';
import AuthForm from '@/components/AuthForm';
import { useRouter } from 'next/navigation';
import { useState } from 'react';

interface Data {
  first_name?: string;
  last_name?: string;
  email?: string;
  password?: string;
}

export default function RegistrationForm() {
  const router = useRouter();
  const [error, setError] = useState('');
  const handleFormSubmit = async (data: Data) => {
    const response = await fetch(`/api/auth/register`, {
      method: 'POST',
      body: JSON.stringify({
        ...data,
      }),
    });
    if (response.status === 201) {
      router.push('/');
      router.refresh();
    } else {
      response.status === 409
        ? setError('A user with this email already exist')
        : null;
    }
  };

  return (
    <>
      {error && <p>{error}</p>}
      <AuthForm
        title="Register here"
        onSubmit={handleFormSubmit}
        buttonText="Register"
        linkDescription="Already have an account?"
        linkText="Login"
        linkHref="/login"
      />
    </>
  );
}

```

The code above performs the following actions:

- Renders the `<AuthForm />` component with some customization as a registration form.
- Gets the input values from the form and sends a `fetch` request to `/api/auth/register` to register a new user.
- If the request is successful, it should redirect the user to the login page or throw an error if the request failed

Next, add the following content to the `page.tsx` to render the registration form:

`app/register/app.tsx`

```tsx
import { getServerSession } from 'next-auth';
import { redirect } from 'next/navigation';
import RegistrationForm from './form';

export default async function RegisterPage() {
  const session = await getServerSession();
  if (session) {
    redirect('/');
  }
  return (
    <div>
      <RegistrationForm />
    </div>
  );
}
```

This should provide you with a UI like this:

![Registration Page UI](register.png)

In code above:

- Renders the registration form created in the previous step
- Imports `getServerSession` from `next-auth` to check if a user currently has a session
- If a user already has a `session` ongoing, instead of rendering the registration form, it redirects them to the user dashboard( `/` )

Hurray! You've created a registration page that registers a new user from your Next.js application to Directus.

Go to your Directus dashboard in  `your-directus-project-url/admin/users`, and confirm that a new user has been added to your list of users. Click on the `Customer` tab on the left, and it will show you all registered customers on your application.

![New Customer](new-customer.png)
The following action is to create the login functionality for the frontend application.

### Implementing the Login Functionality

With the registration page in place, let's implement the login functionality using the `next-auth` package.

#### Creating the Login API

Head to `api/auth` and create a new directory called `[...nextauth]`; this directory will be used by the `next-auth` package for all login logic for the application.

Inside of the `[...nextauth]`, create a new file called `options.ts` with the content:

`[...nextauth]/options.ts`

```ts
import type { NextAuthOptions } from 'next-auth';
import CredentialsProvider from 'next-auth/providers/credentials';
import { Session } from 'next-auth';
import directus from '@/lib/directus';
import { readMe, withToken } from '@directus/sdk';
import { JWT } from 'next-auth/jwt';

export const options: NextAuthOptions = {
  providers: [
    CredentialsProvider({
      // The name to display on the sign in form (e.g. "Sign in with...")
      name: 'Credentials',
      credentials: {
        email: {},
        password: {},
      },
      async authorize(credentials) {
        // Add logic here to look up the user from the credentials supplied
        const res = await fetch('your-directus-project-url/auth/login', {
          method: 'POST',
          body: JSON.stringify(credentials),
          headers: { 'Content-Type': 'application/json' },
        });
        const user = await res.json();
        // If no error and we have user data, return it
        if (!res.ok && user) {
          throw new Error('Email address or password is invalid');
        }
        if (res.ok && user) {
          return user;
        }
        // Return null if user data could not be retrieved
        return null;
      },
    }),
  ],
  secret: process.env.NEXTAUTH_SECRET,
  pages: {
    signIn: '/login',
  },
  callbacks: {
    async jwt({
      token,
      user,
      account,
    }: {
      token: JWT;
      user: any;
      account: any;
    }) {
      if (account && user) {
        const userData = await directus.request(
          withToken(
            user.data.access_token as string,
            readMe({
              fields: ['id', 'first_name', 'last_name'],
            })
          )
        );
        return {
          ...token,
          accessToken: user.data.access_token,
          refreshToken: user.data.refresh_token,
          user: userData,
        };
      }
      return token;
    },
    async session({ session, token }: { session: Session; token: any }) {
      session.user.accessToken = token.accessToken;
      session.user.refreshToken = token.refreshToken;
      session.user = token.user;
      return session;
    },
  },
};
```

Let's break down the `options` object for better understanding:

- The `authorize` function performs an async request to the backend auth URL `your-directus-project-url/auth/login` to log in the user using the credentials provided by the request. If a user is found in the database, it returns the user data; otherwise, it throws an error.
- The `secret` field provides `next-auth` a secret for signing the `JWT` tokens that will be generated when a user is authenticated.
- By default, `next-auth` provides its own auth pages for handling authentication; the `pages` field can be used to customize `next-auth` to use custom pages provided.(In this application, the `signIn` page is the `login` page)
- `callbacks` in `next-auth` are functions after successful authentication. The code above has two callback functions:

  - When a user is authenticated by Directus, the Directus API returns an `access_token` and a `refresh_token`. Whenever `next-auth` generates its `JWT` token for an authenticated user, the `async jwt` function attaches the `access_token` and `refresh_token` to the `JWT` token generated. The function also fetches the `userData` from Directus using the `access_token` and attaches the data to the `JWT` token.

  - In NextAuth.js, a session represents the state of authentication for a user; this includes the user details such as `id`, `email` etc.
  The `async session` function attaches custom fields to the `session.user` object to contain an `id`, `first_name`, and `last_name` as well as the `accessToken` and `refreshToken` gotten from the Directus.

That's it; you've implemented the login logic to authenticate a user and also store its details in a `session`; you can now use this `session` data to check if a user is authenticated or not and whether they have the authorization to view a page or carry out a specific action.

To use this `options` object you created, open the `route.ts` file and update its content:

`[...nextauth]/route.ts`

```ts
import NextAuth from 'next-auth';
import { options } from './options';

const handler = NextAuth(options);

export { handler as GET, handler as POST };
```

#### Creating the Login UI

With the login API ready, let's create the page that will call the login API to authenticate a user:
In the `app` directory, create a new directory called `login`; inside of this directory, create two new files, `form.tsx` and `page.tsx`.

The `form.tsx` will contain the login form and the `page.tsx` will serve as the page rendered on the browser.
Add the following content to `form.tsx`:

`app/login/form.tsx`

```tsx
'use client';
import Link from 'next/link';
import { signIn } from 'next-auth/react';
import { useRouter } from 'next/navigation';

import AuthForm from '@/components/AuthForm';
import { useState } from 'react';

interface Data {
  email?: string;
  password?: string;
}

export default function LoginForm() {
  const router = useRouter();
  const [error, setError] = useState('');
  const handleFormSubmit = async (data: Data) => {
    const response = await signIn('credentials', {
      email: data.email,
      password: data.password,
      redirect: false,
    });
    if (!response?.error) {
      router.push('/');
      router.refresh();
    } else {
      response.status === 401
        ? setError('Your email or password is incorrect')
        : null;
    }
  };

  return (
    <>
      {error && <p>{error}</p>}
      <AuthForm
        title="Login here"
        onSubmit={handleFormSubmit}
        buttonText="Login"
        linkDescription="New here?"
        linkText="Create an account"
        linkHref="/register"
        isFullForm={false}
      />
      <div>
        <Link
          href="/request-reset-password"
        >
          Forgot password?
        </Link>
      </div>
    </>
  );
}
```

The above code:

- Renders the `<AuthForm />` component with some customization as a login form.
- Gets the input values from the form and uses the `signIn` method from `next-auth` to authenticate the user.
- If the request is successful, it should redirect the user to their dashboard or throw an error if the request failed

In your `page.tsx`, update its content to:

`app/login/page.tsx`

```tsx

import LoginForm from "./form"

export default async function LoginPage() {
  return (
    <div>
      <LoginForm />

    </div>
  )
}
```

This will create a page like this in the browser:

![Login Page](login.png)

Kudos, you've just implemented a login functionality using `next-auth` in your Next.js application with Directus.

Go ahead and test it. When a user logs in, the application will navigate to dashboard page (`/`)

### Protecting Private Routes with a Middleware

In a typical application, you'd only want authenticated users to be able to access private routes/pages such as `/dashboards` and user `profile` pages.

To do this in `next-auth`, create a new file called `middleware.ts` with the content:

```ts
export { default } from "next-auth/middleware"

export const config = { matcher: ["/"] }
```

This file will ensure any URL in the `matcher` array will be protected from unauthenticated users.

### Implementing a Forgot Password Request

To implement a forgot password functionality in your Next.js application, create a new directory in the `app` directory called `request-reset-password` with two files, `form.tsx` and `page.tsx`.

Update the `form.tsx` to the following:

`app/request-reset-password/form.tsx`

```tsx
'use client';

import Link from 'next/link';
import { FormEvent, useState } from 'react';
import { passwordRequest } from '@directus/sdk';
import directus from '@/lib/directus';

export default function RequestResetPasswordForm() {
  const [email, setEmail] = useState('');
  const [success, setSuccess] = useState('');
  const [error, setError] = useState('');
  const reset_url = `${process.env.NEXT_PUBLIC_URL}/reset-password`;

  const handleFormSubmit = async (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();

    try {
      const response = await directus.request(
        passwordRequest(email, reset_url)
      );
      setSuccess(
        'An email with a password reset link has been sent to your email!'
      );
    } catch (e: any) {
      console.log(e);
      if (e) {
        setError('An error occurred, please try again!');
      }
    }
  };

  return (
    <form onSubmit={handleFormSubmit}>
      <h1>Reset your password</h1>
      {success && <p>{success}</p>}
      {error && <p>{error}</p>}
      <p>
        Enter your registered email and a reset password link will be sent to
        you
      </p>
      <input
        type="email"
        placeholder="Email Address"
        name="email"
        required
        onChange={(e) => setEmail(e.target.value)}
      />
      <button>Send Reset Link</button>
      <Link href="/login">Login page</Link>
    </form>
  );
}

```

The above code:

- Renders a form to collect the email input from the user.
- Fires a request using the Directus SDK to the Directus backend to reset the user password with an `email` and `reset_url` as request parameters.
- If the request is successful or failed, it should display a success or error message on the screen

In your `page.tsx`, update its content to:

```tsx
import RequestResetPasswordForm from "./form"

export default async function RequestPasswordResetPage() {
  return (
    <div>
      <RequestResetPasswordForm />
    </div>
  )
}
```

This will create a new page that looks like this:

![Request Reset Password Page](request-reset-password.png)

Filling out the reset password form and clicking on the reset button will trigger Directus to send a reset email with a link and a token to the user using the email configurations in your Directus setup configurations.

### Resetting a User Password

Now that the password reset request is successful, let's create a page where users can reset their password with the `url` they receive in their emails.

 Create a new directory in the `app` directory called `reset-password` with two files, `form.tsx` and `page.tsx`.
 In the `form.tsx` add the following:

`app/reset-password/form.tsx`

 ```tsx
'use client';

import { FormEvent, useState } from 'react';
import { passwordReset } from '@directus/sdk';
import directus from '@/lib/directus';
import { useRouter } from 'next/navigation';

export default function RequestResetForm({ token }: { token: string }) {
  const [newPassword, setNewPassword] = useState('');
  const [success, setSuccess] = useState('');
  const [error, setError] = useState('');
  const reset_token = token;
  const router = useRouter();

  const handleFormSubmit = async (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();

    try {
      const response = await directus.request(
        passwordReset(reset_token, newPassword)
      );
      setSuccess(
        'Password successfully reset, redirecting you to login page...'
      );
      setTimeout(() => router.push('/login'), 1000);
    } catch (e: any) {
      console.log(e);
      setError(
        'The reset password token is invalid, please request for a new password reset link!'
      );
    }
  };
  return (
    <form onSubmit={handleFormSubmit}>
      <h1>Provide a new password for your account</h1>
      {success && <p>{success}</p>}
      {error && <p>{error}</p>}
      <p>Enter your new password for your account</p>
      <input
        type="password"
        placeholder="Enter your new password"
        name="password"
        required
        onChange={(e) => setNewPassword(e.target.value)}
        autoComplete="new-password"
      />
      <button>Create new password</button>
    </form>
  );
}

 ```

- The `reset-password/form.tsx` accepts a token and sends a request to Directus using the Directus SDK with the `token` and `newPassword` as parameters for changing the user's password.

- If this request is successful, it redirects the user to the login page to log in with their new password.

Inside of the `page.tsx`, update the content to be:

```tsx
import { redirect } from 'next/navigation';
import ResetPasswordForm from './form';

export default async function ResetPasswordPage({
  searchParams,
}: {
  searchParams: { token: string };
}) {
  const { token } = searchParams;
  if (!token) redirect('/login');
  return (
    <div>
      <ResetPasswordForm token={token} />
    </div>
  );
}

```

The `page.tsx` components checks if a token is present in the `reset-password` url; if it is present, it displays the `ResetPasswordPage`. Otherwise, it redirects the user to the login page.

## Summary

That's a wrap; in this article, you've successfully built an authentication system with password reset functionality using `Next.js`, `NextAuth.js`, and `Directus`.
This is just a glimpse of what you can implement with Directus
Directus runs entirely as a backend service, meaning you can build complex backend services that will serve your frontend application with any database of your choice

Some possible steps you can consider to improve this application:

- Improving the functionality of the authentication system to accept OAuth providers like `Google`, `Twitter`, et. c
- Improving error handling to show the correct errors to your users properly
- Add a
- Create new `Item` models that your `Customers` can use to create their data from your frontend application
- The default Directus user comes with a list of default fields such as `first_name`, `last_name`,  `password`, `email`, and others. You can also extend the `directus_users` schema to contain other fields to suit your needs.

The complete code for this tutorial can be found [here](https://github.com/0xJamin/next-auth-directus)
