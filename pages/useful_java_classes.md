---
layout: page
title: Useful Java Classes
permalink: /pages/useful-java-classes
---

Ever found a class that did that *exact thing* you were looking for... just 3 months later? This is my reference list of those classes.

Java
------

### MessageFormat
This class allows you to perform more advanced formattings than String.format or similar invocations. Most notably, it allows you to perform formatting based upon a number format passed into it, using correct grammatical forms based upon that number.

* JDK Source: [Since 1.5 (possibly earlier)](https://docs.oracle.com/javase/8/docs/api/java/text/MessageFormat.html)
* Android Source:
 * [java.text Version, since 1.0](https://developer.android.com/reference/java/text/MessageFormat.html)
 * [Android ICU Version, Preferred since 24.0](https://developer.android.com/reference/android/icu/text/MessageFormat.html) 


#### Examples
```java
MessageFormat.format("There {0,choice,0#are no files|1#is one file|1<are {0,number,integer} files}", 3)
```

**See Also:** [ChoiceFormat](https://docs.oracle.com/javase/8/docs/api/java/text/ChoiceFormat.html)

### Desktop
This class allows you to access basic functionality of the underlying computer in which the program is running, if such functionality is available. This makes sending simple emails, launching web pages, and printing documents easy, but also lacks much of the customization that using more specialized APIs provides.

* JDK Source: [Since 1.6](https://docs.oracle.com/javase/8/docs/api/java/awt/Desktop.html)
* Android Source: Desktop is not available on Android, but the functionality can be acheived and is more powerfully accessible through [Intent](https://developer.android.com/reference/android/content/Intent.html) objects.

####Examples
Send an email
```java
Desktop.getDesktop().mail(URI.create("mailto:person@gmail.gov"));
```
Open a website
```java
Desktop.getDesktop().browse(URI.create("http://google.com"));
```
Open a file (uses the system default program)
```java
Desktop.getDesktop().open(new File("/path/to/file"));
```
Print a file (uses the system default program to print)
```java
Desktop.getDesktop().print(new File("/tmp/out.txt"));
```

### Externalizable
Externalizable is a specialized form of Serializable that utilizes public, interface declared methods instead of magic reflection & using the private `readObject` and `writeObject` methods. This will prevent default serialization, and instead use the methods provided in Externalizable to do the serialization. **Note**: A public, no argument constructor is explicitly required for Externalizable, but not for Serializable (except on non-final classes). There is an inspection in IntelliJ/Android Studio to verify this.

* JDK Source: [Since 1.1](https://docs.oracle.com/javase/8/docs/api/java/io/Externalizable.html)
* Android Source: 
 * [Since 1.0](https://developer.android.com/reference/java/io/Externalizable.html) objects.
 * **Note:** On android [Parcelable](https://developer.android.com/reference/android/os/Parcelable.html) should be preferred to Externalizable in most cases.

Android
------

### LocalBroadcastManager
Allows sending of a [broadcast](https://developer.android.com/reference/android/content/Context.html#sendBroadcast%28android.content.Intent%29) to application local BroadcastReceivers more easily.

* Android Source: [Android Support Library v4](https://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html)

####Example
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
```
