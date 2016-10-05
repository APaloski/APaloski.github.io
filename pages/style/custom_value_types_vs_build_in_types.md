---
layout: page
title: Code style - Custom Value Types vs Built In Types
permalink: /pages/style/custom-value-types-vs-built-ins
---

### Recommendation
If you can, prefer a custom data type that better represents the actual type of data that you are representing. This has numerous benefits, including:

1. It benefits the idea of Readability is more important than Writability (see: https://blogs.msdn.microsoft.com/oldnewthing/20070406-00/?p=27343/). 

2. It allows you to pack validations and operations into one spot. 
   
   If your class is a Uri, you may want to allow access to the underlying portions of the decomposed String, but if it is simply stored as a String you need a static utiity method to break it down. In addition, the Objects creation can require validation at the time of creation, preventing a bad Uri from ever being created.

3. It gives you type safety to prevent silly mistakes.
   
   Have you ever hit code like: ```void postToUrl(String url, String message)```? If you have then you have probably accidentally called it as ```postToUrl(getMessage(), getTargetUrl())``` and had a runtime failure. Now, if we have the signature of  ```void postToUrl(Uri url, String message)``` then this is a compile error.

### Example from a recent project
Recently on a project, I ran into a case of needing to represent a UPC code within a large model object. The first approach I thought of to this was to store a String in the model, that had some issues associated to it however. 

Pros:

1. The String will need to be revalidated every time it is set in the that model, which is fine, but then if we also pass this UPC around elsewhere it's going to need to have that same validation added in each of those places.
2. A String is not a good indicator of that _actual_ type of the object. We can call any field/parameter/local ```upc```, but in the end anyone can place anything into that location without validating.

Cons:

1. Using a String reduces complexity and, in theory, reduces the amount of classes to handle.

The alternative to this is to create a small value object, which simply wraps the given String, and better represents it.

Pros:

1. Using a ```Upc``` data type, we can easily validate the String in one place, and then anyone taking out ```Upc``` would know that it is already validated.
2. Using a custom ```Upc``` type better represents the actual, underlying data type that we want. When you see ```Upc mProductCode``` it's precisely clear the type of it.

Cons:

1. A string that is validated as a Upc, but does not actually represent a real world Upc, could be passed around wrapped into this class causing confusion.

In the end, I decided that someone intentionally being malicious was in this case a risk worth taking, as the validation for a Upc is relatively strict.
