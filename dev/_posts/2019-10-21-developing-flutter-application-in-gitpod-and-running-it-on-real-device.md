---
title: 'Developing Flutter application in Gitpod and running it on real device'

redirect_from:
  - /2019/10/21/developing-flutter-application-in-gitpod-and-running-it-on-real-device
  - /developing-flutter-application-in-gitpod-and-running-it-on-real-device
---

[HA Client](https://github.com/estevez-dev/ha_client) 0.7.0 was the first HA Client version released from [Gitpod](https://www.gitpod.io/) environment with some help of [Codemagic](https://codemagic.io/). Because clouds rules, you know.

Today I want to show how I configured Gitpod for Flutter development and how I’m running the app from cloud IDE right on my real Android device.

Please note that HA Client development was finished and its gitpod config could be outdated.
{: .notice--warning}

![image](/img/developing-flutter-application-in-gitpod-and-running-it-on-real-device/IMG_20191021_012006-900x620.jpg)

## Gitpod configuration

As you may know Gitpod is a cloud IDE. You can launch it from any GitHub repository by adding `https://gitpod.io#` to repository url right in a browser address bar. You’ll get not only a ready-to-code environment but a fully functional Linux-based virtual machine. It is based on Docker and can be preconfigured for any specific repository or even branch with configuration files in in.

First of all we will need a `.gitpod.dockerfile` created in the root of our repository:

```dockerfile
FROM gitpod/workspace-full:latest

ENV ANDROID_HOME=/workspace/android-sdk \
    FLUTTER_ROOT=/workspace/flutter \
    FLUTTER_HOME=/workspace/flutter

RUN bash -c ". /home/gitpod/.sdkman/bin/sdkman-init.sh \
             && sdk install java 8.0.242.j9-adpt"
```

It will declare some environment variables and install Java 8 with `sdkman` as on the moment of writing this article Android platform tools can’t be built with Java 11.

Next we will need a `.gitpod.yml` file to configure Gitpod IDE:

```yaml
image:
  file: .gitpod.dockerfile

tasks:
- before: |
    export PATH=$FLUTTER_HOME/bin:$ANDROID_HOME/bin:$ANDROID_HOME/platform-tools:$PATH
    mkdir -p /home/gitpod/.android
    touch /home/gitpod/.android/repositories.cfg
  init: |
    echo "Installing Flutter SDK..."
    cd /workspace && wget -qO flutter_sdk.tar.xz https://storage.googleapis.com/flutter_infra/releases/stable/linux/flutter_linux_v1.12.13+hotfix.7-stable.tar.xz && tar -xf flutter_sdk.tar.xz && rm -f flutter_sdk.tar.xz
    echo "Installing Android SDK..."
    mkdir -p /workspace/android-sdk && cd /workspace/android-sdk && wget https://dl.google.com/android/repository/sdk-tools-linux-4333796.zip && unzip sdk-tools-linux-4333796.zip && rm -f sdk-tools-linux-4333796.zip
    /workspace/android-sdk/tools/bin/sdkmanager "platform-tools" "platforms;android-28" "build-tools;28.0.3"
    echo "Init Flutter..."
    cd /workspace/ha_client
    flutter upgrade
    flutter doctor --android-licenses
    flutter pub get
  command: |
    flutter pub upgrade
    echo "Ready to go!"
    flutter doctor
vscode:
  extensions:
    - Dart-Code.dart-code@3.5.0-beta.1:Wg2nTABftVR/Dry4tqeY1w==
    - Dart-Code.flutter@3.5.0:/kOacEWdiDRLyN/idUiM4A==
```

Lets take a closer look on some entries.

`tasks` section will be executed upon your Gitbod environment start. We are setting `PATH` environment variable and creating some directories and files for Android SDK tools to work properly. We make it in `before` subsection of `tasks` to execute those commands on every start because `/workspace` directory of your environment is the only thing that will be saved between environment stops and starts.

That is why we will put Flutter tool and Android SDK into `/workspace` directory to prevent it from downloading on each start. Also we will have full control over SDKs including it’s cache and versions. It is `init` subsection of `tasks`. We are downloading Android SDK tools and Flutter tool, also we are upgrading Flutter and Dart SDK to a latest versions and launching Android licenses accepting process.

Pay attention on `cd /workspace/ha_client` line. You need to replace `ha_client` with your project sources directory name.
{: .notice--warning}

Now we are ready to start our Gitpod environment. You can find additional information in [Gitpod documentation](https://www.gitpod.io/docs/).

## Launching on real device

After configured environment start you should be able to use `adb`. But our IDE is in the cloud, so it can’t access our USB port. Fortunately `adb` can work not only through physical connection but also over the network. All you need is a dedicated IP address, but I think Dynamic DNS will also work.

### Device preparation

Connect you Android device via USB cable to any PC, laptop or even Raspberry Pi where you have `adb` installed.

Run `adb tcpip 5555` to switch adb on your device to work over the network using port 5555. You can check connection with `adb connect 192.168.1.111:5555` where `192.168.1.111` is your Android device IP address.

To switch your device back to USB debug you need to run `adb usb`.

### Network

I didn’t check Dynamic DNS solution because my home router has static dedicated IP address. But I assume you need to forward port on your router in both cases. It is better to open some random external port instead of default `5555`. For example `39393`. So your external `39393` port should be forwarded to your Android device’s IP address and port `5555`.

There is also a simple solution for network tunneling called `ngrok`. Here is a [guide to connect to your Android device from Gitpod being on any network](/adb-over-any-network-without-port-forwarding-even-over-mobile-3g-4g-lte), even mobile.
{: .notice--success}

It is also possible to use VPN if your router supports it. But in this case you will need to set up VPN connection on your Gitpod environment as well.

### Run

Now you should be able to connect to your device from Gitpod environment console with:

```
adb connect 101.101.1.1:39393
```

Replacing `101.101.1.1` with your router IP address. If connection was successful the only thing left is to run your Flutter application from Gitpod console:

```
flutter run
```