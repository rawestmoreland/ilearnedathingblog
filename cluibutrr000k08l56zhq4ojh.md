---
title: "Secure React Native App Authentication with PocketBase: A Step-by-Step Guide"
seoTitle: "Secure React Native App Authentication with PocketBase: A Step-by-Step"
seoDescription: "Master React Native app security with our concise guide on implementing seamless authentication using PocketBase. Elevate your app's protection now."
datePublished: Tue Apr 02 2024 12:00:21 GMT+0000 (Coordinated Universal Time)
cuid: cluibutrr000k08l56zhq4ojh
slug: secure-react-native-app-authentication-with-pocketbase-a-step-by-step-guide
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1711975632028/9e2ace62-476f-482c-bf52-a55cc0320061.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1712056661999/94d7c1cc-1ec2-40ed-b4aa-234305b467af.jpeg
tags: tutorial, authentication, react-native, reactjs, expo, pocketbase, expo-router

---

### What is PocketBase?

[PocketBase](https://pocketbase.io/docs) is an awesome little backend as a service (BaaS) that runs on Go. Its feature set includes an embedded (SQLite) database with real-time subscriptions and built-in auth management.

Client-side SDKs are available for both Dart and JavaScript, which makes it incredibly easy to get up and running within a React Native or Flutter mobile project.

### Project Setup

In this tutorial, we're going to build a simple app that includes a login screen, an account creation screen, and a few screens that require the user to be authenticated to view.

Let's start by setting up a minimal React Native Expo application using the expo-router "tabs" template.

```bash
npx create-expo-app@latest
```

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Pro tip - see a list of expo templates by running <code>npx create-expo-app --templates</code></div>
</div>

We'll also need some other packages to make all of this work. Let's install `pocketbase`, and `@react-native-async-storage/async-storage`. We'll need AsyncStorage to persist sessions within React Native. This is referenced [here in the JavaScript SDK docs](https://github.com/pocketbase/js-sdk?tab=readme-ov-file#asyncauthstore).

Since we're using Expo, use this command to install the packages:

```bash
npx expo install pocketbase @react-native-async-storage/async-storage
```

### Create a PocketBase context provider and hook

Use a context provider and a custom hook to initialize our PocketBase instance within the app. Putting this in a context and provider will allow us to easily access our PocketBase instance anywhere in the app.

Create a folder at the root of the project and name it `context`. Within that folder, create a file named `pocketbase.jsx`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1711977446155/fa68b1cb-5738-45de-9895-531ade37334d.png align="center")

In `pocketbase.jsx`:

```javascript
// pocketbase.js

import React, { createContext, useContext, useState, useEffect } from 'react';
import PocketBase, { AsyncAuthStore } from 'pocketbase';
import AsyncStorage from '@react-native-async-storage/async-storage';

const PocketBaseContext = createContext();

export const usePocketBase = () => useContext(PocketBaseContext);

export const PocketBaseProvider = ({ children }) => {
  const [pb, setPb] = useState();

  useEffect(() => {
    const initializePocketBase = async () => {
      // This is where our auth session will be stored. It's PocketBase magic.
      const store = new AsyncAuthStore({
        save: async (serialized) => AsyncStorage.setItem('pb_auth', serialized),
        initial: await AsyncStorage.getItem('pb_auth'),
        clear: async () => AsyncStorage.removeItem('pb_auth'),
      });
      const pbInstance = new PocketBase('<your-pocketbase-url>', store);
      setPb(pbInstance);
    };

    initializePocketBase();
  }, []);

  return (
    <PocketBaseContext.Provider value={{ pb }}>
      {children}
    </PocketBaseContext.Provider>
  );
};
```

You'll notice that when we initialize PocketBase, we instantiate a `new AsyncAuthStore`. This is responsible for storing our user's session data in `AsyncStorage`. We then pass that `store` to the `new Pocketbase` instance.

In `app/_layout.jsx`, we'll wrap the `RootLayout` with the provider we just created.

```javascript
// app/_layout.jsx

import FontAwesome from '@expo/vector-icons/FontAwesome';
import {
  DarkTheme,
  DefaultTheme,
  ThemeProvider,
} from '@react-navigation/native';
import { useFonts } from 'expo-font';
import { Stack } from 'expo-router';
import * as SplashScreen from 'expo-splash-screen';
import React, { useEffect } from 'react';
import { PocketBaseProvider } from '@/context/pocketbase';

import { useColorScheme } from '@/components/useColorScheme';

export {
  // Catch any errors thrown by the Layout component.
  ErrorBoundary,
} from 'expo-router';

export const unstable_settings = {
  // Ensure that reloading on `/modal` keeps a back button present.
  initialRouteName: '(tabs)',
};

// Prevent the splash screen from auto-hiding before asset loading is complete.
SplashScreen.preventAutoHideAsync();

export default function RootLayout() {
  const [loaded, error] = useFonts({
    SpaceMono: require('../assets/fonts/SpaceMono-Regular.ttf'),
    ...FontAwesome.font,
  });

  // Expo Router uses Error Boundaries to catch errors in the navigation tree.
  useEffect(() => {
    if (error) throw error;
  }, [error]);

  useEffect(() => {
    if (loaded) {
      SplashScreen.hideAsync();
    }
  }, [loaded]);

  if (!loaded) {
    return null;
  }

  return <RootLayoutNav />;
}

function RootLayoutNav() {
  const colorScheme = useColorScheme();

  return (
    <PocketBaseProvider>
      <ThemeProvider value={colorScheme === 'dark' ? DarkTheme : DefaultTheme}>
        <Stack>
          <Stack.Screen name='(tabs)' options={{ headerShown: false }} />
          <Stack.Screen name='modal' options={{ presentation: 'modal' }} />
        </Stack>
      </ThemeProvider>
    </PocketBaseProvider>
  );
}
```

### Create an auth context provider and hook

We want to access the user's authentication status from anywhere in the app. Let's create an `AuthContext` and a `useAuth` hook that will export our user's authentication status and the methods needed to sign in, create an account, and sign out.

Create a file named `auth.jsx` in the `context` folder:

```javascript
// context/auth.jsx

import { useSegments, useRouter, useNavigationContainerRef } from 'expo-router';
import { useState, useEffect, createContext, useContext } from 'react';
import { usePocketBase } from './pocketbase';

const AuthContext = createContext({});

// This hook can be used to access the user info.
export function useAuth() {
  return useContext(AuthContext);
}

function useProtectedRoute(user, isInitialized) {
  const router = useRouter();
  const segments = useSegments();

  // Check that navigation is all good
  const [isNavigationReady, setIsNavigationReady] = useState(false);
  const rootNavRef = useNavigationContainerRef();

  // Set ups a listener to check and see if the navigator is ready.
  useEffect(() => {
    const unsubscribe = rootNavRef?.addListener('state', (event) => {
      setIsNavigationReady(true);
    });
    return function cleanup() {
      if (unsubscribe) {
        unsubscribe();
      }
    };
  }, [rootNavRef.current]);

  useEffect(() => {
    // Navigation isn't set up. Do nothing.
    if (!isNavigationReady) return;
    const inAuthGroup = segments[0] === '(auth)';

    if (!isInitialized) return;

    if (
      // If the user is not signed in and the initial segment is not anything in the auth group.
      !user &&
      !inAuthGroup
    ) {
      // Redirect to the sign-in page.
      router.replace('/(auth)/login');
    } else if (user && inAuthGroup) {
      // Redirect away from the sign-in page.
      router.replace('/(tabs)');
    }
  }, [user, segments, isNavigationReady, isInitialized]);
}

export function AuthProvider({ children }) {
  const { pb } = usePocketBase();
  const [isInitialized, setIsInitialized] = useState(false);
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  const [user, setUser] = useState(null);

  useEffect(() => {
    const checkAuthStatus = async () => {
      if (pb) {
        // Assuming your PocketBase setup includes some method to check auth status
        const isLoggedIn = pb.authStore.isValid;
        setIsLoggedIn(isLoggedIn);
        setUser(isLoggedIn ? pb.authStore.model : null);
        setIsInitialized(true);
      }
    };

    checkAuthStatus();
  }, [pb]);

  const appSignIn = async (email, password) => {
    if (!pb) return { error: 'PocketBase not initialized' };

    try {
      const resp = await pb
        ?.collection('users')
        .authWithPassword(email, password);
      setUser(pb?.authStore.isValid ? pb.authStore.model : null);
      setIsLoggedIn(pb?.authStore.isValid ?? false);
      return { user: resp?.record };
    } catch (e) {
      return { error: e };
    }
  };

  const appSignOut = async () => {
    if (!pb) return { error: 'PocketBase not initialized' };

    try {
      await pb?.authStore.clear();
      setUser(null);
      setIsLoggedIn(false);
      return { user: null };
    } catch (e) {
      return { error: e };
    }
  };

  const createAccount = async ({ email, password, passwordConfirm, name }) => {
    if (!pb) return { error: 'PocketBase not initialized' };

    try {
      const resp = await pb.collection('users').create({
        email,
        password,
        passwordConfirm,
        name: name ?? '',
      });

      return { user: resp };
    } catch (e) {
      return { error: e.response };
    }
  };

  useProtectedRoute(user, isInitialized);

  return (
    <AuthContext.Provider
      value={{
        signIn: (email, password) => appSignIn(email, password),
        signOut: () => appSignOut(),
        createAccount: ({ email, password, passwordConfirm, name }) =>
          createAccount({ email, password, passwordConfirm, name }),
        isLoggedIn,
        isInitialized,
        user,
      }}
    >
      {children}
    </AuthContext.Provider>
  );
}
```

A lot is going on in there! I commented on some of the items that may not be as clear as others. One of the things I struggled with was the navigation state not being initialized *before* checking for the auth state of the user. After doing some digging in the expo router docs, I found that `useNavigationContainerRef()` is useful for checking to see if the `<NavigationContainer />` has mounted.

The high level of the flow in this context is:

1. Wait for the navigation to mount
    
2. Check and see if we're in the `(auth)` segment. (This is the part of the app that requires no auth).
    
3. If we're *not* logged in and *not* in the (auth) segment, redirect to `login` .
    
4. If we're logged in and we're in the (auth) segment, redirect to `home` .
    
5. Export our user, logged-in state, and auth methods.
    

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text"><code>useProtectedRoute</code> performs the check for steps 2-4. This check will be performed on every render.</div>
</div>

Wrap the root `_layout.jsx` with the `AuthProvider`

```javascript
// _layout.jsx
//.....other code

function RootLayoutNav() {
  const colorScheme = useColorScheme();

  return (
    <PocketBaseProvider>
      <AuthProvider>
        <ThemeProvider
          value={colorScheme === 'dark' ? DarkTheme : DefaultTheme}
        >
          <Stack>
            <Stack.Screen name='(tabs)' options={{ headerShown: false }} />
            <Stack.Screen name='modal' options={{ presentation: 'modal' }} />
          </Stack>
        </ThemeProvider>
      </AuthProvider>
    </PocketBaseProvider>
  );
}
```

### Set up our app structure and root index.jsx

As described above, we need to separate our app into "segments". One segment will be screens that don't require auth - log in, create an account, reset your password, etc. The other segment will be "protected routes". Or, screens that require us to be logged in.

Things are *ok* right now, but there's a little "flash" of the home screen when we start the app.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1712053918786/76483d0b-6f89-43ba-bdb1-6b29507b8744.gif align="center")

We can fix this by including a `index.jsx` file at the root of our app that uses the `isInitialized` value exported from the auth context.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1712054069809/70dac200-327d-48af-9d49-efb1250f708c.png align="center")

```javascript
// app/index.jsx

import { View, ActivityIndicator } from 'react-native';
import { useAuth } from '@/context/auth';
import { useRootNavigationState, useRouter, useSegments } from 'expo-router';
import { useEffect } from 'react';

export default function Index() {
  const { isInitialized, isLoggedIn } = useAuth();

  const router = useRouter();
  const segments = useSegments();
  const navigationState = useRootNavigationState();

  useEffect(() => {
    if (!isInitialized || !navigationState?.key) return;

    const inAuthGroup = segments[0] === '(auth)';

    if (
      // If the user is not signed in and the initial segment is not anything
      // segment is not anything in the auth group.
      !isLoggedIn &&
      !inAuthGroup
    ) {
      // Redirect to the login page.
      router.replace('/(auth)/login');
    } else if (isLoggedIn) {
      // go to tabs root.
      router.replace('/(tabs)');
    }
  }, [segments, navigationState?.key, isInitialized]);

  return (
    <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
      {!navigationState?.key ? <ActivityIndicator /> : <></>}
    </View>
  );
}
```

A lot of this should look familiar. It follows the same pattern as the `AuthContext`but performs the checks on app launch. Screen-flash eliminated!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1712054536825/2e6903cb-ff49-4eae-8ded-8ff9f4f25211.gif align="center")

### That's it!

I won't cover the login and account creation UI. You can find all of the code for this article on [GitHub HERE](https://github.com/rawestmoreland/react-native-pb-auth)!

If you loved this article, please let me know! Press the like button below, send me a DM on Twitter [@ctrlaltideate](https://twitter.com/ctrlaltideate), or [buy me a coffee](https://www.buymeacoffee.com/westmorelandcreative) 🤗.

### References

[PocketBase docs](https://pocketbase.io/docs)

[PocketBase JavaScript SDK docs](https://github.com/pocketbase/js-sdk?tab=readme-ov-file#pocketbase-javascript-sdk)

[Expo Router Quick Start](https://docs.expo.dev/router/installation/#quick-start)