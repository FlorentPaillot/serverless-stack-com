---
layout: post
title: Add the Session to the State
date: 2017-01-15 00:00:00
lang: en
comments_id: add-the-session-to-the-state
redirect_from: /chapters/add-the-user-token-to-the-state.html
description: We need to add the user session to the state of our App component in our React.js app. We are going to use React context through the useContext hook to store it and pass it to all our child components. 
comments_id: add-the-session-to-the-state/136
---

To complete the login process we would need to update the app state with the session to reflect that the user has logged in.

### Update the App State

First we'll start by updating the application state by setting that the user is logged in. We might be tempted to store this in the `Login` container, but since we are going to use this in a lot of other places, it makes sense to lift up the state. The most logical place to do this will be in our `App` component.

To save the user's login state, let's include the `useState` hook in `src/App.js`.

<img class="code-marker" src="/assets/s.png" />Replace, the `React` import:

``` javascript
import React from "react";
```

<img class="code-marker" src="/assets/s.png" />With the following:

``` javascript
import React, { useState } from "react";
```

<img class="code-marker" src="/assets/s.png" />Add the following to the top of our `App` component function.

``` javascript
const [isAuthenticated, userHasAuthenticated] = useState(false);
```

This initializes the `isAuthenticated` state variable to `false`, as in the user is not logged in. And calling `userHasAuthenticated` updates it. But for the `Login` container to call this method we need to pass a reference of this method to it.

### Store the Session in the Context

We are going to have to pass the session related info to all of our containers. This is going to be tedious if we pass it in as a prop. Since we'll have to do that manually for each component. Instead let's use [React Context](https://reactjs.org/docs/context.html) for this.

We'll create a context for our entire app that all of our containers will use.

<img class="code-marker" src="/assets/s.png" />Create a `src/libs/` directory. We'll use this to store all our common code.

``` bash
$ mkdir src/libs/
```

<img class="code-marker" src="/assets/s.png" />Add the following to `src/libs/contextLib.js`.

``` javascript
import { useContext, createContext } from "react";

export const AppContext = createContext(null);

export function useAppContext() {
  return useContext(AppContext);
}
```

This really simple bit of code is creating and exporting two things:
1. Using the `createContext` API to create a new context for our app.
2. Using the `useContext` React Hook to access the context.

If you are not sure how Contexts work, don't worry, it'll make more sense once we use it.

<img class="code-marker" src="/assets/s.png" />Import our new app context in the header of `src/App.js`.

``` javascript
import { AppContext } from "./libs/contextLib";
```

Now to add our session to the context and to pass it to our containers:

<img class="code-marker" src="/assets/s.png" />Wrap our `Routes` component in the `return` statement of `src/App.js`.

``` coffee
<Routes />
```

<img class="code-marker" src="/assets/s.png" />With this.

{% raw %}
``` coffee
<AppContext.Provider value={{ isAuthenticated, userHasAuthenticated }}>
  <Routes />
</AppContext.Provider>
```
{% endraw %}

React Context's are made up of two parts. The first is the Provider. This is telling React that all the child components inside the Context Provider should be able to access what we put in it. In this case we are putting in the following object:

``` javascript
{ isAuthenticated, userHasAuthenticated }
```

### Use the Context to Update the State

The second part of the Context API is the consumer. We'll add that to the Login container:

<img class="code-marker" src="/assets/s.png" />Start by importing it in the header of `src/containers/Login.js`.

``` javascript
import { useAppContext } from "../libs/contextLib";
```

<img class="code-marker" src="/assets/s.png" />Include the hook by adding it below the `export default function Login() {` line.

``` javascript
const { userHasAuthenticated } = useAppContext();
```

This is telling React that we want to use our app context here and that we want to be able to use the `userHasAuthenticated` function.

<img class="code-marker" src="/assets/s.png" />Finally, replace the `alert('Logged in');` line with the following in `src/containers/Login.js`.

``` javascript
userHasAuthenticated(true);
```

### Create a Logout Button

We can now use this to display a Logout button once the user logs in. Find the following in our `src/App.js`.

``` coffee
<LinkContainer to="/signup">
  <NavItem>Signup</NavItem>
</LinkContainer>
<LinkContainer to="/login">
  <NavItem>Login</NavItem>
</LinkContainer>
```

<img class="code-marker" src="/assets/s.png" />And replace it with this:

``` coffee
{isAuthenticated
  ? <NavItem onClick={handleLogout}>Logout</NavItem>
  : <>
      <LinkContainer to="/signup">
        <NavItem>Signup</NavItem>
      </LinkContainer>
      <LinkContainer to="/login">
        <NavItem>Login</NavItem>
      </LinkContainer>
    </>
}
```

The `<>` or [Fragment component](https://reactjs.org/docs/fragments.html) can be thought of as a placeholder component. We need this because in the case the user is not logged in, we want to render two links. To do this we would need to wrap it inside a single component, like a `div`. But by using the Fragment component it tells React that the two links are inside this component but we don't want to render any extra HTML.

<img class="code-marker" src="/assets/s.png" />And add this `handleLogout` method to `src/App.js` above the `return` statement as well.

``` javascript
function handleLogout() {
  userHasAuthenticated(false);
}
```

Now head over to your browser and try logging in with the admin credentials we created in the [Create a Cognito Test User]({% link _chapters/create-a-cognito-test-user.md %}) chapter. You should see the Logout button appear right away.

![Login state updated screenshot](/assets/login-state-updated.png)

Now if you refresh your page you should be logged out again. This is because we are not initializing the state from the browser session. Let's look at how to do that next.
