---
title: 'Firebase Cloud Functions development environment in Gitpod'

redirect_from:
        - /2020/05/12/firebase-cloud-functions-environment-in-gitpod
        - /firebase-cloud-functions-development-environment-in-gitpod
---

For me currently Gitpod is the best IDE for Chrome OS users. Not only because it is cloud based with web interface, but because it allows you to set up and run full development environment almost for any purpose not depending on my local system and hardware. The second big thing is that you can switch from one environment to another completely different in a minutes. From time to time I’m realizing there is another project I can run in Gitpod with simple configuration.

Today we gonna set up Firebase Cloud Functions project in Gitpod.

Assuming we already have a project with cloud functions. After launching it in Gitpod first of all we need to install firebase tools. Node.js and Node Package Manager are already preinstalled in default Gitpod environment so we just need to `npm install -g firebase-tools` in console of launched Gitpod environment. Then we need to install our dependencies. Assuming `my_project` is a name of our project lets navigate to `functions` folder with `cd /workspace/my_project/functions` and make `npm install`.

After all dependencies will be installed we’ll need to login to our Firebase account. By default running `firebase login` will open your default browser but it is not possible in Gitpod. We will need to initiate login process with

```bash
firebase login:ci --no-localhost
```

This will output a login url to the console:

![image](/img/firebase-cloud-functions-development-environment-in-gitpod/gitpod.io-2020.05.12-00_38_46.png)

and you’ll need to navigate there with your browser by <kbd>Ctrl+Click</kbd>.

After successful login you’ll be provided with authentication code. Just copy it and go back to Gitpod console to paste it and hit Enter. You’ll get the message:

```bash
✔  Success! Use this token to login on a CI server:
1//038okfzGYjJ0fCgYIARAAGAMSNwF-L9IrN6hf6HG8ELcxmrrksTNr3e7kjjbkbybu7GgjU7qQhMKHJjLGE6MSPeqWm_uPGVg
```

You need to use provided token to execute commands. For example:

```bash
firebase deploy --token "1//038okfzGYjJ0fCgYIARAAGAMSNwF-L9IrN6hf6HG8ELcxmrrksTNr3e7kjjbkbybu7GgjU7qQhMKHJjLGE6MSPeqWm_uPGVg"
```

That’s it. We are ready to write functions and deploy it to Firebase.

We can automate the process described here by adding `.gitpod.yml` file to our project:

```yaml
tasks:
  - init: |
      echo "Installing Firebase tools..."
      npm install -g firebase-tools
      cd /workspace/my_project/functions
      echo "Getting dependencies..."
      npm install
    command: |
      cd /workspace/my_project/functions
      echo "Make sure we are using the latest versions of Firebase tools and libraries..."
      npm install firebase-functions@latest firebase-admin@latest --save
      npm install -g firebase-tools
      echo "Logging in..."
      firebase login:ci --no-localhost
```