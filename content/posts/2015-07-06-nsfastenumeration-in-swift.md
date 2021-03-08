---
title: NSFastEnumeration in Swift
date: 2014-10-01T11:42:36+02:00
images:
toc: false
draft: false
tags:
  - swift
  - ios
  - NSFastEnumeration
  - NSFileManager
---

I spent the past few weeks digging into Swift while working on PSPDFKit. Today I was trying to enumerate a directory recursively.
I resorted to `NSFileManager` and its method `enumeratorAtURL:includingPropertiesForKeys:options:errorHandler:`.
It returns an `NSDirectoryEnumerator` object which supports `NSFastEnumeration`. That means you can use the for-in loop in Objective-C.
Without thinking, I wrote a for-in loop in Swift. Not so fast.

> Type ‘NSDirectoryEnumerator’ does not conform to protocol ‘SequenceType’.

Apple should have supported this. So I spent some time browsing trough the Swift headers and with the help of generics
I wrote a custom substruct (?) that conforms to the `GeneratorType` protocol and can be initialized with an object of type `NSEnumerator`.

{% codeblock lang:swift %}
public struct GenericGenerator<T> : GeneratorType, SequenceType {
  let enumerator: NSEnumerator
  
  init(_ enumerator: NSEnumerator) {
    self.enumerator = enumerator
  }
 
  mutating public func next() -> T? {
    return self.enumerator.nextObject() as T?
  }
 
  public func generate() -> GenericGenerator<T> {
    return GenericGenerator<T>(self.enumerator)
  }
}
{% endcodeblock %}

In my case I was dealing with `NSDirectoryEnumerator` which iterates trough `NSURL` objects. So I wrote an extension to extend it with support for `SequenceType`.
The nice thing is that using for-in in Swift now provides you with `NSURL` objects!

{% codeblock lang:swift %}
extension NSDirectoryEnumerator : SequenceType {
  public func generate() -> GenericGenerator<NSURL> {
    return GenericGenerator<NSURL>(self)
  }
}
{% endcodeblock %}

This generic can now be used to extend any kind of `NSEnumerator` so you can enjoy clean for-in loops in Swift.
