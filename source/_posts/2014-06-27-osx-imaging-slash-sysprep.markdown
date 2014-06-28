---
layout: post
title: "OSX Imaging / Sysprep"
date: 2014-06-27 15:28:54 -0700
comments: false
categories: MacFu
---

###Macintosh Imaging / "Sysprep"

As a rapidly growing company we're constantly adding new team members. Keeping up with hardware provisioning can be hard to manage in any environment, but with the security and audit requirements of the healthcare industry we need to both move quickly and make sure we're always "checking the boxes" for security and encryption.

We've dropped the time needed to provision a new mac user workstation down from about a half a day to about 15 minutes. One of the biggest advantages of the mac platform for us is the ability to have a single gold-master machine image that will run on just about any modern mac hardware.

This has proved to be a huge time and money savings over the hurdles we have to jump through in provisioning our windows and linux machines across disparate hardware platforms from various manufacturers. 

Most of the information in this post has been collected over time from various posts about individual steps in the process from across the web. Where I have notes on where I found the information I've included links at the bottom of the post. 

**In short, our process will be:**

1. Create a bootable OSX mavericks USB installer
2. Image our master master machine
3. Create an admin and user account
4. Create a template user
5. Convert the template user a default user profile
6. Create a filevault corporate encryption key 
7. Check our work by creating a new user
8. Bake all of this goodness into a disk image
9. Cast the image to our target machine(s)
10. Profit!

**Tools/Software we'll need:**

1. A mac machine to image (and ideally one or more target machines to cast the image to)
2. 1 or 2 8GB USB thumb drive(s)
3. Clonezilla open source disk imaging tool
4. Network or USB storage for your machine image
5. Thunderbolt of Firewire cable (and/or adapters)

* * *

While it takes a bit of effort to get through these steps and create that perfect master image - the time savings on the first deploy alone will nearly make up the time spent making the master right.


**1) Create a bootable OSX Mavericks install drive** <sup>[1]</sup>

Our first step will be to create a Mavericks installer to allow us to create a perfectly clean factory fresh mac as our base to build from. If you have a shrink-wrapped mac from the factory that you've never booted you can skip down to step 3 below. However it isn't a bad idea to have the Mavericks USB key in your tech tools arsenal. There are

+ Download the Mavericks installer from the Mac App Store.

+ Connect to your Mac a properly formatted 8GB (or larger) USB thumb drive. Rename the drive to Untitled. (The Terminal command used below assumes the drive is named Untitled.)

+ Select and copy the text block below: 

> _NOTE: The location of the installer referenced below is the default download location, change the path to reflect where you have the installer._

```bash

sudo /Applications/Install\ OS\ X\ Mavericks.app/Contents/Resources/createinstallmedia --volume /Volumes/Untitled --applicationpath /Applications/Install\ OS\ X\ Mavericks.app --nointeraction

```

+ Launch the OSX Terminal (in /Applications/Utilities).

+ Paste the copied command into Terminal and press Return. _Warning: This step will erase the destination drive or partition, so make sure it doesn't contain any valuable data._

+ Enter your admin-level account password when prompted.

+ The Terminal window displays the progress of the process, in a very Terminal sort of way, by displaying a textual representation of a progress bar: Erasing Disk: 0%... 10%...20%... and so on. The program then tells you it's copying the installer files, making the disk bootable, and copying boot files. Wait until you see the text Copy Complete. This can take as long as 30 or 40 minutes, depending on how fast your Mac can copy data to your destination drive.


We now have a bootable Mavericks-install drive.  We could stop at this point and just use this USB boot key to put clean installs of the Mavericks OS on whatever Mac machines we wanted.  But the point of this exercise is to not have to start from factory defaults for each of our machine deploys and for each of our user accounts. 



**2) Image our master machine**

Ideally your template machine should somewhat resemble the hardware configuration you'll be deploying to most often. It can make things easier if your template machine has a smaller hard drive than any target machine you intend to deploy to down the road. For example if most of your machines are going to have 256GB drives but you you have a few 128GB machines then choose a 128GB HDD to build your template. This just makes the deployment process a little easier in subsequent steps where it is far easier to grow than shrink a disk partition. If your master machine has a larger disk you can at least shrink the disk partition down so that you can do partition level deploys to your target machines. Just open disk utility and select the physical disk device and drag to shrink the partition to be less than the size of your smallest target machine.

Now you have a clean machine with a fresh install of OSX mavericks

**3) Create an Admin and template-user account**

Boot this machine and create your login account. This should be your "system administrator" account that will be present on all of your machines. We'll call this user "ops" We'll use this account for machine administration, but this will not be the user we're going to setup as a template. One important note - we want to skip the association of an Apple ID with this user. It takes some effort to skip past this, as apple would really like every user to have an apple ID. It becomes problematic to have software associated with an apple ID that your users can't login as. None of the software we typically install on our base image requires an Apple ID.

If you've recently created your mavericks installer you won't have a ton of software updates to perform, but now is the time to start that process. Click the apple menu and choose Software Update… Click Update All 

Depending on how much there is to download and how fast your internet connection is, this process could take a while and may require a reboot or two. Run through this process a couple of times until you're able to start software update and find that there are no available updates.

Next we'll create a user that we'll setup to be the template for all future users we create on the machine.

**4) Setting up your default user profile** <sup>[2]</sup>

+ Create an administrative user called testing

+ Apple > System Preferences > Users & Groups

+ Click the lock to enable changes

+ Click the + icon to add a user account

+ ![User Accounts > Add User][testing_user]


Log out of your ops account and log in as the testing user.

We'll do all of the software installations and configuration setup as this user.

Now we'll install and run each of our default applications. Set everything for this user exactly as you'd like it to be for each new user you create on the machine. This includes all desktop, dock, system settings, backgrounds, screensavers etc. You'll want to touch everything possible as this user to ensure that you capture a setting that will be saved as part of the user profile. 

Take your time during this process. Anything that isn't set the way you'd like, or anything you overlook will be something you'll have to fix every single time you deploy a workstation. While it is possible to go back and fix your machine image to correct issues, it is far easier to get it right from the start.

Here's a list of the general software suite we want/need across all of our mac machines. Wherever possible I've tried to provide a link to the full/offline installer when there's a choice.


**Suggested software to install:**

- Firefox - <http://www.mozilla.org/en-US/firefox/new/>
- Chrome - <https://www.google.com/intl/en/chrome/browser/#eula>
- GPG Tools - <https://gpgtools.org/>
- Google hangouts plugin - <https://www.google.com/tools/dlpage/hangoutplugin>
- Adobe Flash - <http://www.adobe.com/products/flashplayer/distribution3.html>
- Java 7 JRE - <http://www.oracle.com/technetwork/java/javase/downloads/jre7-downloads-1880261.html>
- Skype - <http://www.skype.com/go/getskype-full>
- OpenOffice - <http://www.openoffice.org/download/>

- VPN / Anti-virus software of your choice / need:
	  - Tunnelblick VPN (NOTE: this is currently beta for mavericks)- <https://code.google.com/p/tunnelblick/wiki/DownloadsEntry?tm=2>
	  - Viscosity VPN- <https://www.sparklabs.com/viscosity/>
	  - Sophos Antivirus - <http://www.sophos.com/en-us/products/free-tools/sophos-antivirus-for-mac-home-edition.aspx>
	  - Forticlient AV/VPN - <http://forticlient.com/>

- Password manager:
	  - Roboform - <http://www.roboform.com/download>
	  - 1Password - <https://agilebits.com/onepassword/mac>
	  - LastPass - <https://lastpass.com/misc\_download2.php>


After installing an application make sure that you run the app as the user to dismiss the warning dialogs and any initial setup screens. If you want to add custom bookmarks or a default webpage to the user's browser this is the time to do so.

Other Settings:

- Give the user a quick way to lock their mac workstation using hot corners
  - System Preferences > Desktop & Screen Saver > Hot Corners
  - Set the upper left corner action as "Start Screen Saver"
  - ![Hot Corners][hot_corners]



  - Optionally set the other corners to your preference.
  - Now set the workstation to lock when the screensaver starts:
	  - System Preferences > Security & Privacy > General
	  - Set the option to require a password 5 seconds after sleep or screensaver
	  - ![Screen Saver Password][screen_pass]


- Bring Back Drive icons on the desktop - At some point apple decided that nothing should appear on the desktop ever.
  - With Finder active presss CMD - , or choose Finder > Preferences
  - Check All the Boxes
  - ![Finder Preferences][finder_pref]

- Configure the Dock - Make sure the dock icons are laid out the way you'd like them to be and that default applications are present. 
  - To keep an application icon in the dock - run the application and right click on the dock icon and choose "Keep in Dock"
  - ![Keep In Dock][keep_dock]

**5) Convert the template user a default user profile**

Now that everything is 100% setup with your testing user. And you're totally sure? Go ahead and log back in and fix that one last thing you forgot, we'll wait. Now, log out of the testing account and log back into your ops account. We need to clean a few things out of our testing user's account. We can dump their cached data. And we need to delete their login keychain to prevent conflicts when deploying users. OSX will automatically create new login keychains during user creation.

> _NOTE: You may want to make a backup copy of the English.lproj directory rather than just blowing it away. If you've messed up your template user, then all future users created on this machine will be similarly messed up. If you keep a copy of the English.lproj you can always put it back and start the user template process over again._

Open the OSX terminal application and run the following commands:

```bash
user$ sudo -i
root# rm -rf /Users/testing/Library/Caches/\*
root# rm -rf /Users/testing/Library/Keychains/\*
root# rm -rf /System/Library/User\ Template/English.lproj/
root# mkdir /System/Library/User\ Template/English.lproj/
root# cp -R /Users/testing/\* /System/Library/User\ Template/English.lproj/
```

**6) Create a filevault corporate key** <sup>[3]</sup>

Now that we have all of our applications installed and configured we want to set a corporate FileVault encryption password/key. IMHO this is important even if you're not implementing encryption across your organization. This step can help prevent a user from encrypting their drive with a password that is unknown to the IT department. Apple actually provides good documentation on doing this step. I've paraphrased their information below and included a link to their source knowledge base article at the end of this post.

- Create a master password
  - Open System Preferences and select the Users & Groups preference pane. If locked, click the lock icon to authenticate.
  - Click the Services button and then select "Set Master Password…" from the pop-up menu.
  - Create a master password using the sheet that appears. You can use the Password Assistant to help you create a strong password. Once set, the following files are created: 
    - /Library/Keychains/FileVaultMaster.cer
    - /Library/Keychains/FileVaultMaster.keychain

  - Copy the /Library/Keychains/FileVaultMaster.keychain file to a safe location for storage, such as an external drive or an encrypted disk image on another physical disk. This file contains the private key required to unlock the encrypted disc.
  - After backing up the FileVaultMaster.cer file to an external location you can safely delete the /Library/Keychains/FileVaultMaster.cer file.

- Delete the Private Key
  - Double-click /Library/Keychains/FileVaultMaster.keychain in the Finder to open the keychain with Keychain Access.
  - In Keychain Access, select FileVaultMaster from the list of keychains on the left.
  - Delete the "FileVault Master Password Key" by highlighting it and then pressing the Delete key on your keyboard. Click Delete in the resulting dialog.
  - Quit Keychain Access.

**7) Check our work**

Here's the moment of truth. Everything is setup and configured. You've checked and double checked all of the user settings. It is now time to create a new user and make sure that everything works correctly.

- System Preferences > Users & Groups > Add a user
  - Make sure the user name and password are different from your testing template user account - we want to make sure that we don't get any false positives in our testing because we took the "happy path"

- Log in as your new user 
  - Voila? - Does everything look as you'd expect.
  - Are all the dock icons where you want them?
  - Are all of you standard applications working correctly?
  - Do you see any error messages?

If you were meticulous and somewhat OCD in your setup of the user template then your new user's desktop environment should look exactly like your testing user and all of the applications should be setup and working as expected. If anything isn't the way you want it you can still fix things. Log out and delete this user. Log back into your testing user account and correct any issues. Go back to step 5 and re-copy the testing user into English.lproj

Keep iterating over this until your new user deployment is perfect. Our next step will be to clean up the machine to keep our disk image small. It won't be impossible to make user profile changes but it will become harder.

**8) Bake all of this goodness into a disk image**

Everything is all set and ready with our machine. We'll do a few cleanup steps to remove extra files.

- Remove all user accounts with the exception of our ops user, make sure you select the option to remove the user's home folder.
- Shut down the template machine
- Re-start the machine in target disk mode by holding down CMD + T as soon as you hear the mac chime at boot.
- Connect the template machine to another mac via thunderbolt or firewire
  - You should be able to see and mount the template machine's hard drive on your mac

- Open the OSX terminal application and run the commands below to remove system specific and swap files from your machine image.
  - _NOTE: Replace imageName below with the actual name of the mounted drive. Also this step seems to be largely optional with the current generation mac platform. We have not run into issues deploying the machine image across modern retina and macbook-airs of varying configurations with these files either present in the image or absent._

```bash
user $ sudo -i
root # rm /Volumes/imageName/var/db/BootCache.playlist
root # rm /Volumes/imageName/var/db/volinfo.database
root # rm /Volumes/imageName/System/Library/Extensions.kextcache
root # rm /Volumes/imageName/System/Library/Extensions.mkext
root # rm -r /Volumes/imageName/var/vm/swap\*
```

- Disconnect the two machines and boot your master machine using a Clonezilla (<http://clonezilla.org/>) USB key
- Follow the prompts in Clonezilla to "save disk" using device-image mode. 
- When prompted you'll mount or attach your bulk storage or network storage device to store the machine image.
- Go get a cup of coffee while Clonezilla does its thing and captures the full disk to an image

**9) Cast the image to our target machine(s)**

Now that we have a Clonezilla disk image created and stored we can use clonezilla to boot and restore the disk image back to our target machines.

- Boot the target machine using the Clonezilla USB key
- Choose "restore disk" using device-image mode
- Plug in or mount your storage volume with the saved machine image.
- After casting the image boot remove clonezilla usb and any other external media and boot the mac
- Login as your "ops" user
- Open disk utility and expand the disk partition to fill the drive
- Enable FileVault encryption 
	- System Preferences > Security & Privacy > FileVault > Turn On Filevault
	- Restart the machine
- Create the user's account
	- System Preferences > Users & Groups > Add User
- Change Sharing settings to name machine after user
	- System Preferences > Sharing > Computer Name
- Login as user
	- Double check and make sure everything is working as you'd expect
	- Perform any additional software package installs that this particular user might require

**10) Profit**

Congratulations! You've successfully cast your gold-master image out to your first machine. All that hard work and patience have paid off in a massive time savings. Not only does it make provisioning a new machine go much faster, you're now guaranteed that every single user account you create on these machines get setup exactly the same way and all of your users have a consistent introductory experience.

* * * 

References: 

1. <http://www.macworld.com/article/2056561/how-to-make-a-bootable-mavericks-install-drive.html>
1. <https://sites.sas.upenn.edu/jasonrw/blog/2013/03/create-custom-default-user-profile-os-x-107108>
1. <http://support.apple.com/kb/ht5077>

[1]: http://www.macworld.com/article/2056561/how-to-make-a-bootable-mavericks-install-drive.html 

[2]: https://sites.sas.upenn.edu/jasonrw/blog/2013/03/create-custom-default-user-profile-os-x-107108

[3]: http://support.apple.com/kb/ht5077

<!-- https://s3.amazonaws.com/eng.grandroundshealth.com/images/kgm_post/ -->

[testing_user]: https://s3.amazonaws.com/eng.grandroundshealth.com/images/kgm_post/image001.png "Add Testing User"
[hot_corners]:  https://s3.amazonaws.com/eng.grandroundshealth.com/images/kgm_post/image003.png "Add Hot Corner"
[screen_pass]:  https://s3.amazonaws.com/eng.grandroundshealth.com/images/kgm_post/image005.png "Screen Saver Password"
[finder_pref]:  https://s3.amazonaws.com/eng.grandroundshealth.com/images/kgm_post/image007.png "Finder Preferences"
[keep_dock]:  https://s3.amazonaws.com/eng.grandroundshealth.com/images/kgm_post/image009.png "Keep In Dock"

