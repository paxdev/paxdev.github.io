---
title: Create an Ionic mobile app with Angular and Cordova
categories:
  - mobile
tags:
  - ios
  - ionic
  - cordova
  - angular
  - mobile
---

## Create an Angular 12 App using Ionic 6 and Cordova 10

### NB
You can develop the app and debug in a browser on any OS capable of running Node/NPM however you will need OS X to build, iOS resources and use the Xcode iOS Simulator or deploy to an iOS device locally for debugging.

1. Install globall `NPM` CLI tools

    ```shell
    npm install -g @angular/cli@12.1.4
    npm install -g @ionic/cli@6.18.0
    npm install -g cordova@10.0.0
    ```

1. Create the app

    ```shell
    ionic start my-app blank --cordova --type=angular
    ```

    * This tells the `Ionic CLI` to create an app called `my-app` usin the `blank` template using `Cordova` as the Native Runtime and to use the `Angular` framework.

    * There are a number of pre-built app templates that you can use with `Ionic`, but here I am using the `blank` template as I can build up the app how I like.
    
    * You will be prompted to confirm that you want to use `Cordova` instead of `Capacitor` - enter `y` to continue.

        > `Capacitor` is the `Ionic` developed framework for hosting a `Webview` based app on a mobile device.
        > The chief difference is that `Cordova` uses the command line to build native projects as a _build_ asset whereas `Capacitor` treats them as a _source_ asset and expects you to use native tooling, e.g. `Xcode` and `Android Studio` 
    
    * You will also be prompted to create an `Ionic` account. There is absolutely no need to do so!

1. You can now `cd` into the app directory and start serving your app.

    ```shell
    cd my-app
    ionic serve
    ```

    * This starts a development server with Live Reload (typically on port `localhost:8100' unless that port is already in use).
    * When making Web Service calls remember that you may need to configure your allowed origins to handle this when debugging locally.

You can use the Ionic CLI to scaffold new Components, Modules etc. (`ionic g ...`), stick with Angular CLI (`angular g ...`) or use both.

At it's heart `Ionic` consists of a set of `Components` that are optimised for display on Mobile devices and deal with things like camera cutouts etc. ([Components documentation](https://ionicframework.com/docs/components)).
    <br/>You'll probably find that it's easier to use the Ionic CLI for Ionic components.

The next step is to add a Platform such as iOS or Android so that you can compile for those devices. I will cover creating and deploying an iOS app in the next posts.