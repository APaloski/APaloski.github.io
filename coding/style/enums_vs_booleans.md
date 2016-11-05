---
layout: page
title: Code style - Enums vs Booleans
permalink: /coding/style/enum-vs-boolean
---

This is the first in a set of new blog posts where I record coding rules that I find and believe will improve code quality over time. The goal of this is to present each rule with a list of pros and cons to help decide on a case by case basis if it should be applied to a new piece of code.

## Enums should be preferred to booleans for parameter and return values.

When creating or modifying a function, often times we can run into wanting to change a small part of the algorithm. Many times the easiest way to do this is to have a simple switch in the form of a boolean flag. This little flag allows us to change that one small part, but reuse the rest of the code in multiple places and we move on. The problem is, that this often times creates huge ambiguities at the reading site (remember: [code is read much more often than it is written](https://blogs.msdn.microsoft.com/oldnewthing/20070406-00/?p=27343/)). The difference between  ```java Sorter.sort(myList, true)``` and ```java Sorter.sort(myList, SortOrder.FORWARD)``` is the difference between needing to read the javadoc and not, especially when it's contained in an otherwise complex function.

### A world without Enums

Take the following for instance:

```java
  List<Document> findDocuments(boolean includeDraftDocuments) {
    //...
  }
```

This function is simple enough from the writers point of view, simply say if you want drafts or not, but when looked at from the callers point of view it becomes

```java
  List<Document> myWantedDocuments = findDocuments(false);
```

can you tell me at a glance what that flag means?

It gets worse when you include the fact that APIs tend to get more complex as they evolve. Lets say someone also wanted to add in functionality to be able to get documents that are soft deleted by marking them as deleted, but not actually removing the data. They decide to reuse the existing method, everyone already uses it! ...but they have to add another flag for soft deletes.

```java
  List<Document> findDocuments(boolean includeDraftDocuments,
                               boolean includeSoftDeletes) {
    //...
  }
```

This function would be fine from the point of view when reading the documentation, but often times that's the ideal, not the real world. Realistically we want to be able to read this function clearly in a large block of code. Tell me, if you saw the following in a large block of code would you know what false and true were?

```java
  List<Document> myWantedDocuments = findDocuments(false, true);
```

This also leaves us open to other vulnerabilities, because we are not enforcing type safety on the arguments. Once again, could you catch the bug here at a glance reading through a function?

```java
  List<Document> myWantedDocuments = findDocuments(mIncludeSoftDeletes, mIncludeDrafts);
```
#### Replacing booleans with Enums

Lets now take the alternative (suggested) route, and use an enum instead. By using an enum, we instantly get more type safety and prevent the use of an invalid parameter (such as using a boolean variable intended for use into another parameter). So lets go back to the original example, by including a new ```java DocumentType ``` enum, we can explicitly clarify the type of the first parameter and transform our code into 

```java
  List<Document> findDocuments(DocumentType toInclude) {
    //...
  }
 
  //... (in a later function)
  List<Document> myWantedDocuments = findDocuments(DocumentType.DRAFTS);
```

This also cascades into our potential future example, where we ran into issues of transposing arguments and loss of clarity, our future example then becomes 

```java
  List<Document> findDocuments(DocumentType toInclude, DocumentDeletedType deletionType) {
    //...
  }
 

  //Compile error, DocumentDeletionType cannot be cast to DocumentType
  List<Document> myWantedDocuments = findDocuments(mIncludeSoftDeletes, mIncludeDrafts);
  
  //Without members
  List<Document> myWantedDocuments = findDocuments(DocumentType.COMPLETED, DocumentDeletionType.NOT_DELETED);
```

This significantly improves the at-a-glance readability of the function call as well as preventing future bugs.

### Pros and Cons

#### Pros
1. Improves readability of the function call at the point of calling
2. Provides additional type safety over boolean constants 
3. Standardizes to one type things that may be used globally. Your DocumentType can also be returned from, and used in, other function calls.

#### Cons
1. Adds an additional .class file per 'type' of boolean flag you want to replace. This may be important in embeded and low resource systems such as Android, but has become less important over time.
2. Adds the possibility of Null Pointer Exceptions if you are not judicious about preventing null arguments.
3. May add complexity to the system, by having additional types in the system.

### Supporting Articles
* [The boolean trap](https://ariya.io/2011/08/hall-of-api-shame-boolean-trap)
* [Try to avoid having BOOL parameters](https://blogs.msdn.microsoft.com/oldnewthing/20060828-18/?p=29953)  
