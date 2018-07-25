# How to set up Android test environment on Jenkins

## Requirements

* Install Java(1.8.0_144) [Click here](https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-get-on-ubuntu-16-04)
* Install Appium(1.8.1) [Click here](http://ubuntubegin.blogspot.com/2015/07/how-to-setup-appium-in-ubuntu.html) 
* If there is any issue while installing Brew, then [Click here](https://github.com/Linuxbrew/brew) and follow all steps as per Appium document.
* Install Android Studio(26.1.1) [Link 1](https://developer.android.com/studio/install#linux) or [Link 2](https://askubuntu.com/questions/634082/how-to-install-android-studio-on-ubuntu)
* Start Appium server programmatically [Click here](http://www.seleniumeasy.com/appium-tutorials/how-to-start-appium-server-programmatically)

## Steps to set android path for ubuntu

``` 
$ whereis android

E.g  android: /var/lib/android-sdk/tools/android

$ vi ~/.bashrc (or vi ~/.rc depending on the shell)

# Insert Android path to shell
export ANDROID_HOME=/var/lib/android-sdk
export PATH=$ANDROID_HOME/platform-tools:$PATH
export PATH=$ANDROID_HOME/tools:$PATH
export PATH=$PATH:$ANDROID_HOME/build-tools 

# Now check below command
$ adb devices
```

## Nested virtualization for Google cloud VM Instances 
* Nested virtualization [Click here](https://cloud.google.com/compute/docs/instances/enable-nested-virtualization-vm-instances)

## Install KVM
* Install KVM(2.5.0) [Click here](http://xmodulo.com/use-kvm-command-line-debian-ubuntu.html)


## Steps to Download system images for emulator

* System images are pre-installed Android operating systems, and are only used by emulators. 
* For e.g: API Level 24-25 is Nougat.

```
$ cd $ANDROID_HOME/tools/bin
$ ./sdkmanager "system-images;android-25;google_apis;x86"
$ ./sdkmanager --licenses

```

## Steps to Create Emulator

```

$ cd $ANDROID_HOME/tools/bin
$ ./avdmanager create avd -n <DeviceName> -k "system-images;android-25;google_apis;x86"

```

## Verify Emulator

```
$ cd $ANDROID_HOME/tools/bin
$ ./avdmanager list avds

--------------------
Available Android Virtual Devices:
    Name: testEmulator
    Path: /var/lib/jenkins/.android/avd/credit.avd
  Target: Google APIs (Google Inc.)
          Based on: Android 6.0 (Marshmallow) Tag/ABI: google_apis/x86
--------------------

```

## Jenkins environment configuration

* Login to jenkins.
* Click on Manage Jenkins from left menu.
* Click on Configure System.
* As per your configuration add below environment variables.

```

Name : ANDROID_HOME
Value: /var/lib/android-sdk

Name : PATH
Value: /home/linuxbrew/.linuxbrew/sbin:/home/linuxbrew/.linuxbrew/bin:/var/lib/android-sdk/tools:/var/lib/android-sdk/platform-tools:/var/lib/android-sdk/build-tools

```


## Jenkins job configuration(Execute below script using shell)
```

# stop adb
adb kill-server

#Start adb
adb start-server


#Start the emulator
$ANDROID_HOME/tools/emulator -avd <DeviceName> -no-window -wipe-data &
EMULATOR_PID=$!


# Wait for Android to finish booting
WAIT_CMD="adb wait-for-device shell getprop init.svc.bootanim"
until $WAIT_CMD | grep -m 1 stopped; do
  echo "Waiting..."
  sleep 1
done


# Unlock the Lock Screen
adb shell input keyevent 82


# Clear and capture logcat
adb logcat -c
adb logcat > build/logcat.log &
LOGCAT_PID=$!

# Allow “unknown sources” from Terminal without going to Settings App
adb shell settings put global install_non_market_apps 1

# Go to android team project dir
cd /var/lib/jenkins/qa_automation/android_apps/AndroidTeamProject
git checkout -f develop;
git pull origin develop;
./gradlew installStagingDebug;

# Check devices
adb devices;

# Go to testing team project dir
cd ~/qa_automation/test_automation/TesterTeamProject;
mvn clean compile test 


# Stop the background processes
kill $LOGCAT_PID
kill $EMULATOR_PID

```




