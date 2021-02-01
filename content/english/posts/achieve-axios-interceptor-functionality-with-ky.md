---
title: "Achieve Axios' Interceptor's Functionality with Ky"
date: 2021-01-30T20:44:41-05:00
draft: false
tags: ["ky", "axios"]
categories: ["Frontend"]
toc: true
comment: true
featuredImage: "modify-ky-request.png"
---

## Background

[axios](https://github.com/axios/axios) is the most popular choice (based on my personal experience) when Vue/React developers need an HTTP client, and that's also the case in a Vue 3 course that I have been following recently. However, I personally prefer the more platform-native (and hopefully more stable) [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) so I decided to go with my preference and use Fetch throughout the project. But if you've written plain Fetch code, you would know that it's kinda verbose. Imagine writing plain `XMLHttpRequest` code without using axios. Luckily, we have [ky](https://github.com/sindresorhus/ky) to our rescue.

`ky`, just like `axios`, is an HTTP client library that we can use in the browser, key difference being it's based on `Fetch` rather than `XMLHttpRequest`. There are also many other HTTP client libraries you can choose from the community and [a comparison](https://github.com/sindresorhus/got#comparison) can be found at [got](https://github.com/sindresorhus/got)'s repo.

## What are we trying to achieve?

Using `ky` should be pretty straightforward if you follow its documentation. However, in the course I follow they have some addition security measures: In order to access a private API endpoint the instructor provides us, we need to:

- include our access code as a query string at the end of URL
  **and**
- append our access code to the payload if we are submitting a form with `FormData`
- include our access code in the payload if we are submitting JSON

Such global options can be configured in one `axios`/`ky` instance, which then gets exported and used all across your application. There is a well-known solution in `axios`: interceptors. Note that there are two types of interceptors, one for request and one for response. We only need to hook into the request interceptor here. The `axios` example would look like this:

```ts
import axios from "axios";

const secretCode = "******";

axios.defaults.baseURL = "http://apis.somecourseurl.com/api/";

axios.interceptors.request.use((config) => {
  config.params = { ...config.params, secretCode };
  if (config.data instanceof FormData) {
    config.data.append("icode", "******");
  } else {
    config.data = { ...config.data, secretCode: "******" };
  }
  return config;
});

export axios
```

## How are we going to do this in `ky`?

**TL;DR;**

```ts
import ky, { Options } from "ky";

const baseUrl = "http://apis.somecourseurl.com/api/";
const secretCode = "this-is-super-secret-code";
const secretCodeParam = `secretCode=${secretCode}`;

const addSecretCodeToPostBody = (request: Request, options: Options) => {
  if (request.method == "POST") {
    if (options.body instanceof FormData) {
      const newBody = options.body;
      newBody.append("secretCode", secretCode);
      return new Request(request, {
        body: newBody,
      });
    } else if (options.json instanceof Object) {
      return new Request(request, {
        body: JSON.stringify({ ...options.json, secretCode }),
      });
    }
  }
};

export const api = ky
  .create({ prefixUrl: baseUrl, searchParams: icodeParam })
  .extend({
    hooks: {
      beforeRequest: [addSecretCodeToPostBody],
    },
  });

export default api;
```

What did we do here? First, `ky` doesn't have the concept of "interceptor". Instead, it provides a set of hooks where we can tap into the lifecycle and make our changes to our requests and responses. More details can be found [here](https://github.com/sindresorhus/ky#hooks). What we need to use is the `beforeRequest` hook.

Inside `beforeRequest`, the hook functions will receive two arguments: `request` of type [Request](https://developer.mozilla.org/en-US/docs/Web/API/Request) and `options`, which is more or less of type `RequestInit` defined in TypeScript's dom lib.

What we can do inside the hook function is described as such in `ky`'s doc:

> The hook can return a Request to replace the outgoing request, or return a Response to completely avoid making an HTTP request.

Therefore, we are doing the following things with these two arguments:

- Determine the HTTP method. Only modify the payload if it's a `POST` request.
- Grab the `POST` body from `options.body`
  - If the body is an instance of `FormData`, i.e. the request is a form submission with `Content-Type` of `multipart/form-data`, append the `secretCode` at the end.
  - If the payload is in `options.json`, we include our `secretCode` insde it.
- Return a new `Request` based off of the original request, with a only the `options.body` modified:
  ```ts
  // basically a copy constructor
  return new Request(request, {body: <Your New Body>,});
  ```

## Some Extra Tricks

With the hooks, we can achieve some cool nifty tricks that would be cumbersome otherwise. One example would be to have a automatic loading effect while the request is pending.

Let's say you have a pair of actions defined in you Vuex store named `showLoading` and `hideLoading` that toggle a global state property `isShowingLoadingScreen` which then toggles the visibility of your global loading screen mask. You can then do something like this:

```ts
// ... same code as before

const showLoading = () => {
  store.dispatch("showLoading");
};
const hideLoading = () => {
  store.dispatch("hideLoading");
};

// modify your ky instance configuration
export const api = ky
  .create({ prefixUrl: baseUrl, searchParams: icodeParam })
  .extend({
    hooks: {
      beforeRequest: [addIcodeToPostBody, /*new->*/ showLoading],
      afterResponse: [/*new->*/ hideLoading],
    },
  });
```

Here we also made use of the `afterResponse` hook.

## Conclusion

There you have it! A nicely configured `ky` instance at your disposal. Check out [ky](https://github.com/sindresorhus/ky)'s doc to make use of its other features and configurations. And a big shout out to Sindre for making such a nice library! Give it a :star: if you enjoy it!

Thanks for reading my post and happy coding!