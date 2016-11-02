---
layout: page
title: Useful Java Classes
permalink: /pages/useful-java-classes
---

Ever found a class that did that *exact thing* you were looking for... just 3 months later than when you needed it? This is my reference list of those classes.

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


Unit Testing
------

### ParameterSuppliedBy
This class allows you to create a custom annotation for use with JUnit Theories where the arguments to that annotation can be used to determine what values are passed into the theory.

* JUnit source: [Available in 4.12](http://junit.org/junit4/javadoc/4.12/org/junit/experimental/theories/ParametersSuppliedBy.html)


#### Examples
```java
@Retention(RetentionPolicy.RUNTIME)
@ParametersSuppliedBy(MySupplier.class)
public @interface MyTheoryType {}

public class MySupplier extends ParameterSupplier {
  @Override
  public List<PotentialAssignment> getValueSources(final ParameterSignature sig) throws Throwable {
    MyTheoryType myTheory = sig.getAnnotation(MyTheoryType.class);
    //Use any parameters on the annotation
    //Generate any data
    return Arrays.asList(PotentialData.forValues("Name", "Value"));
  }
}

```

**See Also:** [JUnit Blog](http://www.junit-blog.com/search/label/parameterSuppliedBy)

### System Rules
Deferring from the standard 'single class' set up, this is actually a (very) small library that allows you to emulate interactions with the Java System class. This allows you to easily redirect or mock System.in, System.out or System.err, as well as allowing setting of system/environment properties and then restoring them after a unit test. Finally, it also has some security manager configuration help.

#### Examples
See: [Github.io page for the project](stefanbirkner.github.io/system-rules/)

#### Library dependency:
See: [Multi-dep declaration page](http://stefanbirkner.github.io/system-rules/download.html)

Android
------

### LocalBroadcastManager
Allows sending of a [broadcast](https://developer.android.com/reference/android/content/Context.html#sendBroadcast%28android.content.Intent%29) to application local BroadcastReceivers more easily.

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
```
