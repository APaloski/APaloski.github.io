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


**Example:**
```java
MessageFormat.format("There {0,choice,0#are no files|1#is one file|1<are {0,number,integer} files}", 3)
```

**See Also:** [ChoiceFormat](https://docs.oracle.com/javase/8/docs/api/java/text/ChoiceFormat.html)


Android
------

