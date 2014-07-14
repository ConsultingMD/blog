---
layout: post
title: "Setting Up Your Mobile Development Environment"
date: 2014-07-11 14:56:44 -0700
comments: true
categories:
---

If you are developing mobile applications using cordova or phonegap, the easiest way to test
your applications is setting up the mobile emulators in your machine.

We'll learn how to setup and optimize the Android emulator on an Intel machine,
and how to debug the applications on iOS and Android.

## Setting up and optimizing our mobile devices emulators

### iOS
  Assuming you have a Mac and Xcode installed you already have the iOS emulator and most likely
  it runs pretty fast, so no need for tune up there.

### Android

First we need to install the Android SDK and add some of it's directories to our `$PATH`

  1. [Download](http://developer.android.com/sdk/index.html) and extract the SDK
  2.  Add the `sdk/tools` and `sdk/platform-tools` directories to your `$PATH`

#### Optimizing the Android emulator.

  You can skip this section if you do not have an Intel machine.

  If you do have an Intel machine, we're going to speed up our emulator with
  *Intel's HAXM* (Hardware Accelerated Execution Manager).

  The Intel Hardware Accelerated Execution Manager is a hardware-assisted virtualization
  engine (hypervisor) that uses Intel Virtualization Technology to speed up
  Android app emulation on the host machine. In combination with Android x86 emulator
  images provided by Intel and the official Android SDK Manager, HAXM allows for
  faster Android emulation on Intel VT enabled systems.

  1. [Download](https://software.intel.com/en-us/android/articles/intel-hardware-accelerated-execution-manager) and Install Intel Hardware Accelerated Execution Manager

  2. Download an Android* x86 system image through the Android* SDK manager.

  ![Window > Android SDK Manager][android_sdk]

  ![Intel CPU][x86_cpu]

  3. Go to the *Android Virtual Device Manager* create or edit a device and set the CPU/ABI to Intel Atom (x86)

  ![VDM][andorid_vdm]

  Creating

  ![Create][new_vdm]

  Edit

  ![Edit][edit_vdm]

  Now that we installed Intel HAXM and set up our Android virtual device with an Intel processor, it will use
  Intel HAXM to accelerate the speed of our emulator to a level where it is at least usable.


## Debugging our Cordova Applications
  In this section we are going to use Safari and Chrome's remote debugging tools respectively for
  iOS and Android remote debugging.

### iOS
  For you to access an app running locally from the iOS simulator is very easy,
  localhost on the simulator points to the localhost of your machine, so accessing
  your app will no be a proble.

  Debugging an application running in the iOS emulator is fairly simple with Safari.

  You can skip to step 4 if you can see the `Develop` menu in the menu bar.
    1. Go preferences
    2. Advanced
    3. Show Develop menu in menu bar.

  Once we've done that, we can run our application then, in Safari

    4. Develop
    5. iPhone or iPad Simulator, depending on the hardware we are emulating.
    6. The name of our App, most likely `index.html`, since we are running our HTML5 app.

  As you can see you'll only need to repeat the last 3 steps next time you start
  your app on the simulator. An easy way to do this is to set up and applescript
  to do it for you.

  To set up an AppleScript:

  1. Open the AppleScript Editor.
  2. Copy the next script and paste it in the editor:


        tell application "Safari"
           activate
           delay 2
           tell application "System Events"
             tell process "Safari"
               set frontmost to true
               click menu item 2 of menu 1 of menu item "iPhone Simulator" of menu 1 of menu bar item "Develop" of menu bar 1
             end tell
           end tell
         end tell


    This AppleScripts assumes that you are running only one simulator and the hardware
    you are emulating is an iPhone, if you decide to emulate an iPad change `iPhone Simulator`
    to `iPad Simulator` on the script.

  3. Export the script as an application.


  If you are running OSX Mavericks, since we are targeting “System Events” Accessibility API,
  the OS will display a warning and you will need to go to the Accessibility settings under
  the Privacy tab of the Security & Privacy preferences.

  Now we have everything in place to start debugging our application like a boss.
  You can keep that AppleScript on your dock and you'll be able to click it whenever
  you run your simulator and need to debug it.

### Android
  Unlike the iOS simulator, localhost in the Android emulator does not point to
  localhost in the host machine. In order to access it you will have to go to
  `10.0.2.2` in the emulator and it will route to your localhost.

  For Android we will use Chrome and Chrome DevTools

  While the emulator is running open Chrome

  1. Chrome menu
  2. Tools
  3. Inspect Devices

  Another option is to paste this on a new tab "chrome://inspect/#devices"

  Here you will see all the WebViews running in your emulator including your app and any tab running in the browser.

  Again if you want to make this step easier make an AppleScript with this code
  and keep it in your dock.

    tell application "Google Chrome"
       activate
       open location "chrome://inspect/#devices"
       delay 1
       activate
    end tell


If you are debugging and you close the App in the emulator, on Android and iOS
this will close the window with the debugger tools. This means you'll have to
open the debugger tools every time you restart your App for some reason. There
the motivation to make this process easier by using AppleScripts to open them.

Most likely you'll want to restart our Apps to make it reload some cool new stuff
you added to the code. If you our app is contained in an iframe there is a way to
do this without closing the App. If you want to reload your app without closing
the debugger tools window just go to the debugger tools window and reload from
there `CMD + R` or `F5` depending on the OS you use.


[android_sdk]: http://eng.grandroundshealth.com/images/mr-post/android-sdk.png "Android SDK"
[x86_cpu]:     http://eng.grandroundshealth.com/images/mr-post/Screen%20Shot%202014-07-11%20at%203.42.09%20PM.png "Intel CPU"
[andorid_vdm]: http://eng.grandroundshealth.com/images/mr-post/Screen%20Shot%202014-07-11%20at%204.01.56%20PM.png "Android Virtual Device"
[new_vdm]:     http://eng.grandroundshealth.com/images/mr-post/Screen%20Shot%202014-07-11%20at%204.03.24%20PM.png "Create new Virtual Device"
[edit_vdm]:    http://eng.grandroundshealth.com/images/mr-post/Screen%20Shot%202014-07-11%20at%204.04.23%20PM.png "Edit Virtual Device"