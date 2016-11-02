---
layout: page
title: Useful Android Classes
permalink: /coding/tikf/useful-android
---

Useful Android SDK/Support library classes
------

### LocalBroadcastManager
Allows sending of a [broadcast](https://developer.android.com/reference/android/content/Context.html#sendBroadcast%28android.content.Intent%29) to application local BroadcastReceivers more easily. This acts as an application local (e.g. no inside or outside communication) global invoke/callback mechanism, based upon the already powerful android Intent mechanism. One suggested use for this pattern is a service locator of sorts, as anything can register against the LocalBroadcastManager.

* Android Source: [Android Support Library v4](https://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html)

#### Example

Registering for broadcasts

```java
//onStart
LocalBroadcastManager.getInstance(this).register(mBroadcastReceiver);
```

Unregistered for broadcasts

```java
//onStop
LocalBroadcastManager.getInstance(this).unregister(mBroadcastReceiver);
```

Broadcasting to local classes

```java
//Blocking
LocalBroadcastManager.getInstance(this).sendBroadcastSync(new Intent());

//Non-blocking
LocalBroadcastManager.getInstance(this).sendBroadcast(new Intent());
```

#### Things to note
Receives registered with this class *will not* receive notifications from outside applications, likewise outside applications cannot receive notifications from the LocalBroadcastManager. Additionally, you cannot register for this class from your application manifest, you must do it dynamically at runtime. Finally, this class holds all of the dangers of memory leaking when using an anonymous class that all other registrations do, but is slightly more dangerous because all references are held *as strong references* and thus will live for the life of the singleton instance.

Useful Android Libraries
------
This is a set of libraries that I like to use in every project and find myself wondering why I don't in any project I don't have.

### Timber
This small library simplifies android logging, allows format based logging and allows "planting" and "uprooting" (registering and unregistering, respectively) of different logging implementations at runtime.

#### Reference
[Timber](https://github.com/JakeWharton/timber)

#### Download
```compile 'com.jakewharton.timber:timber:4.3.1'```
