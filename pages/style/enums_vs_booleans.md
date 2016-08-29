---
layout: page
title: Code style - Enums vs Booleans
permalink: /pages/style/enum-vs-boolean
---

**Rule**: When creating a method, always prefer an enum over a boolean as a paramter.

When creating a function, often times using shared code with a simple switch seems like the best choice. Everything is the same except one little flag which will alternate between one of two different states. The problem is, that this often times creates huge ambiguities at the reading site (remember: [code is read much more often than it is written](https://blogs.msdn.microsoft.com/oldnewthing/20070406-00/?p=27343/)). 

Take the following for instance:
```java
  List<Document> findDocuments(boolean includeDraftDocuments) {
  
  }
```

This function is simple enough from the writers point of view, simply say if you want drafts or not, but when looked at from the callers point of view it becomes

```java
  List<Document> myWantedDocuments = findDocuments(false);
```

can you tell me at a glance what that flag means?

It gets worse when you include the fact that APIs tend to get more complex as they evolve. Lets say someone also wanted to add in functionality to be able to get documents that are soft deleted by marking them as deleted, but not actually removing the data. They decide to reuse the existing method, everyone already uses it! ...but they have to add another flag for soft deletes. Even worse, what happens if we add a stage between draft and final?

```java
  List<Document> findDocuments(boolean includeDraftDocuments,
                               boolean includeSoftDeletes) {
  
  }
```

But then the caller sees 
```java
  List<Document> myWantedDocuments = findDocuments(false, true);
```

...and then the caller does
```java
  List<Document> myWantedDocuments = findDocuments(mIncludeSoftDeletes, mIncludeDrafts);
```
But, if we instead have the declaration 
```java
  List<Document> findDocuments(DocumentType toInclude) {
  
  }
```
then we can have
```java
   List<Document> myWantedDocuments = findDocuments(DocumentType.DRAFTS);
```
and our future looks less bleak
```java
  //Compile error, DocumentDeletionType cannot be cast to DocumentType
  List<Document> myWantedDocuments = findDocuments(mIncludeSoftDeletes, mIncludeDrafts);
  
  //Without members
  List<Document> myWantedDocuments = findDocuments(DocumentType.COMPLETED, DocumentDeletionType.NOT_DELETED);
```

**Exception**: A simple setter on a boolean property, though that property being a boolean should likely be reconsidered in and of itself.
  
Supporting Articles: [The boolean trap](https://ariya.io/2011/08/hall-of-api-shame-boolean-trap), [Try to avoid having BOOL paramters](https://blogs.msdn.microsoft.com/oldnewthing/20060828-18/?p=29953)  
