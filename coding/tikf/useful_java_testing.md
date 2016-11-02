---
layout: page
title: Useful Java Unit Testing Classes
permalink: /pages/useful_java_testing
---

Unit testing things (Mostly JUnit 4)
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
  public void square_isEqualToParamTimesParam(int param) {
    MyCalculator calculator = new MyCalculator();
    
    int result = calculator.square(param);
    
    assertEquals(result, param * param);
  }

}
```


* JUnit Source: [Since JUnit 4 (Unknown minor version)](http://junit.org/junit4/javadoc/4.12/org/junit/experimental/theories/Theories.html)
* Superseded in JUnit 5 by dynamic tests and parameter suppliers. See [JUnit 5 docs](http://junit.org/junit5/) for more information.

