---
title: "Speed Up Angular Dev With Cache"
date: 2021-10-23T20:10:52-04:00
draft: false

tags: ["Angular"]
categories: ["Frontend"]
hiddenFromHomePage: false

featuredImage: ""
featuredImagePreview: ""

toc: true
autoCollapseToc: true
---

Angular is about to release v13!

{{< tweet user="angular" id="1451642769819242498" >}}

One of the new features that excites me the most is the persistent build cache. In v12.1, this feature was introduced as an opt-in by the CLI via an environment variable `NG_PERSISTENT_BUILD_CACHE=1`. Now this environment variable has been disabled. Instead, this caching feature is now enabled by default and can be configured inside `angular.json` under the `cli` option. If you want to learn more about this feature, [here is the RFC discussion](https://github.com/angular/angular-cli/issues/21545).

## How to enable it (in Angular 13+)?

At the time of writing, this configuration is enabled only on V13, which is currently in RC stage. You can try it out by upgrading your project with `ng update --next`.

After upgrading to 13.0.0-rc1, open your project's `angular.json`, which should normally look like this:

```json
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "version": 1,
  "newProjectRoot": "projects",
  "projects": {
    // ...
  },
  "defaultProject": "frontend"
}
```

The new `cli` option can be configured like this:
{{< highlight json "linenos=table,hl_lines=9-15" >}}
{
"$schema": "./node_modules/@angular/cli/lib/config/schema.json",
"version": 1,
"newProjectRoot": "projects",
"projects": {
// ...
},
"defaultProject": "frontend",
"cli": {
"defaultCollection": "@angular-eslint/schematics",
"cache": {
"enabled": true,
"environment": "all",
"path": ".angular/cache"
}
}
}
{{< / highlight >}}

There are three options under `cli`, `enabled` which is a boolean, `environment` which is one of `all`, `ci` and `local`, and `path` which is a string that points to your cache on disk. If your
CI can reuse the same file directory between runs and disk space is not a problem, definitely configure the `environment` option to `all`. Some users have reported massive reduction in build time (details in [this RFC](https://github.com/angular/angular-cli/issues/21545)). The default location of cache is `.angular/cache`. However, I am not sure about the default for `environment` as it's not configured in the `schema.json` file.

## Result

My personal project is a pretty simple one but you can still get a good sense of the dramatic improvement. Just a FYI, the project uses TailwindCSS in JIT mode.

```sh
> tokei src/app --exclude "*.{module,spec}.ts"
===============================================================================
 Language            Files        Lines         Code     Comments       Blanks
===============================================================================
 HTML                   20         2009         1831          119           59
 Sass                   20           59           54            0            5
 TypeScript             55         1985         1783           12          190
===============================================================================
 Total                  95         4053         3668          131          254
===============================================================================
```

### `ng build`

{{< image src="ng-build.svg" caption="`ng build`" width="60%" >}}

### `TAILWIND_MODE=watch ng serve --hmr`

{{< image src="ng-serve.svg" caption="`ng serve`" width="60%" >}}

Amazing! :tada: For `ng build`, I am able to achieve a whopping 66% build time reduction for the second run, with the cost of the first run being slightly slower. For `ng serve`, it's a similar story where I am able to cut the time by over 50% starting from the second run.

## Conclusion

This a cool productivity boost and at the same time a huge CI cost saver. As mentioned, you can enjoy this feature in Angular 12.1+ already, with `NG_PERSISTENT_BUILD_CACHE=1` command line argument. However, in Angular 13+ you should configure it in your `angular.json`. If you are not using it already, you definitely should. NOW! :bicyclist::sweat_drops::sweat_drops:

As always, thanks for reading!
