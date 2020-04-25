# AntennaPod CryptoBundle

This is a fork of [AntennaPod](https://github.com/AntennaPod/AntennaPod) (a great podcast manager for Android). This fork bundles a modern security provider ([Conscrypt](https://github.com/google/conscrypt)) instead of relying on the one provided by the OS (or the proprietary provider from the Google Services). The means modern network protocols and cipher suites on all versions of Android, such as TLSv1.3. This includes old versions like Android 4.4, KitKat.

I will maintain this fork by releasing a new build when AntennaPod releases a new version, starting with version 1.8.1. Hopefully my changes will be pulled back into the official project, at which point this fork will no longer be necessary.

## Motivation
I was motivated to create this fork after more and more podcasts I listen to stopped working with AntennaPod (Reply All, Software Freedom Podcast, Jimquisition/Podquisition, Boston's Favorite Son, ...). This was because the device I'm using is using Android 4.4.4, which doesn't support modern cipher suites. The device is small and perfect as a dedicated podcast+music player. Running a later version of Android on it caused several issues and I also didn't want to install and rely on the proprietary Google Services to use a newer security provider.

Looking at the most active [bug report for one of the errors](https://github.com/AntennaPod/AntennaPod/issues/2814) it seemed like no one was trying to solve the issue for the Free build (which is the one [provided](https://f-droid.org/en/packages/de.danoeh.antennapod/) by [F-Droid](https://f-droid.org/)), so I decided to fix it myself. This version of AntennaPod can access and download episodes for all podcasts I've tried without any problems. And I hope it will be of help to more people as more and more devices are stuck with old versions of Android.

By "Free build" I mean the version of AntennaPod that doesn't depend on Google's [proprietary](https://www.gnu.org/proprietary/malware-google.html) Libraries and Services. This is opposed to the "Play" builds that do. It's not about money (both versions of AntennaPod are free of charge), but about respecting the [Freedom](https://www.gnu.org/philosophy/free-sw.html) of the User. Forcing the user to install Google services just to keep using the app and device would be very unfortunate, as the number of users this affects will only increase in the future.

## Modern Protocols and Hardened Security
The latest version of Conscrypt is bundled and can is easily updated when building (by just specifying the new version number). Compared to the security provider included with Android (which can be more than 10 years old!), the bundled version offers security fixes, support for new protocols (like TLSv1.3) and new cipher suites. Is also disables many now obsolete cipher suites. Even though TLSv1.2 can be supported on as early as Android 4.4 (KitKat), and AntennaPod tries to enable it, the lack of modern cipher suites is what's causes most (if not all) current AntennaPod download issues (giving errors like SSL_ERROR_ZERO_RETURN and SSL23_GET_SERVER_HELLO from missing cipher suites and SSLv3 downgrades from missing cipher suites).

I also decided to make the app more future-proof by explicitly enable TLSv1.3 and TLSv1.2, and to disable all older protocols (SSLv3, TLSv1.0 and TLSv1.1 which are all considered obsolete these days). Also, on old Android versions AntennaPod manually enables the following obsolete cipher suites:

- TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
- TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA

These were disabled by default for a reason. Old implementations of CBC can suffer from padding vulnerabilities and SHA(-1) collision attacks are possible these days. I have removed the enabling of these cipher suites.

None of these changes affects the "Play" version of the app. It still uses Google services to try to update the security provider, and tries to enable the two unrecommended cipher suites on older Android versions. And since the security provider can not be guaranteed to be up to date, support for TLSv1.3 and modern cipher suites is not guaranteed. This also means that (this version of) the Free build offers far higher security and protocol support for modern usage, without requiring Google services or a modern version of Android. Maybe the Play flavor should be changed to bundle Conscrypt in the same way, but I will leave this decision to the AntennaPod developers. I'm only concerned with the Free builds and keeping support for older devices without forcing the user to depend on Google services.

## Branches
The following branches should be of interest:

- conscrypt_bundle: Contains only the changes I hope will be accepted upstream into the official version of AntennaPod. Just the changes for improved features and security.
- master_CryptoBundle: The default branch for this project page. Like conscrypt_bundle but it also renames the app to make it possible to install in parallel with the original and makes this README visible for anyone visiting the project. (Hi!)

## Installing
With each release (starting from version 1.8.1) I will include an apk (found in the "releases" tab) which can be installed (installing apk files usually requires changing some settings, which can be changed back after installation). You can also build it yourself (see below). The fork can be installed and used in parallel to the normal AntennaPod app. AntennaPod CryptoBundle will use its own name, directory and database. It's also possible to migrating all podcast information between AntennaPod and AntennaPod CryptoBundle.

## Migrating Podcast Data between AntennaPod and AntennaPod CryptoBundle
The two apps can be used independently. But if you want to easily move from one version to the other, you can use the built in support for exporting/importing databases. Each app will still use its own copy of the database and download new episodes to its own directory, but already downloaded episodes can be played by both apps. There are a few caveats to keep in mind if you want to share the podcasts and episodes between both apps in this way (instead of keeping the apps separated):

- Only do this with the same version (for example: AntennaPod 1.8.1 and AntennaPod CryptoBundle 1.8.1).
- This only syncs podcasts and playback status. Not settings. So you still need to configure your preferences (like automatic updates, audio player, queues, skip lengths, auto-deletion and similar).
- Use one "main" app at a time: don't let the databases diverge by downloading episodes on both apps (if you then overwrite one database by importing another, it will work but obviously leave untracked downloaded files which can be difficult to track down and clean up).
- Also remember not to enable automatic downloading of episodes for the same reason as above.
- If you delete one of the apps, all episodes downloaded by it will be removed. Worst case the remaining app will still show deleted episodes as downloaded, but when you try to play one it will be changed to not downloaded (but the playback position and status will remain).
- Later versions of Android might prevent access of files from other app directories in this way. But then you probably don't need this fork to get podcasts.

As always, you can instead use the OPML export/import or gpodder (which I have not tried myself). Or just manually enter the podcasts as usual. All of these options will keep the files of the apps completely separated, meaning files downloaded in one app will need to be downloaded in the other, but you will not have to worry about any of the points above.

## Updating Conscrypt
When building the app Conscrypt is automatically downloaded (through maven) and bundled for Free builds. Specifying a new version is as simple as changing conscryptVersion in build.gradle. Another option would be to set it to "latest.release", but that makes it harder to verify reproducible builds (and could also result in an unnoticed update to Conscrypt with untested changes). As of writing I have simply set it to the latest release, "2.4.0".

## Building AntennaPod CryptoBundle
You can build it yourself just like the original version, from either the source for a release or by cloning the git repository (and maybe checking out a branch, commit or tag you are interested in), but please read all of the following first:

It turns out the debug build of AntennaPod crashes on Android < 5.0. This is not caused by this fork, but is a problem with the original version. If you are on an affected version of Android (like KitKat, 4.4), I suggest making a release build. To build a release you need a key, a simple throwaway key can be created like this (from the root of the source code):

```
keytool -noprompt -genkey -v -keystore "app/keystore" -alias alias -storepass password -keypass password -keyalg RSA -validity 10 -dname "CN=Your Name, OU=dummy, O=dummy, L=dummy, S=dummy, C=US"
```

Also when doing a Free build, it's important to pass the "-PfreeBuild" flag to gradle:
```
./gradlew assembleFreeRelease -PfreeBuild
```
This is not mentioned in the official build instructions, but it's necessary (and meant to be done). Otherwise you will get an error that Conscrypt is missing (it does not get bundled with the Play build). If you are using Android Studio to build, open "Settings" -> "Build, Execution, Deployment" -> "Compiler" and add -PfreeBuild to "command-line Options".

Play builds are not changed from the original AntennaPod, and building them is no different. But be sure to use "assemblePlayRelease" or "assemblePlayDebug":
```
./gradlew assemblePlayRelease
```
The assembleRelease and assembleDebug targets do not work since it makes gradle try to compile both the code that assumes Conscrypt is bundled and the code requiring Google API/services instead, at the same time. This results in build errors: either Conscrypt is missing for the Free code, or Conscrypt conflicts with the Google API, depending on whether "-PfreeBuild" is passed or not - both cases results in build errors.
