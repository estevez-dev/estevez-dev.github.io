---
title: 'How to run Android Studio on a Chromebook with 4GB of RAM'

redirect_from:
    - /2019/09/01/how-to-run-android-studio-on-a-chromebook-with-4gb-of-ram
    - /how-to-run-android-studio-on-a-chromebook-with-4gb-of-ram
---

It is not a secret that Android Studio could almost perfectly run on a PC with 4GB RAM if we are talking about Windows OS or Linux. Yes we already have an official Android Studio build for Chrome OS, but the things are not so simple here.

The main reason why you could have problems running Android Studio on a Chrome OS with 4GB of RAM is that Android Studio is a Linux application, so it would run on Chrome OS inside a Crostini virtual environment. And virtual environment means that you don’t have direct access to all your hardware resources. Your RAM is not an exception. Free RAM for Crostini is restricted and controlled by Chrome OS.

My own experience with Crostini ends up on building a Flutter application. I mean Android Studio runs almost perfectly until you’ll run an app with debugger attached. Sometimes it even starts an app on connected Android device and can handle debug messages for 10 seconds, but most of the time build fails with unexpected disappearance of Gradle daemon or even Android Studio hang or crash.

Actually it is not a surprise because official requirements for running Android Studio on Chrome OS contains “8GB of RAM minimum”.

This issue can be solved in different ways. You can buy new Chromebook with 8GB of RAM. You can write code in Caret and run it with command line tools. Or you can use [Crouton](https://github.com/dnschneid/crouton).

Crouton is a set of scripts and tools to generate [chroot environments](https://en.wikipedia.org/wiki/Chroot). It allows you to run Ubuntu or Debian in parallel with Chrome OS. The main difference with Crostini is that you will have direct access to all your hardware resources. And yes, it will allow you to run Android Studio having just 4GB of RAM.

The simplest way to start is to set up default chroot with Ubuntu 16.04 and xfce desktop environment.

It is required to put your Chromebook in [Developer Mode](https://www.chromium.org/chromium-os/developer-information-for-chrome-os-devices/generic). Doing this will erase all user data.

If your Chromebook is already in Developer Mode, next you need to

1. Download Crouton installer. It is the first link on [Crouton github page](https://github.com/dnschneid/crouton) right in repository description. Put it to your Downloads folder.
2. Open Chrome OS developer shell with Ctrl+Alt+t and execute shell command there:
```
crosh> shell
chronos@localhost / $
```
3. Make Crouton installer a system executable with:
```
sudo install -Dt /usr/local/bin -m 755 ~/Downloads/crouton
```
4. Create your first chroot:
```
sudo crouton -t xfce
```
and wait for installation to finish.

This is almost all. Actually you already can run Linux environment, but lets install some additional component with this command:

```
sudo crouton -u -t extension -n xenial
```

It will add `extension` component to your chroot. You need it to synchronize clipboard between Chrome OS and Ubuntu and some other useful things. You also need to install [crouton integration](https://chrome.google.com/webstore/detail/crouton-integration/gcpneefbbnfalgjniomfjknbcgkbijom) Chrome extension.

This is it. Now you can start Ubuntu with:

```
sudo startxfce4
```

Use <kbd>Ctrl+Alt+Shift+Back</kbd> and <kbd>Ctrl+Alt+Shift+Forward</kbd> to switch between Ubuntu and Chrome OS.

{% include ads_in_post.html %}

## Set up Android Studio

Buy default your Downloads folder of Chrome OS is mounted to ~/Downloads in your chroot, so basically you have the same Downloads folder in Chrome OS and Ubuntu. This will make thing easier.

1. Download Android Studio for Linux (not Chrome OS) from [download options page of developer.android.com](https://developer.android.com/studio#downloads) and save it to your Downloads folder.
2. Switch to Ubuntu with <kbd>Ctrl+Alt+Shift+Forward</kbd>, launch File Manager app and simply extract Android Studio archive from Downloads folder to any location inside your home folder. I’m using `~/bin` folder.
3. After that you will be able to start Android Studio in your Ubuntu by executing
```
~/bin/android-studio/bin/studio.sh
```
in Terminal app.
4. You also can edit your `.bashrc` file in home directory of Ubuntu and add alias for this command. Use File Manager app to open your home directory (It is `/home/[your_username]`), make hidden files visible with <kbd>Ctrl+h</kbd> and double-click `.bashrc` file to open it in default editor. Now just add this line to the end of file:
```
alias 'android-studio'='~/bin/android-studio/bin/studio.sh'
```

From now on you can just execute `android-studio` command in terminal of Ubuntu to start Android Studio.

## Some notes

If you don’t want to loose your data, you need to properly exit Android Studio, then Log Out from Ubuntu and wait for chroot to stop in Chrome OS shell. You need to see something like this:

```
Unmounting /mnt/stateful_partition/crouton/chroots/xenial…
Sending SIGTERM to processes under /mnt/stateful_partition/crouton/chroots/xenial…
chronos@localhost / $
```

Only after that you can send your Chromebook to sleep or turn it off. It is not recommended to go to sleep with launched chroot.

Note that sometimes when free RAM of your system comes to the end, browser tabs of Chrome OS will be unloaded from memory. As Chrome OS shell also runs inside browser tab it can be also unloaded to free some memory. In that case you will see a new crosh session in that tab instead of your chroot environment output. Just use git for your code and push often and your data will be safe.

There is a possibility to run any app from your chroot environment in a [standalone window right in your Chrome OS](https://github.com/dnschneid/crouton/wiki/crouton-in-a-Chromium-OS-window-%28xiwi%29), without even starting xfce. **Just don’t use it for Android Studio**! There is a number of known issues that will be not resolved in nearest future: code hints with <kbd>Ctrl+Space</kbd> and clipboard sync will be broken for example.