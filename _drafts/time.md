---
layout: post
title: Customizing Java Time: Playing around with Clocks
---

There are at least tens of articles, if not hundreds out there describing the basics of the [java.time](https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html#package.description) APIs introduced in Java 8. Including one released by Oracle themselves, found [here](http://www.oracle.com/technetwork/articles/java/jf14-date-time-2125367.html). But from my experience most of them barely touch on making extensions to the API. So if you're looking for the basics, this article isn't for you; if you want to do some first step customization it is!

# Clocks and you

One of the most stepped over classes in the time APIs is the humble [Clock](https://docs.oracle.com/javase/8/docs/api/java/time/Clock.html). This class acts as the core that generates your `LocalDate`, `LocalDateTime`, etc. classes for a given time period when you don't know the *exact* value you're working with.

Clocks are made up of two simple components:
* The `ZoneId`: This is the time zone that the clock resides in, this allows for classes such as `ZonedDateTime` to extract the appropriate zone from the clock. Clocks can have their time zones changed via the [withZone(ZoneId)](https://docs.oracle.com/javase/8/docs/api/java/time/Clock.html#withZone-java.time.ZoneId-) method.
* An `Instant`: This is the current time *as seen by the Clock*. This can be obtained either as an `Instant` object, or as a number of millis since the epoch *in UTC*. Both of these methods must return equivalent values if invoked at the exact same time.

By default, the JDK provides the following Clock types, as well as some overloads for working with TimeZones:
* System Clock: This is a Clock based upon the current time on the System, as determined by `System.currentTimeMillis()`. By default it uses the systems default timezone as its own.
* Fixed Clock: This is a clock that always returns the same `Instant` instance, set at a given `ZoneId`. This is especially useful for unit testing.
* Offset Clock: This is a `Clock` that works by taking the time result of another `Clock`, and then offsetting that value by a fixed duration amount.
* Tick Clock: This, just like the Offset Clock above, works by taking the result of another `Clock` instance, then it truncates that result to the nearest Duration (by default providing minutes and seconds).

But just like all code, for all uses that are anticipated by library designers there's a hundred that aren't. So why don't we create some of our own clocks.

# Writing your own Clock

So you want to write your own Clock, simple enough. We just need a ZoneId and Instant right? Exactly... kind of. There's a bit of weirdness with serialization, thread safety and then contract requirements with `equals` and `hashCode`. But let's ignore that for now and create our easy example.

## The Basics

First things first: What do we need our Clock to do? Where I work, we have a requirement to always pull fresh time from an external source whenever we need it. So what I want is a Clock that will do that magically for me, but instead of hard coding in the `getExternalTime` method, why don't we just make it use a `Supplier`, so that it can be expanded in the future?

```java
public class SupplierClock {
  private LongSupplier mMillisSupplier;
  
  //Constructor, zoneId, other boilerplate...

  @Override
  public Instant instant() {
    //Construct a new Instant from our milliseconds
    return Instant.ofEpochMillis(millis());
  }
  
  @Override
  public long millis() {
    return mMillisSupplier.getAsLong();
  }

}
```

The one important thing to note here is that one of the instant or millis methods delegates to the other. This ensures that calling one of those methods instead of the other will return the same value.

But this is simple enough, and then with a simple global we can make a new Clock that delegates to our current time supplier!

```java
public static final Clock EXTERNAL 
  = new SupplierClock(TimeSource::getExternalTime); 
```

Now, we can follow company policy of using the external time source for all new time instances.

## TODO ZONE ID!

## The weird parts

So I mentioned before that there were a few more complexities that we would need to tackle relating to thread safety, serialization and `equals`/`hashCode`. So lets take a look:

### Equals and HashCode

The `Clock` class states the following on equals:
>  Clocks should override this method to compare equals based on their state and to meet the contract of {@link Object#equals}. If not overridden, the behavior is defined by {@link Object#equals}

So for our class, this means we should define our equals method as something like
```java
@Override
public boolean equals(Object obj) {
  return (obj instanceof SupplierClock) &&
         (obj.getZone().equals(getZone())) &&
         ((SupplierClock) obj.mMillisSupplier.equals(mMillisSupplier));
}
```

However, this will still have some strange behavior. This is mostly due to lambdas using mem equals for equality comparisons, however from local testing two separate definitions of the same lambda (or method reference) in the same method will result in two different instances, but every call to a factory method such as [Function.identity()](https://docs.oracle.com/javase/8/docs/api/java/util/function/Function.html#identity--) will always return the same instance. So in general the equals here, due to the supplier, is going to act strangely.

Likewise, `Clock` states for hash code:
> Clocks should override this method based on their state and to meet the contract of {@link Object#hashCode}. If not overridden, the behavior is defined by {@link Object#hashCode}

So for our class, this means a definition like
```java
@Override
public long hashCode(Object obj) {
  return Objects.hash(getZone(), mMillisSupplier);
}
```

### Noting Thread Safety and Serialization

The clock class specifically states:
> Implementations should implement {@code Serializable} wherever possible and must document whether or not they do support serialization.

So, the first question then is: Can we ever be Serializable? My first instinct is yes, someone could directly implement our Supplier on a class (instead of using a lambda or method reference), and that could be serialized. But is there other ways? The answer is yes.

Java 8 introduced the ability to cast objects to an *intersection* of class bounds, meaning that silly things like the following are possible:

```java
LongSupplier sup = (Serializable & LongSupplier) () -> 100L;
```

This creates a lambda that can be serialized (how it's internal state is actually represented, that's a completely different question). If you want some additional reading see [Stack Overflow](http://stackoverflow.com/questions/22807912/how-to-serialize-a-lambda).

The clock implementation also states:
> All implementations that can be instantiated must be final, immutable and thread-safe.

The final and immutable parts of this are easy peasy, but the thread-safe part is unfortunately not as easy. In our example case of using a time server, we're using a global, external resource for it so that's fine. But what if someone wanted to use a class of theirs that was implicitly *not* thread safe? Unfortunately, that won't work with the contract.

There's no way to us to ensure that what we're given matches that description, so unfortunately we need to pass the buck here and add our own javadoc that states that the Supplier given to us must be thread safe (maybe we want to provide an implementation that takes a `ThreadLocal` or throws an exception if it's on an unexpected thread.

# Wrapping up

Creating your own Clock honestly isn't that bad, but you do need to be careful with the small contract parts. As I've delved deeper into the java time api I've found more and more small gotcha moments like that, so you should read the contracts carefully.


