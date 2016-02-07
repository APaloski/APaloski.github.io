---
layout: post
title: Writing a custom Collector for Java 8 Streams
---

Java 8 introduced a standard approach to a functional pipeline in Java, through the [Stream](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html) apis which are attached to all Collection implementations. Streams allow you to manipulate the collection by mapping, filtering and reducing the elements; but while filtering and mapping are easy, the reduction hides behind the less than clear [Collector](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collector.html) API.

## Introduction to Collectors

### What is a Collector?

A [Collector](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collector.html) is described by the JDK as:
> A mutable reduction operation that accumulates input elements into a mutable result container, optionally transforming the accumulated result into a final representation after all input elements have been processed. Reduction operations can be performed either sequentially or in parallel.

In laymans terms, this means its an operation that takes what's in your stream, shuffles the elements into a single data type (such as adding them to a list), and then gives you back that list. It can happen on both parallel and non-parallel streams.

The `Collector` has three generic arguments (in practice when using a `Stream` you'll likely never need them, but it's good to know when writing a `Collector`):
* `T` - The incoming type of elements from the `Stream`
* `A` - The type of intermediate container of the `Collector`. This is almost always obfuscated in factory methods by returning a wildcard because it's purely an implementation detail.
* `R` - The resutling type of the `Collector`. This is the `List<T>` in a Collector that produces a List. It's the final type that your variable will be set equal to when all is said and done.

### Collectors from the JDK

The JDK provides a number of built in Collectors, featured in the [Collectors](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html) class. These allow you to do normal functions such as creating [lists](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html#toList--), [sets](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html#toSet--) and [maps](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html#toMap-java.util.function.Function-java.util.function.Function-) from your stream, [concatenate](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html#joining-java.lang.CharSequence-java.lang.CharSequence-java.lang.CharSequence-) the elements into a formatted string and get the [sum](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html#summingDouble-java.util.function.ToDoubleFunction-) of all values in the stream. The Collectors class even allows you to have some customization using the [collectingAndThen](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html#collectingAndThen-java.util.stream.Collector-java.util.function.Function-) method. 

Using these built in Collectors is relatively simple, let's say I have a Collection of People, and want to get a list of all of their names.

```java
persons .stream()
        .map(Person::getName)
        .collect(Collectors.toList());
```

This is probably the simplest example of using a `Collector`, but it works well to illustrate our point.

### Guava collections

For this article I'm going to use the specialized Collections provided by [Guava](https://github.com/google/guava) as an example. If you haven't checked out this library, I highly recommend it for the custom collection types (e.g. `Multimaps` and `BiMaps`), truly [Immutable](https://github.com/google/guava/wiki/ImmutableCollectionsExplained) collection types and it's many other features.

The important thing to know about Guava for this article, is that it provides a type called [ImmutableList](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableList.html). ImmutableList is a truly immutable collection that can never be modified directly and does not store a view to an otherwise accessible collection (more on that a bit later).

## Writing your own Collector

### Collecting to an ImmutableList: Directly

The `ImmutableList` javadoc states that it can be created directly, simply by giving it the elements in a collection. So if we want to collect to an `ImmutableList` we can just do that right? It can't be very hard given our earlier example.

```java
List<String> names = persons.stream()
                            .map(Person::getName)
                            .collect(Collectors.toList());
ImmutableList<String> immutableNames = ImmutableList.copyOf(names);
```

And that works fine! ...for the first, second, and maybe even 5th time. But at the end of the day, I don't want to be writing that extra line every time I want to create an `ImmutableList` from a `Stream`. Can't we just pack it into a `Collector` directly?

### Collecting using collectingAndThen

As you might imagine, the answer is yes. We can easily create a separate Collector by using the `Collectors.collectingAndThen(Collector, Function)` method provided by the `Collectors` class. So what does that look like?

```java
ImmutableList<String> immutableNames = persons  .stream()
                                                .map(Person::getName)
                                                .collect(Collectors.collectingAndThen(Collectors.toList(), ImmutableList::copyOf);
```

So that works, but I personally like the pattern provided by the JDK of creating a method that creates the collector. So we can create something like

```java

class CustomCollectors {
    
    public static <T> Collector<T, ?, ImmutableList<T>> toImmutableList() {
        return Collectors.collectingAndThen(Collectors.toList(), ImmutableList::copyOf);
    }
}

//...
ImmutableList<String> immutableNames = persons  .stream()
                                                .map(Person::getName)
                                                .collect(CustomCollectors.toImmutableList());
```

and that's great! So we use it throughout all of our code base... and then we realize, wait we're creating a separate Collection and copying it? Why? Can't we just create it directly?

### Anatomy of a Collector

`collectingAndThen` can be great for simple transformations, but when you want to do some more complex `Collector` creation the place to turn is directly in the `Collector` interface. Each `Collector` consists of the following elements:

* `Supplier`: This is a parameter, not surprisingly, of type [Supplier](https://docs.oracle.com/javase/8/docs/api/java/util/function/Supplier.html). It creates the *intermediate* storage type of the Collector. The `Collector` interface requires that this returns a new, clean instance of the intermediate data type. So usually you'll want to pass something like `HashSet::new`. Additionally, this can be called multiple times per collection operation, when working with Parallel Streams.

* `Accumulator`: This is a parameter of type [BiConsumer](https://docs.oracle.com/javase/8/docs/api/java/util/function/BiConsumer.html)  that takes the elements of the stream, and integrates them into the intermediate data type. For example, if you were working with a `Collection` you would likely want to pass `Collection::add` so the elements are added to the intermediate `Collection`.

* `Combiner`: This is a parameter of type [BinaryOperator](https://docs.oracle.com/javase/8/docs/api/java/util/function/BinaryOperator.html) that takes two intermediate containers and combines the elements of them into a single container. For a `Collection` this could be something like:
`(left, right) -> { left.addAll(right); return left; }`

* `Finisher (Optional)`: This is a parameter of type [Function](https://docs.oracle.com/javase/8/docs/api/java/util/function/BinaryOperator.html) that transforms the intermediate data type into the final data type. If the final data type is the same as the intermediate, then this can simply be not given.

* `Characteristics`: These describe optimizations to how the `Collector` operates.
    * [CONCURRENT](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collector.Characteristics.html#CONCURRENT) States that the intermediate data type can have the Accumulator called by multiple threads without concurrency issues.
    * [IDENTITY_FINISH](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collector.Characteristics.html#IDENTITY_FINISH) States that the Finisher operation is an Identity function, and can be skipped.
    * [UNORDERED](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collector.Characteristics.html#UNORDERED) States that the collection operation does not preserve element ordering. (e.g. collecting to a set instead of a list would give this characteristic)

Whew! That was a lot to take in. So how do we use it now? The `Collector` class provides two [factory](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collector.html#of-java.util.function.Supplier-java.util.function.BiConsumer-java.util.function.BinaryOperator-java.util.function.Function-java.util.stream.Collector.Characteristics...-) [methods](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collector.html#of-java.util.function.Supplier-java.util.function.BiConsumer-java.util.function.BinaryOperator-java.util.stream.Collector.Characteristics...-) for constructing a Collector, so unless you *really* want to, you don't need to implement the interface yourself.

### Putting it all together: The ImmutableList standalone Collector

So back to our `ImmutableList`, one thing I neglected to mention before is that it provides a [Builder](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableList.html#builder()) method and class. This class can be used to construct our elements into a `Builder` before finishing by calling `build()`. If this sounds a lot like elements of our `Collector`, then your head is in the right place for the next part.

```java
public static <T> Collector<T, ?, ImmutableList<T>> toImmutableList() {
	Supplier<ImmutableList.Builder<T>> supplier = ImmutableList.Builder::new;
	BiConsumer<ImmutableList.Builder<T>, T> accumulator = ImmutableList.Builder::add;
	BinaryOperator<ImmutableList.Builder<T>> combiner = (left, right) -> left.addAll(right.build());
	Function<ImmutableList.Builder<T>, ImmutableList<T>> finisher = ImmutableList.Builder::build;
	return Collector.of(supplier, accumulator, combiner, finisher /* No characteristics for this one */);
}
```

Here we use an `ImmutableList.Builder` as our intermediate object, we create it using the `supplier`, we then add new elements to it using `ImmutableList.Builder.add(T)`, next we combine them by adding the result of building from one list to the other. Finally, we finish by calling `build()` on our builder, and returning it.

## Conclusion

The `Collector` APIs can seem overwhelming at first, but when you sit down and break down your problem into whats my result and how do i get my elements to there it becomes much simpler. Don't forget that applying a post transform to a `List` or `Set`, via `Collectors.collectingAndThen` can often be the simplest way to reach this goal.

If you're interested in more `Collectors` for the Guava collection types, I recommend looking at [GuavaCollectors](https://gist.github.com/APaloski/dc81144b2c9ba48477e5).

