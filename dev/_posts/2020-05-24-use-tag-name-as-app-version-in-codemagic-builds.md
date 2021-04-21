---
title: 'Use tag name as app version in Codemagic builds'
layout: post
redirect_from:
    - /2020/05/24/use-tag-name-as-app-version-in-codemagic-builds
    - /use-tag-name-as-app-version-in-codemagic-builds
---

There is several ways to create different build flows for your Flutter app. You can use [flavors](https://flutter.dev/docs/deployment/flavors) for example. But today I want to describe my Android release flow where Codemagic builds my app from GitHub tag to publish it to Google Play and to create a release in GitHub. And the most comfortable thing is that app version is taken from tag name.

I have **Build triggers** set to **Trigger on tag creation**.

I also have additional **Build arguments** for Android added in **Build** section:

```
--dart-define=versionName=$(/usr/bin/git describe) --build-name=$(/usr/bin/git describe)
```

`--build-name` parameter allows us to define versionName for our future apk.

`--dart-define` allows to define environment specific variables to use in application code with `String.fromEnvironment` so we’ll be able to show application version with `String.fromEnvironment('versionName')`.

`git describe` command will show most recent tag for current commit.

Also I have **Google Play** and **GitHub** releases enabled in **Publish** section.

To make this work I need to create an annotated tag and push it to remote:

```
git tag -a "1.1.0-alpha" -m ''Release 1.1.0 alpha"
git push origin 1.1.0-alpha
```

That’s it.