---
slug: running-a-full-stack-daml-app
id: 8r6xwafctrqz
type: challenge
title: Running a full-stack Daml App
teaser: Learn how to run and pass tests
notes:
- type: text
  contents: |
    This is the first in a series of four tutorials to get you up and running with full-stack Daml development. We do this through the example of a simple social networking application. The three tutorials cover:

    1. How to build and run the application (this tutorial)
    1. How to write a [new feature for the app](https://digitalasset.com/developers/interactive-tutorials/getting-started/your-first-feature/)
    1. How to deploy your app to [Daml Hub](https://www.digitalasset.com/developers/interactive-tutorials/getting-started/deploy-to-dabl/)

    You can also read more about the [architecture of this application here](https://docs.daml.com/getting-started/app-architecture.html)

    The final application will look something like this:

    ![Final App](/daml/courses/getting-started/build-and-run/assets/gsg_better.gif)

    By the end of this tutorial, you’ll have a good idea of the following:

    1. What Daml contracts and ledgers are.
    1. How a user interface (UI) interacts with a Daml ledger.
    1. How Daml helps you build a real-life application fast.
    1. How you can deploy your app to a locally hosted environment.

    With that, let’s get started!
tabs:
- title: Sandox
  type: terminal
  hostname: vsc-ide
- title: Terminal - UI
  type: terminal
  hostname: vsc-ide
- title: VSC IDE
  type: service
  hostname: vsc-ide
  port: 8080
- title: Daml UI
  type: service
  hostname: vsc-ide
  path: /damlui
  port: 3000
difficulty: basic
timelimit: 2000000
---
Creating the App
================

We’ll start by getting the app up and running, and then explain the different components.

First off, open a terminal and instantiate the template project. You can click on the below snippets to run them in the terminal.

```
daml new create-daml-app --template create-daml-app
```

This creates a new folder with contents from our template. To see a list of all available templates run `daml new --list`. Change to the new folder:

```
cd create-daml-app
```

Any commands starting with `daml` are using the [Daml Assistant](https://docs.daml.com/tools/assistant.html), a command line tool in the Daml SDK for building and running Daml apps.

Running the Daml Ledger
=======================

We can run the app in two steps.

The first is to start the Daml ledger in terminal.

```
daml start
```

You will know that the command has started successfully when you see the output:

`Press 'r' to re-build and upload the package to the sandbox.
Press 'Ctrl-C' to quit.`

The `daml start` command does a few things:

1. Compiles the Daml code to a DAR file as in the previous `daml build` step.
1. Starts an instance of the [Sandbox](https://docs.daml.com/tools/sandbox.html), an in-memory ledger useful for development, loaded with our DAR.
1. Starts a server for the [HTTP JSON API](https://docs.daml.com/json-api/index.html), a simple way for the UI to communicate with our Daml ledger.

We’ll leave this `daml start` running so it can serve requests to our UI.

Build the UI
============

Now, in the second terminal we will go to the `ui/` folder of the `create-daml-app` and use `npm` to install the dependencies for the UI:

```
cd create-daml-app/ui
npm install
```

This step may take a couple of moments. You'll be back to the terminal prompt `$` once the process is done.


Start the UI
============

We can now start the UI by running

```
npm start
```

This starts the web UI which will connect to our JSON API server.
Once the web UI has been compiled and started, you should see `Compiled successfully!` in your terminal.

Now [open the UI tab](https://[[HOST_SUBDOMAIN]]-3000-[[KATACODA_HOST]].environments.katacoda.com) to see your running application and continue to the next step to see what you can do with it.

Play with the App
=================

[On the the UI tab](https://[HOSTNAME]-[PORT]-[PARTICIPANT_ID].env.play.instruqt.com) you should now see a login page. For simplicity of this app, there is no password or sign-up required.

Log in as `alice`. Other valid usernames are `bob` and `charlie`.

> Note: Usernames are all lower-case, although they are displayed in the social network UI by their alias instead of their user id, with the usual capitalization). Usernames are case sensitive.

You should see the main screen with two panels. The top panel displays the social network users you are following; the bottom displays the aliases of the users who follow you. Initially these are both empty as you are not following anyone and you don’t have any followers. Log out fom `alice` and log in as `bob`. Then you will be able to start following `alice` by either 1) typing her name into the text box or 2) selecting it from the drop-down list and clicking the *Follow* button in the top panel.

The user you just started following (`alice` in this case) appears in the *Following* panel. However, she does not yet appear in the *Network* panel. This is because `alice` has not yet started following `bob`. This social network is similar to Twitter and Instagram, where by following someone you make yourself visible to them but not vice versa.

To make this relationship reciprocal, log out from `alice` and log in as `bob`.

Now have `bob` follow `alice` by clicking the + sign next to `alice`'s name.

Once `bob` starts following `alice` both of them can now see each other in their Network. Try logging back in as `alice` and you'll see `bob` is now in her network. Then log back in as `bob` and you'll see the same for him. You can repeat the process for `charlie` to get a better understanding when a user becomes visible to others - this will be important to understanding Daml’s privacy model later.

> Note: If you want to see the code within this `create-daml-app` project click on the IDE tab or [check it out on GitHub](https://github.com/digital-asset/daml/tree/master/templates/create-daml-app).

About the App
=============

This social network is similar to LinkedIn or a private Instagram, where by following someone (ie. "Bob"), you make your profile visible to him but not vice versa. Bob still needs to choose on whether or not you're allowed to see his profile.

To make this relationship reciprocal, logout from "Alice" and login as "Bob".

Now have Bob follow Alice by clicking the + sign next to Alice's name.

Once Bob starts following Alice both of them can now see each other in their Network. Try logging back in as Alice and you'll see Bob is now in her network. Then log back in as Bob and you'll see the same for him.

Play around more with the app at your leisure: create new users and start following more users. Observe when a user becomes visible to others - this will be important to understanding Daml’s privacy model later.

> Note: If you want to see the code within this `create-daml-app` project click on the IDE tab or [check it out on GitHub](https://github.com/digital-asset/daml/tree/master/templates/create-daml-app).

