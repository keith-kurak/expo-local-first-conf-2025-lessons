# Module 03: All about deployment

### Goal

Let's learn how to deploy mobile apps with Expo. This part is a little tricky for everyone to do live, because you may need an Apple Developer account, but we can walk through these and you can try them out later if you'd like.

### Concepts

- How Continuous Native Generation to treat native project files as ephemeral resources akin to **node_modules**, created just in time for building.
- Different native builds for production and development, and how to set these up with a dynamic app configuration and environment variables
- How to build and deploy the server solution with Cloudflare Durable Objects
- How to build and deploy all of the frontends with EAS

### Tasks / Demonstration

- Deploy the durable object backend to Cloudflare
- Plan out a comprehensive development / staging / production environment setup for both web and mobile, using a dynamic app config to generate apps for distinct envronments.
- Build those environment versions with EAS Build
- Deploy the web frontend to EAS Hosting

### Useful links
- [Getting started with Cloudflare Durable Objects](https://developers.cloudflare.com/durable-objects/get-started/)

# Exercises

## Exercise 1: Deploy the Cloudflare Durable Object Worker

A Durable Object combines compute with storage into a single Cloudflare worker. At the beginning, our app was communicating with an already deployed worker, and then with a worker running locally. Now, we are going to deploy our own worker to our own Cloudflare account.

1. If you haven't alredy, sign up for a Cloudflare account. SQLite-backed durable objects are [available on the free tier](https://developers.cloudflare.com/durable-objects/platform/pricing/).

2. Login to your Cloudflare account on the CLI by running `npx wrangler login`.

3. Deploy the durable object worker:

```bash
cd packages/sync-server
npx wrangler deploy
```

🏃**Try it.** You should be able to point your mobile frontend to the URL for your worker and start syncing against it. But... you may want to hang on for a second, because we're going to work on setting up some environments for this.

## Exercise 2(a): Introduction to development builds

Until now, you've been testing your mobile app in Expo Go. This is a great way to try out Expo, but long term, it's not how you want to develop your app. Each React Native app has what we call a "native runtime." The native runtime is the native packages and configuration that define what JavaScript is compatible with the app. The native runtime is compiled into the app and can't change after being built, but JavaScript is embedded (or downloaded), and you can swap out different JS bundles onto the same runtime.

This is how Expo Go works. It's a fixed runtime that happens to be compatible with a good bit of React Native JavaScript. But, there's a lot of packages that aren't in Expo Go that you might want to use (In app purchase is a good example). However, being able to run your code from a development server in Expo Go is pretty convenient. How do we get the best of both worlds?

The answer is a **development build**. This is just a separate build of your app that includes the same app launcher interface as Expo Go, and can thus run any bundle that is compatible with its runtime. Except now, the runtime is completely up to you. You can add native code to your app, rebuild the development build, and continue working just like you were using Expo Go. 

To get a development build, we just need to install the `expo-dev-client` package and then create a debug build of our app.

## Exercise 2(b): App variants in Expo

Now that we've got a development and production environment, let's update our mobile app to dynamically switch between different environments. For good measure, let's add a third, `preview`, in case we add a non-prod database server for testing.

Currently, all the configuration that defines how your Expo app is built into a native binary is enclosed in your **app.json** file. This includes the short name under the app icon, the bundle identifier, the app icon itself, etc. As long as those never change, you could only ever build one environment at a time. If you build a production app and then a development app, the latter would overwrite the former.

However, we can make an **app.config.js** file, which makes that configuration dynamic. Then, by using environment variables, we can change the app names and bundle identifiers based on whether we want a development, preview, or production app. By doing this, we can actually have all three app variants installed on the same phone. We can be developing on our phone one minute, and be testing production in the next.

1. Add the dynamic config by creating the **app.config.js** file inside **packages/mobile**. Add the following code there:

```js

```

This will cause the default values to come from **app.js**, and we only have to customize what is unique about each one.

2. Add these customization points based on `APP_VARIANT`:

```ts

```

3. Update the **package.json** `dev` script to use `APP_VARIANT=development`:

```json

```

🏃**Try it.** You can run `pnpm run dev` and now it'll use the `development` variant values when running the app. However, we don't yet have an app to run it in. Let's fix that.

## Exercise 3: Building with EAS Build

Let's build and deploy our `development` and `production` builds with EAS Build, setting up environments along the way.

### Initialize EAS Build

TBD

### Setup environments

TBD

### About deploying to phones (Apple, specifically)

TBD

### Let's build!

TBD

## Exercise 4: Deploying the web frontend with EAS Hosting

TBD

Expo apps can also target web. Currently, Livestore for web doesn't yet support Expo due to some differences in module compatibility with Metro bundler as compared to Vite, but even today, you can deploy the web app to EAS Hosting.

1. ...

## See the solution

[Solution PR](https://github.com/keith-kurak/expo-router-codemash-2025-starter/pull/1)
