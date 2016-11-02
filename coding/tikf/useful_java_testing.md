---
layout: page
title: Useful Java Unit Testing Classes
permalink: /pages/useful_java_testing
---

Unit testing classes
------

### JUnit Theories!

This isn't a class so much as a "better" way to write tests. JUnit theories lets you use any combination of data points to test all different crazy scenarios in your unit tests. With a few simple methods and annotations your square root method can easily test a large subset of all positive numbers for one case, all negatives for another, and all edge cases for a third. It's marked as Experimental (but has been forever), and it should give a good basis for going into dynamic tests and parameterized tests in JUnit 5.

#### Examples
```java
//A very simple, contrived example. Theroies are much more powerful than shown here
@RunWith(Theories.class)
public class MyCalculatorTest {

  @DataPoints
  public static int[] getPossibleValues() {
    return IntStream.range(-10, 10).toArray();
  }
  
  //This will be called for all values provided by getPossibleValues
  @Theory
  public void square_isEqualToParamTimesParam(final int toSquare) {
    MyCalculator calculator = new MyCalculator();
    
    int result = calculator.square(param);
    
    assertEquals(result, param * param);
  }

}
```

* JUnit Source: [Available in 4.12](http://junit.org/junit4/javadoc/4.12/org/junit/experimental/theories/Theories.html)
* Superseded in JUnit 5 by dynamic tests and parameter suppliers. See [JUnit 5 docs](http://junit.org/junit5/) for more information.


### ParameterSuppliedBy
This class allows you to create a custom annotation for use with JUnit Theories where the arguments to that annotation can be used to determine what values are passed into the theory.

#### Examples
```java
@Retention(RetentionPolicy.RUNTIME)
@ParametersSuppliedBy(MySupplier.class)
public @interface IntRange { 
  int min() default Integer.MIN_VALUE;
  int max() default Integer.MAX_VALUE;
}

public class MySupplier extends ParameterSupplier {
  @Override
  public List<PotentialAssignment> getValueSources(final ParameterSignature sig) throws Throwable {
    IntRange myTheory = sig.getAnnotation(IntRange.class);
    //Use any parameters on the annotation
    //Generate any data
    return IntStream.range(myTheory.min(), myTheory.max())
                    .mapToObj(i -> PotentialData.forValue("", i))
                    .toArray(PotentialData[]::new);
  }
}

//In my theory class from the Theories example...

@Theory
public void square_isEqualToParamTimesParam(@IntRange(min = -10, max = 10) final int toSquare) {
  //Same implementation
}

```

* JUnit source: [Available in 4.12](http://junit.org/junit4/javadoc/4.12/org/junit/experimental/theories/ParametersSuppliedBy.html)

**See Also:** [JUnit Blog](http://www.junit-blog.com/search/label/parameterSuppliedBy)

Unit testing libraries
------
