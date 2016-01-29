# Little Fluffy Location Library #
## Battery-friendly background location updates for Android, using Google Play Location where available ##

## v17 beta including Google Play Services (GMS) fused location provider is available now! For production code use v15 ##

Drop this tiny library into your Android app, configure your manifest, initialise it in your application's onCreate method... and that's it!

Location updates will be broadcast to your app periodically, including after phone reboot. And your users won't complain about it killing their batteries.

Perfect for widgets that require periodic updates in background, and for apps that need a reasonably current location available to them on startup. Also works great for apps that do something with location periodically, such as using it to get updates from a server.

Based on the concepts in, and some code adapted from, [android-protips-location](http://code.google.com/p/android-protips-location/) by [Reto Meier](https://plus.google.com/111169963967137030210), from his blog post [A Deep Dive Into Location](http://android-developers.blogspot.co.uk/2011/06/deep-dive-into-location.html).

Requires Android 2.1 and up. Works best with 2.2 and up.

Now works with the closed-source and copyrighted Google Play Services (GMS) fused location provider, if available. If not (eg Kindle Fire, Nokia X) it falls back to using Android Open Source Project location providers.

The library works by using Froyo's passive location listener (only possible with Android 2.2 and up, hence why it works best with it), which listens to location updates requested by other apps on your phone. The most accurate location is broadcast to your app approximately every 15 minutes. If a location update hasn't been received from another app for an hour, the library forces a location update of its own.

Features:
  * location updates broadcast to your app approximately every 15 minutes (if a new location update was found)
  * if no location update found for an hour, an update is requested
  * optionally, every location update is broadcast
  * frequency of regular broadcasts (default 15 minutes) and forced updates (default 60 minutes) configurable
  * force update feature
  * get latest location feature
  * choose the most accurate location from the location providers available
  * abstracts away the complexity of Google Play Services versus Android Open Source
  * debug output optional

Written by Kenton Price, [Little Fluffy Toys Ltd](http://www.littlefluffytoys.mobi)

---

## Downloads ##
v17 and later [downloads available here](https://drive.google.com/folderview?id=0B6Cy8bGPZRCcVHh3V1otUFJqdzQ&usp=sharing); earlier versions under the Downloads tab

---

## What's new - v18 - May 2014 (COMING SOON) ##
  * release quality code. For current production apps, please use v15
## What's new - v17 - April 2014 ##
  * BETA - not for production code!
  * uses Google Play Services (GMS) fused location provider, where available. This requires you to add a meta-data line in your manifest, and configure the Google Play Services SDK. See details below
  * Please help test v17 but do NOT use in production apps yet - for production apps use v15
## What's new - v16 - February 2014 ##
  * ignore this - it has fundamental bugs!
## What's new - v15 - June 2013 ##
  * fixed an issue whereby if a user has GPS location services enabled but not network (wifi/celltower) services enabled, the library would never return an on-demand or after-timeout location
  * workaround for a bug in Android 4.2.1 so the one-shot location request doesn't keep on repeating forever
  * allow the library to only request fine accuracy instead of course accuracy when it asks for its own location hits, by setting LocationLibrary.useFineAccuracyForRequests(true)
  * exposed the passive location provider in LocationInfo.lastProvider
## What's new - v14 - May 2013 ##
  * bugfix
## What's new - v13 - May 2013 ##
  * changed the default location stabilisation timeout from 10 seconds to 5 seconds. This is the timeout after a location update is received, after which the library assumes no further updates are forthcoming, and so it then uses the most accurate of the flurry of updates that it received. It was formerly 10 seconds, and can be set to this (or any other value) by setting LocationLibrary.stableLocationTimeoutInSeconds
## What's new - v12 - May 2013 ##
  * library now throws UnsupportedOperationException during initialiseLibrary if the device has no location providers, so that you can handle this appropriately in your code
  * on the very first execution, the first location hit will be much quicker - used to be 10 seconds, but could now be under a second

---

## Getting started ##
To use the library, add **littlefluffylocationlibrary.jar** as an external JAR into your Android project. In Eclipse, this is easiest done by creating a folder in your project called libs, copying the file into it, refresh the project, right-click the .jar file, and  then select Add to build path.

Next, edit your application's manifest, adding the following:

```
...
<manifest {...}>
...
  <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
  <uses-feature android:name="android.hardware.location" android:required="true" />
  <uses-feature android:name="android.hardware.location.gps" android:required="false" />
  ...
  <application {...}>
    ...
    <service android:name="com.littlefluffytoys.littlefluffylocationlibrary.LocationBroadcastService" />
    <receiver android:name="com.littlefluffytoys.littlefluffylocationlibrary.StartupBroadcastReceiver" android:exported="true">
      <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />  
      </intent-filter>  
    </receiver>
    <receiver android:name="com.littlefluffytoys.littlefluffylocationlibrary.PassiveLocationChangedReceiver" android:exported="true" />
    ...
    <meta-data android:name="com.google.android.gms.version" android:value="@integer/google_play_services_version"/>
  </application>
  ...
</manifest>
```

Ensure that your application extends the Application class, and inside its `onCreate()` method, call

`LocationLibrary.initialiseLibrary(getBaseContext(), "com.your.package.name");`

substituting your package name as appropriate. Be prepared to catch an UnsupportedOperationException which indicates that the device has no location providers.

And now you're ready to use it!

The app broadcasts an action of `com.your.package.name.littlefluffylocationlibrary.LOCATION_CHANGED` with a SerializableExtra of type `LocationInfo` called `com.littlefluffytoys.littlefluffylocationlibrary.LocationInfo` that contains the location details. So you need to **set up a broadcast receiver that listens for this action**. Doing this is beyond the scope of this documentation, but you can refer to the sample application in the downloads for an example of how to do so.
## The LocationInfo object ##
The broadcast gives you a LocationInfo object containing the following five fields:
  * lastLocationUpdateTimestamp - the time the last location update was recorded, in milliseconds
  * lastLocationBroadcastTimestamp - the time a location update was last broadcast, in milliseconds
  * lastLat - the last location update's latitude
  * lastLong - the last location update's longitude
  * lastAccuracy - the last location update's accuracy, in metres
You can also just create a LocationInfo object at any time, and it will have the latest location info within it.

The LocationInfo object has some useful methods on it:
  * refresh - refresh the fields with the latest location info
  * anyLocationDataReceived - has any location data been received since the last reboot
  * anyLocationDataBroadcast - has any location data been broadcast since the last reboot
  * hasLatestDataBeenBroadcast - true if the location data in the object has already been broadcast
  * getTimestampAgeInSeconds - return the age of the last location update in seconds

---

## Advanced topics ##
### Debugging ###
Debug output is off by default. To switch it on, add the following before the call to `initialiseLibrary`:

`LocationLibrary.showDebugOutput();`

### LocationLibraryConstants ###

This contains several useful constants and functions:
```
public static final long DEFAULT_ALARM_FREQUENCY = AlarmManager.INTERVAL_FIFTEEN_MINUTES;
public static final int DEFAULT_MAXIMUM_LOCATION_AGE = (int) AlarmManager.INTERVAL_HOUR;
public static final String LOCATION_BROADCAST_EXTRA_LOCATIONINFO = "com.littlefluffytoys.littlefluffylocationlibrary.LocationInfo";
public static String getLocationChangedPeriodicBroadcastAction()...
public static String getLocationChangedTickerBroadcastAction()...
```
### initialiseLibrary overloaded methods ###
You can use any of the other overloaded methods of `initialiseLibrary` if you require fine tuning of the configuration parameters. For example, during testing, you could set `alarmFrequency` to 60000 (1 minute in milliseconds) and `locationMaximumAge` to 120000 (2 minutes in milliseconds).

The following parameters are available across the various methods:

`alarmFrequency`: How often to broadcast a location update in milliseconds, if one was received. For battery efficiency, this should be one of the available inexact recurrence intervals recognised by [AlarmManager.setInexactRepeating(int, long, long, PendingIntent)](http://developer.android.com/reference/android/app/AlarmManager.html). You are not prevented from using any other value, but in that case Android's alarm manager uses setRepeating instead of setInexactRepeating,and this results in poorer battery life. The default is `AlarmManager.INTERVAL_FIFTEEN_MINUTES`.

`locationMaximumAge`: The maximum age of a location update. If when the alarm fires the location is older than this, a location update will be requested. The default is `AlarmManager.INTERVAL_HOUR`.

`broadcastEveryLocationUpdate`: If true, in addition to broadcasting periodic updates, it broadcasts every location update as it is found, using intent action `com.your.package.name.littlefluffylocationlibrary.LOCATION_CHANGED_TICK`. The default is `false`.

`broadcastPrefix`: It is strongly recommended that you provide a unique value for this parameter, such as your package name. In an earlier version of the Little Fluffy Location Library, the broadcast action to signify the location had changed was always the same - specifically, `com.littlefluffytoys.littlefluffylocationlibrary.LOCATION_CHANGED` (and the `LOCATION_CHANGED_TICK` variant, if `broadcastEveryLocationUpdate` is true). This meant that if multiple apps using the library were installed on the device, each app would receive the broadcast sent by any one of the apps. At the time of writing, the very popular Groupon app uses the library and sends this constant broadcast, so it is a significant issue. Therefore, to ensure that your app only responds to broadcasts sent from your app, and not duplicate broadcasts from other apps, please provide a unique value for this parameter. However, if you must maintain backward compatibility with the prior version of this library and you are for some reason unable to rework your client app, just use `com.littlefluffytoys` for this parameter.

### Accessing the latest location ###
To get the latest location info without waiting for a broadcast, just create a LocationInfo object:

`LocationInfo latestInfo = new LocationInfo(getBaseContext());`

If you already have a LocationInfo object, just call its `refresh()` method.

### Forcing a location update ###
To force a location update, call `LocationLibrary.forceLocationUpdate();` and soon you'll get a broadcast containing the latest location.

---

## Little Fluffy Toys loves your feedback! ##
I'm Kenton Price, co-founder of Little Fluffy Toys Ltd, a specialist Android consultancy. We help companies get the hard stuff like this right, and we help turn their ideas into truly great Android apps. We also publish apps of our own on Google Play Store. Please [visit our website](http://www.littlefluffytoys.mobi), [follow us on Twitter](https://twitter.com/#!/LFT_Android), or [email us](mailto:support@littlefluffytoys.mobi).