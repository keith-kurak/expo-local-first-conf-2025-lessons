# Module 01: Getting started with Expo and Livestore

### Goal

Let's dig into our monorepo containing a Cloudflare durable object worker backend, Vite web frontend, and an Expo cross-platform native mobile app. These all connect to each other to provide a universal local first solution with cloud sync.

### Concepts

- How to get started with local development in Expo
- The basics of running and debugging a universal Livestore app
- Essential information about how Livestore works with Cloudflare durable objects

### Tasks

- Get the Expo frontend running on your device, pointing to Livestore on our workshop's Cloudflare durable object worker.
- Switch over to local development, running the Cloudflare worker locally and connecting to it the web and native mobile frontends.
- Watch Livestore syncing in action through the eyes of the debugging tools.

# Exercises

## Exercise 1: Local development, cloud sync

Let's get the mobile app running in everyone's local environment, pointing to the workshop's Cloudflare durable object instance.

To keep things going fast, we're going to use Expo Go to simulate a local React Native development environment. Expo Go is a sandbox environment that can be downloaded right from the App Store, so you don't need to build an app or have an Apple Developer account to get started with building your app. It's a _fixed runtime_, so it can only run certain React Native code, but fortunately everything we're doing today runs in Expo Go.

### Initialize the environment and run the app

1. Run `pnpm install` in the root directory.
2. Run `cd packages/mobile` to switch to the mobile app directory.
3. Run `cp .env.local.example .env.local` to copy the example environment configuration to a local **.env** file.

Notice in the **.env** file is a variable called `EXPO_PUBLIC_LIVESTORE_SYNC_URL`. Make sure this is set to `https://<insert prod url here>`. Variables that start with `EXPO_PUBLIC_` are inlined directly into your code by Metro bundler when you reference them via `process.env`.

4. Run `pnpm run dev` to start the local development environment.
5. Scan the QR Code that appears in the console with your phone.

🏃**Try it.** You're running the app now, and it's connected to the workshop's shared Livestore instance!

### Test-driving Livestore

We have implemented a quick-and-dirty simulation of authentication - you just enter a name in order to fill the "auth token" and cause the sync server to recognize you (more on this later). So, enter a name and try posting something (be nice with your posts, everyone else can see them!).

### Try out the devtools

Livestore includes devtools that let you inspect the state of the sync engine. Make sure you're back online again. Then:

1. In the console, press `Shift + M`.
2. Choose `@livestore/devtools-expo`.

This should open up a browser. Expo devtools plugins are small web apps within your Expo project or an Expo library that can communicate back and forth with your app and tell you information about it.

Let's try some things with the devtools:

#### Browse data in the data browser

This will show you actual tables. You can see the current state of your table, with the schema just as you defined it. You can see the definition of these tables in **packages/shared/schema.ts**.

#### Queries

Go to Queries -> Reactivity Graph, and then create or edit a note. Watch nodes appear and dissappear. This is showing where queries are being re-run live as data is being updated, causing it to refresh in the UI.

In Live Queries, you can see some of the queries that map to the queries in **packages/shared/queries.ts**. These are queries that are predefined and can be fed with a simple variabled into `useQuery`, which then defines some data in a view that you can react to. You can also define a query inline in a `useQuery` hook.

#### Events

Look at the sequence of events as you enter data and others update data, as well. These map to materializers defined in your Livestore configuration. You can see these in **packages/shared/schema.ts**. Each materializer defines a way data can be changed.

#### Sync

You can pause and restart the sync at any time here. Thus, you can simulate a disconnect while still having your phone connected to your local development server and dev tools. Try flipping it on and off while adding notes.

## Exercise 2: Fully-local development

Let's move things over to our own fully-local dev environment, running the mobile app, web app, and cloudflare durable object worker all within your local environment, so you can develop without affecting live data, or even needing anything deployed.

Note that the app isn't entirely responsive yet (you're be working on that!). So, it might be easiest to shrink the app down to mobile size by opening Chrome Devtools or resizing your browser window.

## Exercise 1: Routing and layout basics

We're adding a new screen with info about visiting the museum... but where? Let's try a few places, and learn some Expo Router basics along the way.

_This short exercise is helpful if you're pretty new to Expo Router, but doesn't factor into the final app, so if you're already familiar with stack layouts and route groups, you can skip to Exercise 2._

1. Create a **visit.tsx** file in the **(app)** folder. Copy our [**visit.tsx** starter here](/files/01/visit.tsx) for its contents. `(app)` is a _route group_, representing the "logged-in" experience of the app (this implies a "logged out" experience; we'll get back to that).
2. In your web browser, try navigating to it by using the URL bar, by entering: `http://localhost:8081/visit`. Notice how the route group folder doesn't factor in the URL. These are just for organizing routes and defining their behavior without affecting the URL.
3. Try adding a `Link` to the "visit" page. The header of the **index** page in **app/(app)/(tabs)/index.tsx** will work:

```tsx
<Link href="/visit">Visit</Link>
```

Don't forget to `import { Link } from "expo-router";`

4. Tap on the link in the browser or the mobile app. It should take you to the "visit" page, and you should be able to go back to the main tabs. This is because the **\_layout.tsx** inside the **(app)** folder groups the routes at this level into a _stack navigator_.
5. Get rid of that `Link` - we're going to put it somewhere else

### Not found routes.

There's a special route at the top of your **app** folder (which is the folder that always contains your Expo Router routes): **not-found.tsx**
This route will render if your app navigates to a route that doesn't exist. 6. Try typing a non-existent route into the browser, e.g., `http://localhost:8081/slartibartfast`. Where does it go?

## Exercise 2: Add the Visit page to the tab navigator.

We've decided it would be best to have information for visitors as a separate tab.

1. Create a **visit.tsx** file in the **(app)/(tabs)** folder. Copy our [**visit.tsx** starter here](/files/01/visit.tsx) for its contents. (or, if you already created **visit.tsx** in Exercise 1, just drag it into **(tabs)**)
2. Refresh your browser / reload the app to see the new tab. Now check out **(app)/(tabs)/\_layout.tsx**. This layout defines a tab navigator, so it's saying any routes at this level should be arranged as tabs.
3. The Visit tab looks pretty basic right now. Not the right icon or text. We can information in **\_layout.tsx** to change the behavior and appearance of the tab. Add this between the exhibits and profile tab:

```tsx
<Tabs.Screen
  name="visit"
  options={{
    title: "Visit",
    tabBarIcon: ({ color }) => (
      <TabBarIcon type="MaterialIcons" name="map" color={color} />
    ),
  }}
/>
```

🏃**Try it.** Now the tab should look pretty good.

## Exercise 3: Navigating to arbitrary things with dynamic routes

Dynamic routes let you put a variable into your URL, that you can then query on the destination screen. They are defined with square brackets, e.g., `[id]`.

There's already a dynamic route in the app: `/exhibits/[exhibitName]`. When you navigate to that route, the URL will contain the actual exhibit, e.g., `/exhibits/Textiles`

Notice how the artworks don't go anywhere yet when you click on them. Let's fix that.

1. Create the file **app/(app)/works/[workId].tsx** (add the extra folder, of course). Use the [**[workId].tsx** starter here](/files/01/[workId].tsx).
2. In **Artwork.tsx** (shared by both screens), wrap the artwork `Pressable` in a `Link`, navigating to `works/[workId]`:

```diff
+  <Link asChild href={`/works/${artwork.id}`}>
     <Pressable className="flex-row sm:flex-col gap-x-2 h-48 sm:h-96">
        // ...
     </Pressable>
+  </Link>
```

🏃**Try it.** You should be able to navigate to artwork now. Try a few.

Oh wait, it's going to the same work every time. That's because we didn't read the route parameter inside the **[workId].tsx** screen itself.

3. Add this at the top of **[workId].tsx**, replacing the hardcoded `workId`:

```tsx
const { workId } = useLocalSearchParams<{
  workId: string;
}>();
```

Don't forget to `import { useLocalSearchParams } from "expo-router";`

🏃**Try it.** You should now be able to navigate to the correct artwork from both the Home and Exhibits tabs.

### About the elephant in the header...

By now, you're probably quite annoyed by that header with route garbage at the top of your screen. You could set `title` in screen options to at least make it something nice, but you'd still have double headers! We've clearly implemented our own header in **[workId].tsx**, so let's get rid of React Navigation's.

4. In **app/(app)/\_layout.tsx**, add a reference to `works/[workId]`, so you can hide the header with screen options:

```tsx
<Stack.Screen
  name="works/[workId]"
  options={{
    headerShown: false,
  }}
/>
```

## Exercise 4: Nested routes

Notice that not everything in that tabs route is just a single file. There's the **exhibits** folder, with multiple routes. This is a stack nested inside a tab.

Let's experiment with a few scenarios here. **Refresh the browser / shake and refresh your app before each one to reset navigation history**:

- **(A)** Exhibits tab -> click on an exhibit -> go back
- **(B)** (browser-only) Home tab -> click on an exhibit -> reload page -> go back (can you do it?)

You might have had some problems with B). Expo Router has no idea that the "first" page of the Exhibits tab should always be stacked below `[exhibitName]`, even when the app is reloaded on the page. How could it?

The `initialRouteName` configuration lets you specify a child route that should exist in history _if the app is first loaded on one of its sibling routes_. That sounds tailor-made for B).

2. Add the following to **(app)/(tabs)/exhibits/\_layout.tsx**:

```tsx
export const unstable_settings = {
  // Ensure any route can link back to `/`
  initialRouteName: "index",
};
```

This says that, when inside the **exhibits** route, the index should be the first route. The ensures that, even if you go directly to a exhibit, the exhibits list will be underneath it.

🏃**Try it.** **(B)** should work better now.

## Exercise 5: A little bit of responsiveness

Head over to your browser, and make the window wide. Now open an artwork. Real "blown up mobile app" vibes, right? Let's use some navigator props and creative styling to make this a centered modal on web, floating above the underlying content.

1. In **app/(app)/layout.tsx**, add this code just inside the options for this screen:

```diff
<Stack.Screen
   name="works/[workId]"
-  options={{
-    headerShown: false,
-  }}
+ options={{
+  ...Platform.select({
+    web: {
+      presentation: "transparentModal",
+     animation: "fade",
+    },
+    default: {
+      presentation: "modal",
+   },
+  }),
+ headerShown: false,
+ }}
/>
```

Don't forget to `import { Platform } from "react-native";`

<details>
  <summary>Expand to just get the whole function for easy copying</summary>

```tsx
options={{
  ...Platform.select({
    web: {
      presentation: "transparentModal",
      animation: "fade",
    },
    default: {
      presentation: "modal",
    },
  }),
  headerShown: false,
}}
```

</details>

This will make this screen present like a transparent modal, but it will not look very obvious (minus the previous screen exposed underneath) until we shrink the size of the content, so the transparent area will show up underneath it.

Let's use Nativewind's _responsive breakpoints_ to adjust the window size. If you're not used to Nativewind or Tailwind, that's OK. We're just going to do a very little bit to get a taste of it.

Nativewind works by letting you use a standard set of descriptive CSS classes to style components. You put these in the `className` prop instead of `style`. What's neat about it is that your styling code looks completely the same across React Web and React Native.

_Breakpoints_ let you put something in front of one of these classes that says it only applies in certain scenarios. So, `sm:bg-black` means, "only set the background to black when the width is greater than the "small" size breakpoint (>640 device pixels)". Nativewind syntax defaults to "mobile" size. You only use breakpoints when targeting larger screens.

2. Let's update the top-level `View` in **[workId].tsx** so it is centered and only takes up some of the screen when the screen is wider, so we can see our modal effect in action.

```diff
- <View className="flex-1">
+ <View className="sm:my-20 sm:w-3/4 sm:self-center flex-1">
```

🏃**Try it.** Open an artwork and change the screen size. It should show the modal when the screen is wider, but look like a pretty typical mobile full screen view otherwise.

3. However, it doesn't look like a very compelling modal because the background behind it doesn't change. Let's add some shade by wrapping everything in **[workId].tsx** in another view that adds said shade:

```tsx
<View
  className={classNames(
    "bg-white sm:bg-black",
    "sm:bg-opacity-20",
    "flex-1 justify-center",
    Platform.OS === "android" && "pt-safe",
  )}
>
  <!-- everything else -->
</View>
```

Don't forget to `import classNames from "classnames";` and `import { Platform } from "react-native";`

> [!NOTE]  
> Hello, new friend! `classNames` is a utility function, helpful for organizing all these bits of Nativewind!

### How would I do this without Nativewind?

There's some fancy media query hooks for React Native that give you programmatic access to breakpoints, which you can then use inline in `style`. It's also pretty straightforward to roll your own:

```ts
import { useWindowDimensions } from "react-native";

export function useMediaQuery() {
  const { width } = useWindowDimensions();
  return {
    isSm: width >= 640,
    isMd: width >= 768,
    isLg: width >= 1024,
    isXl: width >= 1280,
  };
}
```

## See the solution

[Solution PR](https://github.com/keith-kurak/expo-router-codemash-2025-starter/pull/1)

## Next exercise

[Module 02](02-api-routes-and-auth.md)
