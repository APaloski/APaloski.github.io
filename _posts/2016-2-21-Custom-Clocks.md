---
layout: post
title: Customizing Java Time - Playing around with Clocks
---

There are at least tens of articles, if not hundreds out there describing the basics of the [java.time](https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html#package.description) APIs introduced in Java 8. Including one released by Oracle themselves, found [here](http://www.oracle.com/technetwork/articles/java/jf14-date-time-2125367.html). But from my experience most of them barely touch on making extensions to the API. So if you're looking for the basics, this article isn't for you; But if you want to do some first step customization it is!

# Clocks and you

One of the most stepped over classes in the time APIs is the humble [Clock](https://docs.oracle.com/javase/8/docs/api/java/time/Clock.html). This class acts as the core that generates your `LocalDate`, `LocalDateTime`, etc. classes for a given time period when you don't know the *exact* value you're working with. 

Clocks are made up of two simple components:
* The `ZoneId`: This is the time zone that the clock resides in (sadly, the name TimeZone was already taken in the Calendar era), this allows for classes such as `ZonedDateTime` to extract the appropriate zone from the clock. Clocks can provide an equivalent instance, but with a different time zone by using the [withZone(ZoneId)](https://docs.oracle.com/javase/8/docs/api/java/time/Clock.html#withZone-java.time.ZoneId-) method.
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

First things first: What do we need our Clock to do? Where I work, we have a requirement to always pull fresh time from an external source whenever we need a reliable source for time. So what I want is a Clock that will pull from that external source for me, so I can just use all of the built in factory methods.

So lets assume that we have an existing function for pulling the time from the server, defined as 
```java
  /**
   * @return the time on the time server, as millis since epoch UTC.
   */
  public static long getServerTime() {
    //HTTP calls and other fun
  }
```

So with knowing the above, lets create the basic bare bones of our clock.

```java
public class ServerClock extends Clock {
  
  //TODO Constructor/ZoneId information

  @Override
  public Instant instant() {
    //Construct a new Instant from our milliseconds
    return Instant.ofEpochMillis(millis());
  }
  
  @Override
  public long millis() {
    return ServerTimeProvider.getServerTime();
  }

}
```

With this, we have our basics: We can get an `Instant` or we can get millis, and most importantly if called at the exact same time they will return equivalent values from the `millis()` and `instant()` methods.

But in addition to the time, we also need to provide ZoneId support via the `getZone()` and `withZone(ZoneId)` methods. 

```java
public class ServerClock extends Clock {
  
  private final ZoneId mZoneId;

  public ServerClock(ZoneId zoneId) {
    mZoneId = Objects.requireNonNull(zoneId);
  }
  
  @Override
  public ZoneId getZone() {
    return mZoneId;
  }
  
  @Override
  public ServerClock withZone(ZoneId zone) {
    if(zone.equals(mZoneId)) {
      return this; //No need for a new object with the same zone
    }
    return new ServerClock(zone);
  }
  
  //Time accessors are the same.

```

So lets take a look at this. The first thing to note is that we don't alter our time accessors, this is because both *must* always return as millis since the epoch in UTC, so any change in time zone doesn't affect it. That makes the ZoneId just a simple property of our Clock, with no special handling needed.


## The weird parts

I mentioned before that there were a few more complexities that we would need to tackle relating to thread safety, serialization and `equals`/`hashCode`. So lets take a look:

### Equals and HashCode

The `Clock` class states the following on equals:
>  Clocks should override this method to compare equals based on their state and to meet the contract of {@link Object#equals}. If not overridden, the behavior is defined by {@link Object#equals}

So for our class, this means we should define our equals method as something like
```java
@Override
public boolean equals(Object obj) {
  return (obj instanceof ServerClock) &&
         (obj.getZone().equals(getZone()));
}
```

Likewise, `Clock` states for hash code:
> Clocks should override this method based on their state and to meet the contract of {@link Object#hashCode}. If not overridden, the behavior is defined by {@link Object#hashCode}

So for our class, this means a definition like
```java
@Override
public long hashCode(Object obj) {
  return Objects.hash(getZone());
}
```

### Handling Thread Safety and Serialization

The clock class specifically states:
> Implementations should implement {@code Serializable} wherever possible and must document whether or not they do support serialization.

As long as someone has all libraries we use to hit the server on the class path, we can be serialized. So we simply implement Serializable and document it on the class for this one.

The clock implementation also states:
> All implementations that can be instantiated must be final, immutable and thread-safe.

The final and immutable parts of this are easy, and are handled by simply leaving our ZoneId field as final. The thread safety is entirely dependent on the thread safety of our `getServerTime()` method. For this example I'll assume it is thread safe, as we use it globally from any thread already, but it is something that needs to be kept if you implement something similar.

# Improving our design

This example was by design contrived and simple. In a real world situation you likely would have multiple separate time providers that all want to do something similar to this, and you don't want to create a separate subclass for each one. Due to this, you would want to use something more generic like a clock running off of a `Supplier<Instant>` or `LongSupplier` that provides your UTC since epoch time.

This creates additional challenges with Serialization (see: [How to serialize a lambda](http://stackoverflow.com/questions/22807912/how-to-serialize-a-lambda)), equality (lambdas always check instance equality, but depending on how they're initialized may be one or more instances), and thread safety (the lamdba creator must ensure thread safety) to maintain the contracts of Clock.

An implementation of the `SupplierClock`,  as well as other Clocks and other Java 8 time utilities can be found in the [Timestreams](https://github.com/APaloski/Timestreams) library.

# Wrapping up

Creating your own Clock honestly isn't that bad, but you do need to be careful with the small contract parts. As I've delved deeper into the java time API I've found more and more small gotcha moments like that, so you should make sure to read the contracts on all of the classes you're implementing carefully.

The full source code for the ServerClock implementation can be found [here](https://gist.github.com/APaloski/35234b40f1ef6340567f)
