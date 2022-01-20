---
title: Deploy iOS/Ionic/Cordova apps to the Simulator or a Device
categories:
  - mobile
tags:
  - ios
  - ionic
  - cordova
  - angular
  - mobile
---

## How to deploy an Ionic/Cordova app to the iOS Simulator or an iOS device.

### Pre-requisites
* You will, of course need an Ionic/Cordova app!
* This can only be performed on a device running OS X. There are cloud solutions but this post deals only with running on a physical device
* Xcode + Xcode Command Line Utilities installed (these steps are valid for 12.x and 13.x)
* An [Apple Developer Account](https://developer.apple.com/account/) 

### 1. Install tooling
Assuming that you already have `Angular`, `Ionic` and `Cordova` CLI tools installed globally:

1. To generate resources

    ```shell
    npm install -g cordova-res
    ```

2. To run the app in Simulator/Device

    ```shell
    npm install -g native-run
    ```

### 2. Generate the Xcode project

1. Clean up the workspace (assuming you are using `git`)

    ```shell
    git clean -xfd
    ```

1. Reinstall `npm` packages

    ```shell
    npm install
    ```

1. Generate `Cordova` resources (splash screen and icons)
    <br/>_Although unintuitive this must be done_ before _adding `iOS` as a platform_

    ```shell
    ionic cordova resources ios
    ```

1. Add `iOS` as a platform for `Cordova`
    <br/>_This will restore `Cordova` plugins and build the `Xcode` project_

    ```shell
    ionic cordova platform add ios
    ```

### 3. Run the app in the Simulator

1. To run the app in the simulator

    ```shell
    ionic cordova emulate ios [-l]
    ```

    _The optional flag [-l] enables live reloading (i.e. as changes are saved the app is updated)_
    
### 4. Access Browser Developer Tools for the App

* With the `Simulator` running launch `Safari`
* If you don't have the `Develop` menu, enable it under `Safari/Preferences/Advanced`
* From the `Develop` menu, select `Simulator { simulated device and OS }/{App name}/localhost - {relative URL}`
* You can now access the `Developer Tools` as though you were running the app in a browser

### 5. To run the Simulator for a specific Device/OS

* To get a list of devices

```
native-run ios --list
```

* You will see something like the following. Make a note of the Target ID you want to run:

    | Name | API | Target ID |
    |------|-----|-----------|
    | iPad (5th generation) | iOS 12.2 | 11904882-5E5A-4941-B291-5D38F3B51E98 |
    | iPad (6th generation) | iOS 12.2 | 82079A1A-87C1-47EC-BEC2-69BA97FC440C |
    | iPad Air | iOS 12.2 | 7E90921D-D166-4CE1-956F-0193EF6E972F |

* You can now run, e.g.

```
ionic cordova emulate ios -l --target="7E90921D-D166-4CE1-956F-0193EF6E972F"
```

### 6. Pair a device with Xcode for local debugging

1. Connect your device to the Mac with a Lightning Cable. You may need to click `Trust` on the device to trust the Mac

1. If the device repeatedly disconnects/reconnects then run the following from a terminal window (adding your credentials when prompted)

    ```shell
    sudo killall -STOP -c usbd
    ```

1. Launch `Xcode` and open `Windows/Devices and Simulators`

1. Click `Devices`, then select your device. Ensure that `Show as run destination` and `Connect via network` are both selected.

    ![Select Device](/assets/posts/iphone/SelectDevice.jpg)

1.  You can now disconnect the Lightning Cable
    <br/>_N.B. you don't necessarily need to use 'Connect via network' if you are happy to always connect using a lightning cable._

### 7. Deploy/Debug with a device locally

#### Preparation

1. Install `ios-deploy`

    ```shell
    npm install -g ios-deploy
    ```

1. Launch 'Xcode' and open `/platforms/ios/your.app.xcodeproj`

1. Double-click on the project file at the top of the Project Explorer window.

1. Select tge scheme for the device you want (near to the "Build & Run" icon)

1. Ensure that the desired version of iOS is selected

1. Click on the Signing and Capabilities tab

1. Check automatically manage signing. Ensure that a team with a developer account is selected

#### Run on an external device with live reload

1. Run

    ```shell
    ionic cordova run ios -l --external
    ```

1. If you have more than one network interface (e.g. WiFi, ethernet, VPN) you will be prompted to select one. Ensure you select the WiFi interface.
    This will be used to host the live reload server.

#### Run on an external device _without_ live reload

1. Run

    ```shell
    ionic cordova run --ios device
    ```

1. This allows you to use the app without needing the live reload server running on your Mac.

### 5. Access Browser Developer Tools remotely

1. Follow the [steps for the Simulator](#4.-Access-Browser-Developer-Tools-for-the-App) but select your device instead of a simulator device.
