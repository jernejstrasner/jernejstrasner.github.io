---
title: Singleton Pattern in Swift
date: 2014-06-07T14:20:26+02:00
images:
toc: false
draft: false
tags:
    - swift
    - ios
    - singleton
---

Like just about any other iOS developer, I was caught completely by surprise when Apple announced Swift.
I was expecting modest improvements to Objective-C, maybe even an Objective-C 3.0 release, but nothing as wild as a new language.
But I got excited, really excited, when I watched the state of the union video (no, I was not one of the lucky WWDC ticket lottery winners).
I started reading the Swift book right away. _“I’m going to read this trough first,”_ I said to myself, _“and hack later.”_
Well, guess what? I started hacking after the introduction chapter. After I alternated between reading and hacking for a while,
I needed a project to work on. And so I chose to port [JSImageLoader](https://github.com/jernejstrasner/JSImageLoader) to Swift and see how it goes.

__Update__: See the updated code for Swift 1.2 at the bottom of this post.

The first thing I did was to outline classes and add properties. This was easy. The second step was to implement initializers.
Trivial, if it’s not a singleton. My first idea was to write it the exact same way I would do it in Objective-C, using dispatch_once.
I came across a major difference right away; static variables can only be declared inside enums and structs.
Luckily structs can be defined inside methods and, like in this case, property getters. So I came up with the code below:

```swift
class MyClass {
    class var sharedInstance: MyClass {
        struct Singleton {
            static var token : dispatch_once_t = 0
            static var instance : MyClass?
        }
        dispatch_once(&Singleton.token) {
            Singleton.instance = MyClass()
        }
        return Singleton.instance!
    }
}
```

Works, great!

Then I read more of the book, watched some more WWDC videos and browsed more of the Apple developer forums.
Then I found this post by Joe Groff (@jckarter) who works on the Swift compiler:

> Global and static properties are already dispatch_once’d, so there’s really no need for this. You can simply define the singleton instance as a global variable or static let property of a struct.

This is getting better and better! He says there are two options, one with a global variable and one with a static let as the property of a struct.
I’m not a fan of polluting the global space too much so I chose the latter:

```swift
class MyClass {
    class var sharedInstance: MyClass {
        struct Singleton {
            static let instance = MyClass()
        }
        return Singleton.instance
    }
}
```

Much nicer!

While I do agree that this could be made even nicer by having a private static class let property,
I think that it’s great that global variables and static let properties are wrapped in dispatch_once.
I’m sticking to this implementation for now.

__Update__: Swift 1.2 added support for static properties on classes so now a singleton can be simply initialized when declaring the property. No more embedded structs!

```swift
class MyClass {
    static let sharedInstance = MyClass()
}
```

That’s it!
