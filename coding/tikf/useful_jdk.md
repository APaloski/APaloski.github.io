---
layout: page
title: Useful Java (JDK) Classes
permalink: /coding/useful-java-classes
---

Useful classes in the JDK (and maybe Android SDK) that I keep forgetting about
------

### MessageFormat
This class allows you to perform more advanced formattings than String.format or similar invocations. Most notably, it allows you to perform formatting based upon a number format passed into it, using correct grammatical forms based upon that number.

* JDK Source: [Since 1.5 (possibly earlier - it's undocumented)](https://docs.oracle.com/javase/8/docs/api/java/text/MessageFormat.html)
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

#### Examples

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

### ExecutorCompletionService (and indirectly CompletionService)
Acts a point of rejoining for a set of tasks that are submitted to an Executor. Instead of awaiting the first submitted task or polling tasks to see if they are "actually done", the service allows you to synchronously acquire results as they complete. This allows you to take synchronous actions on the more quickly completing entries as they come in, instead of possibly bottle necking on the longest running async task and then having to do all of the synchronous processing once it comes in.

* JDK Source: [Since 1.5](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ExecutorCompletionService.html)
  * See also: [CompletionService (Interface) Since 1.5](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/CompletionService.html)
* Android Source: [Since 1.0](https://developer.android.com/reference/java/util/concurrent/ExecutorCompletionService.html)
  * See also: [CompletionService (Interface) Since 1.0](https://developer.android.com/reference/java/util/concurrent/CompletionService.html)

#### Examples
```java
public void writePageContentToDatabase(List<URI> pageNames) {
  ExecutorCompletionService<String> service = new ExecutorCompletionService<>(mBackgroundLoadingPool);
  for(URI uri : pageNames) {
    service.submit(new PageLoadingCallable(uri));
  }
  for(int countCompleted = 0; countCompleted < pageNames.size(); countCompleted++) {
    try {
      // We get them in first come first serve result order, instead of awaiting the first 
      // submitted and then running our long running sync operation against it.
      Future<String> pageContentFuture = service.take();
      savePageToDatabase(pageContentFuture.get());
    } catch (InterruptedException p_e) {
      Thread.currentThread().interrupt();
    } catch (ExecutionException p_e) {
      //TODO Handle me!
    }
  }
}
```

#### Articles
[DZone: ExecutorCompletionService](https://dzone.com/articles/executorcompletionservice) 

### LineNumberReader
An implementation of BufferedReader that also provides the ever useful ```getLineNumber()``` method. This method allows you to get the current line number that is being read, and allows easy output of a line numbered document from a regular one (probably more useful in IDEs, but could be useful!).

#### Examples
```java
public void readNumbered(File file) {
  try (LineNumberReader reader = new LineNumberReader(new FileReader(file))) {
    for(String s = reader.readLine(); s != null; s = reader.readLine()) {
      System.out.println(reader.getLineNumber() + " " + s);
    }
  }
}
```

* JDK Source: [Since 1.1](http://docs.oracle.com/javase/7/docs/api/java/io/LineNumberReader.html)
* Android Source: [Since 1.0](https://developer.android.com/reference/java/io/LineNumberReader.html)
