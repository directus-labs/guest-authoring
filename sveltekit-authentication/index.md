---
title: 'Authentication in SvelteKit with Directus'
description: 'How to authenticate users and integrate state managment with Cookies using SvelteKit SSR'
author:
  name: 'Eike Thies'
  avatar_file_name: 'avatar.jpg'
---


## Overview

- Login/Singup User
- Save Tokens in Cookie
- Retrieve Tokens and use State Management (SSR)
- Use Directus SDK (With Token) on unified Server/Client and/or Client
- Example Role based Access



## Primer Authentication

- Different authentication mechanisms to choose from
- In this article using the most secure form: HTTP Only Same-Site Cookies

## Adapt Directus Wrapper

See SvelteKit First Blog Post for initial setup! We need to change a few lines in order to:
- Utilize the Access Token in the Directus SDK
- Define a global Cookie Options Schema (using HttpOnly and Same-Site Attribute)

```js
import { createDirectus, rest  } from "@directus/sdk"; // [!code --]
import { createDirectus, rest, authentication  } from "@directus/sdk"; // [!code ++]
import { PUBLIC_APIURL } from '$env/static/public'; // [!code --]
import { PUBLIC_APIURL,PUBLIC_COOKIE_DOMAIN } from '$env/static/public'; // [!code ++]

function getDirectusInstance(fetch,token) {

  const options = fetch ? { globals: { fetch } } : {};

  const directus = createDirectus(PUBLIC_APIURL, options)
  .with(authentication('cookie', { credentials: 'include' })) // [!code ++]
  .with(rest());

  if(token) directus.setToken(token); // [!code ++]

  return directus;
}
export default getDirectusInstance;

export const constructCookieOpts = (age) => { // [!code ++]
	return { // [!code ++]
			'domain': PUBLIC_COOKIE_DOMAIN, // [!code ++]
			// send cookie for every page // [!code ++]
			'path': '/', // [!code ++]
			// server side only cookie so you can't use `document.cookie` // [!code ++]
			'httpOnly': true, // [!code ++]
			// only requests from same site can send cookies // [!code ++]
			// https://developer.mozilla.org/en-US/docs/Glossary/CSRF // [!code ++]
			'sameSite': "strict", // [!code ++]
			// only sent over HTTPS in production // [!code ++]
			'secure': process.env.NODE_ENV === 'production', // [!code ++]
			// set cookie to expire after a given time // [!code ++]
			'maxAge': age // [!code ++]
		} // [!code ++]
	} // [!code ++]
```

Additionally we need to define the Domain the Cookie is valid for in the  `.env` file:

```
PUBLIC_APIURL= "https://directus.example.com"
PUBLIC_COOKIE_DOMAIN= "example.com" // [!code ++]
```

## Setup Login/Signup Form

Let's start the user journey from adding a login/signup form

```svelte [/signin/+page.svelte]
<script lang="ts">
	/** @type {import('./$types').PageData} */
	import { page } from '$app/stores';
  let email,password;
	const redirectedFrom = $page.url.searchParams.get('redirectedFrom')
</script>

<form
	action="?/login{redirectedFrom?`&redirectedFrom=${redirectedFrom}`:''}"
	method="POST">
	<div>
		<label for="email">Email</label>
		<input id="email" name="email" type="text" bind:value={email}	
		/>
	</div>

	<div>
		<label for="password">Password</label>
		<input id="password" name="password" type="password" required bind:value={password}	
		/>
	</div>

	<button type="submit">Log in</button>
	<button formaction="?/register{redirectedFrom?`&redirectedFrom=${redirectedFrom}`:''}">Register</button>
</form>
```

The html template is straight forward. We only have a small function to let us redirect the user to a page he was trying to access before needing to login/signup.

The javascript form action below will handle the actual request.
The Directus SDK cannot save HttpOnly Cookies, thus the Directus API is directly accessed in order to be able to save the tokens in a secure cookie via SvelteKit's cookie handler.

```js [/signin/+page.server.js]
import { redirect,fail } from '@sveltejs/kit';
import { PUBLIC_APIURL } from '$env/static/public';
import { constructCookieOpts } from '$lib/directus';

const REFRESH_TOKEN_EXPIRATION = 30; // in days - adapt this with your Directus Settings

export const load = async ({ locals,url }) => {
	// redirect user if already logged in
	if (locals.token) throw redirect(302, '/profile')
	return {};
}

const loginUser = async (request,email,password) => {
	let req = await fetch(`${PUBLIC_APIURL}/auth/login`, {
		method: 'POST',
		headers: {
			'Content-Type': 'application/json',
			'user-agent':request.headers.get("user-agent"),
		},
		body: JSON.stringify({
			email,
			password
		})
	});
	if(req.status !== 200){
		throw new Error(await req.text());
	}
	req = await req.json();
	return req.data;
}


const login = async ({ cookies, request, url }) => {
	const data = await request.formData();
	const email = data.get('email');
	const password = data.get('password');
	const redirectedFrom = url.searchParams.get('redirectedFrom');
	
	try {
		let tokens = await loginUser(request,email,password);

		cookies.set('access_token',tokens.access_token, constructCookieOpts(Math.floor(tokens.expires/1000)));
		cookies.set('refresh_token', tokens.refresh_token, constructCookieOpts(60 * 60 * 24 * REFRESH_TOKEN_EXPIRATION));
	} catch (err) {
		return fail(400, {email,message:err.message});
	}

	throw redirect(302, redirectedFrom ? redirectedFrom : `/profile`)
}

const register = async ({ cookies, request, url }) => {
	const data = await request.formData();
	const email = data.get('email');
	const password = data.get('password');
	const redirectedFrom = url.searchParams.get('redirectedFrom')
	
  //First create the user
	let signupRequest = await fetch(`${PUBLIC_APIURL}/users`, {
		method: 'POST',
		headers: {
			'Content-Type': 'application/json',
			'user-agent':request.headers.get("user-agent")
		},
		body: JSON.stringify({
			email,
			password
		})
	});
	if(signupRequest.status !== 200){
		return fail(400, { email, message:await signupRequest.text() });
	}
	try {
		let tokens = await loginUser(request,email,password);

		cookies.set('access_token',tokens.access_token, constructCookieOpts(Math.floor(tokens.expires/1000)));
		cookies.set('refresh_token', tokens.refresh_token, constructCookieOpts(60 * 60 * 24 * REFRESH_TOKEN_EXPIRATION));
	} catch (err) {
		return fail(400, err.message);
	}
	
	throw redirect(302, redirectedFrom ? redirectedFrom : `/profile`)
}

export const actions = { login,register }
```

- Make sure public Access Role has create permission for directus_user collection

## Setup Directus Roles

- Create new Role in Directus (image)
- Setup Permissions (image)
- Bonus: How to define default role in Directus (image)

Let's now try to register a new user ....


## Handle Authentication State

Next up we need to handle the actual Cookie and keep the user remain logged in. For that we need to adapt the `hooks.server.js` file.

```js
import jwt from "jsonwebtoken";
import { PUBLIC_APIURL } from '$env/static/public';
import { redirect} from '@sveltejs/kit';
import { constructCookieOpts } from '$lib/directus';

const TOKEN_EXPIRATION_BUFFER = 300;

// exchange the refresh token for an access token
async function refreshAccessToken(cookies) {
    let res = await fetch(PUBLIC_APIURL + "/auth/refresh", {
      method: "POST",
      mode: "cors",
      headers: {
        Accept: "application/json, text/plain, */*",
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ refresh_token: cookies.get('refresh_token') }),
    });
  
    if (res.status != 200) {
      cookies.delete('refresh_token');
      cookies.delete('access_token');
      throw new Error("Refresh Token Status != 200");
    }
    let data = (await res.json()).data;
  
    cookies.set("refresh_token", data.refresh_token, constructCookieOpts(60 * 60 * 24 * 30));
    cookies.set("access_token", data.access_token, constructCookieOpts(Math.floor(data.expires/1000)));
  }

function isTokenExpired(jwtPayload){
    return jwtPayload?.exp < Math.floor(Date.now()/1000) + TOKEN_EXPIRATION_BUFFER;
}

function shouldProtectRoute(url) {
    return url.split("/").includes("(protected)")
}

export async function handle({event, resolve}) {
    const { cookies,url } = event
    
    if (cookies.get('access_token') || cookies.get('refresh_token')) {
        let jwtPayload = cookies.get('access_token') ? jwt.decode(cookies.get('access_token')) : false;
  
        //check if token is expired and renew it if necessary
        if (isTokenExpired(jwtPayload) || !cookies.get('access_token')) {
          try {
            await refreshAccessToken(cookies);
            jwtPayload = cookies.get('access_token') ? jwt.decode(cookies.get('access_token')) : false;
          } catch (err) {
            cookies.delete('refresh_token');
            cookies.delete('access_token');
          }
        }
        
        event.locals.user = jwtPayload?.id;
        event.locals.token = cookies.get('access_token');
    }

    if (event.route.id && shouldProtectRoute(event.route.id) && !event.locals.user) {
        throw redirect(302,`/signin?redirectedFrom=${encodeURIComponent(url.pathname)}`)
    }

    return await resolve(event, {
        filterSerializedResponseHeaders: (key, value) => {
            return key.toLowerCase() === 'content-type'
        }
    });
}
```

Every single request goes passtrough this file. It checks the tokens and if valid sets the user id and the token to the locals object, which can be accessed througout SvelteKits Load Functions. If the Access Token is expired, a new one will be generated. Finally we can define routes under the `(protected)` Directory. The "protected" keyword will not appear in the url but is only visible in our file structure.

In order for the authentication to work however, we also need to tell SvelteKit to actually pass those local variables through.

Create a file `/+layout.server.js`:

```js
/** @type {import('./$types').LayoutServerLoad} */
export async function load({locals}) {
	return {
		user: locals.user,
		token: locals.token
	};
}
```

## Create Profile Page

The locals will now be used in every `+page.js` file to initialize the Directus SDK with the users session token. To test this out, let's now create our `profile` page, which is only accessible if the user is logged in.

Create a file `/(protected)/profile/+page.js`

```js [/(protected)/profile/+page.js]
import getDirectusInstance from "$lib/directus";
import { readMe } from "@directus/sdk";
export async function load({ parent, fetch }) {
  const { token } = await parent();
  const directus = getDirectusInstance(fetch, token);
  try {
    return {
      user: await directus.request(
        readMe({
          fields: ["*"],
        })
      ),
    };
  } catch (err) {
    throw error(404, "User not found");
  }
}

```

As you see we are getting the token and initialize our Directus Instance as usual. This time however we also give it the Access Token so that every request will now have the Users Session attached. In this case we are reading the users profile. If you try this without the token the request will fail because of missing permissions.

Let's continue writing the Html Template:

```svelte [/(protected)/profile/+page.svelte]
<script>
  export let data;
</script>
  
<p>This is only visible if you are logged in</p>

<p>Your Email: {data.user.email}</p>
```


### Authenticated requests on the client and logging the user out

Until now we have used the Directus SDK solely in the load functions. In order to also use it on the client we can define a global context object, which we can use on every svelte component. For this the `+layout.svelte` file needs to be adapted:

```svelte
<script>
    export let data;
    import getDirectusInstance from '$lib/directus'; // [!code ++]
	  import { setContext } from 'svelte'; // [!code ++]

    // Make directus available to all components via Context API // [!code ++]
    const directus = getDirectusInstance(fetch,data.token) // [!code ++]
    setContext('directus', directus); // [!code ++]
</script>

<a href="/">Home</a>
<a href="/pages/about">About</a>
<a href="/pages/conduct">Code of Conduct</a>
<a href="/pages/privacy">Privacy Policy</a>
<a href="/blog">Blog</a>

{#if data.token} // [!code ++]
    <a href="/logout" data-sveltekit-preload-data="off">logout</a> // [!code ++]
{:else} // [!code ++]
    <a href="/signin">signin</a> // [!code ++]
{/if} // [!code ++]

<div>
    <slot></slot>
</div>
```

As you see we also change the layout based on the login state and already add a logout endpoint for later usage.

A simple example to use the Directus SDK on the client now is to change the users email in the profile page. For this let's revisit the profile page and adapt it:

```svelte [/(protected)/profile/+page.svelte]
<script>
	import { getContext } from 'svelte'; // [!code ++]
  import { updateMe } from '@directus/sdk'; // [!code ++]
  export let data;

  const directus = getContext('directus'); // [!code ++]

  async function changeEmail() { // [!code ++]
    await directus.request(updateMe({email:data.user.email})) // [!code ++]
  } // [!code ++]
</script>
  
<p>This is only visible if you are logged in</p>

<p>Your Email: {data.user.email}</p>

<div> // [!code ++]
  <label for="email">Your E-Mail</label> // [!code ++]
  <input bind:value={data.user.email} required type="email" autocomplete="email" autocapitalize="off" /> // [!code ++]
  <button on:click={changeEmail}>Change Email</button> // [!code ++]
</div> // [!code ++]
```

Now go to /profile. testing out.....

Lastly let's define a server endpoint to enable logout:

```js [/(protected)/logout/+server.js]
import { redirect,fail } from '@sveltejs/kit';
import { PUBLIC_APIURL } from '$env/static/public';

export async function GET({locals,request,cookies}) {
  try {
    if(cookies.get('refresh_token'))
      await fetch(`${PUBLIC_APIURL}/auth/logout`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'user-agent':request.headers.get("user-agent")
        },
        body: JSON.stringify({ refresh_token: cookies.get('refresh_token') })
      });
  } catch (err) {
    throw fail(400,err);
  }
  
  cookies.delete('refresh_token');
  cookies.delete('access_token');

  throw redirect(302,`/signin`);
}
```

This will simple delete the cookies and also calls the Directus API to invalide the stored session in the DB. Then redirecting the user back to the login page.

# Conclusion

In this Guide ...