---
title: Zustand for state management
date: "2020-10-20"
spoiler: Zustand Intro
cta: "none"
---

Are you happy with your current state management library in your react application? As software engineers, we are always looking for simpler, more powerful libraries.

When it comes to state management, there are many existing library e.g., Redux, MobX etc. These have significant learning curve and mostly used to manage application state including server state (e.g., data returned from the REST/GraphQL APIs) and client state (e.g., user information, theme selection etc).

Once you've handled server state in a library like ract query. You are left with very little state on the client side.

React's Context API is the default choice for managing clinet state and sharing it between components, but it feels like adding boilerplate without much advantage.

## Context API usage

To use the Context API, you need to create context and expose the provider.

```jsx
// UserContext.js
import React from "react";

const UserInfoContext = React.createContext({});

export const UserInfoProvider = UserInfoContext.Provider;

export default UserInfoContext;
```

Next you need to wrap your main component (or any sub component that need to have access to this state) with the provider. You can also initiate the state here e.g., in the below code, we are only going to manage a user name in the state.

```jsx
import UserInfoContext from "../state/UserContext";

<UserInfoContext.Provider value={{ userName: "" }}>
  <MyComponent {...pageProps} />
</UserInfoContext.Provider>;
```

And finally you can update or read this state in any of the components in the tree. In order to use this context, you can use built-in useContext hook from react.

```jsx
import UserInfoContext from '../state/UserContext'
import { useContext } from 'react'

const userContext = useContext(UserInfoContext)

// display userName
<h1>User Name {userContext.userName}</h1>

// set userName e.g., in login handler
userContext.userName = loggedinUserName
```

As you can see, it requires wrapping you component tree in Provider and then you can use a useContext hook to manage the state. One of the drawback is that you can set the state from anywhere.
This is unlike a managed store that only allows to set the state from a reducer.

## Zustand to the rescue

Zustand is a relatively new library that offers a very elegant state management solution which is based on hooks. it has a very simple API and you can avoid boilerplate code.

So first step is always to add the required modules.

```jsx
yarn add zustand redux-devtools-extension
```

Zustand has a create function that returns a hook. The function uses a set function parameter to change the state and as you will see in the later snippers, the state can not be minipulated directly (unlike Context API).

In the below snippet, I've wrapped the function with a call to devtools function. This is the middleware function from zustand that can be used to integrate with Redux devtools chrome plugin.

```jsx
// UserStore.js
import create from "zustand";
import { devtools } from "zustand/middleware";

export const useUserStore = create(
  devtools((set) => ({
    userName: "",
    loggedIn: false,
    login: (_user) =>
      set({
        loggedIn: true,
        userName: _user,
      }),
    logout: () =>
      set({
        loggedIn: false,
        userName: "",
      }),
  }))
);
```

Next we can use this hook to read the state or update the state using the provided functions.

```jsx
import { useUserStore } from '../stores/UserStore'

const { login, loggedIn, userName } = useUserStore((state) => ({ ...state }))

// display userName
<h1>User Name {userName}</h1>

// update state in login handler
login(loggedInUserName)
```

## Zustand advantages

- Simple state management solution
- Elegant API based on hooks
- Devtools integration

##Resources##

- [Zustand github repo](https://github.com/pmndrs/zustand)
- [React Query: It’s Time to Break up with your "Global State”! –Tanner Linsley](https://www.youtube.com/watch?v=seU46c6Jz7E&ab_channel=ReactConferencesbyGitNation)
- [React Context](https://reactjs.org/docs/context.html).
- [Using React Context with Functional Components](https://medium.com/@danfyfe/using-react-context-with-functional-components-153cbd9ba214)

<hr/>
