---
title: Testing Throwable Methods in Swift 2
date: 2015-07-08T15:47:48+02:00
images:
toc: false
draft: false
tags:
    - swift
    - testing
---

With the addition of error handling to Swift 2, we got a way to deal with errors with the nice try/catch exception handling syntax from many languages,
but without the associated cost of stack unwinding.

As with anything new, some questions come to mind. How do we test methods that can throw errors? In Objective-C we had a few `XCTest` assert macros to help with this, like `XCTAssertThrows`. Swift 1 does not support exceptions, so these macros,
which get translated to Swift functions, are absent from the `XCTest` framework in Swift. I was expecting to find them in the Swift 2 version of the framework,
but they're missing.

So the first thing that comes to mind is to just wrap the asserts into do/catch blocks.

```swift
func testSomething() {
    do {
        try XCTAssert(parseSomething() == "result")
    } catch {
        XCTFail("Parsing failed.")
    }
}
```

And Xcode complains: _"Call can throw, but it is executed in a non-throwing autoclosure."_

Assert macros use the `@autoclosure` attribute, so everything you pass to them becomes a closure. A non-throwing closure.
So you need to first try to get the result and pass just that.

```swift
func testSomething() {
    do {
        let result = try parseSomething()
        XCTAssertEqual("result", result)
    } catch {
        XCTFail("Parsing failed.")
    }
}
```

This works. But, we hate repetition, don't we?

```swift
func XCTAssertThrows<T>(@autoclosure expression: () throws -> T, _ message: String = "", file: String = __FILE__, line: UInt = __LINE__) {
    do {
        try expression()
        XCTFail("No error to catch! - \(message)", file: file, line: line)
    } catch {
    }
}

func XCTAssertNoThrow<T>(@autoclosure expression: () throws -> T, _ message: String = "", file: String = __FILE__, line: UInt = __LINE__) {
    do {
        try expression()
    } catch let error {
        XCTFail("Caught error: \(error) - \(message)", file: file, line: line)
    }
}

func testSomething() {
    XCTAssertNoThrow(try parseSomething())
    XCTAssertThrow(try parseOtherThing())
}
```

Good. We can check if a method throws or not. But some methods return things and we would like to compare these things to other things.

```swift
func XCTAssertNoThrowEqual<T : Equatable>(@autoclosure expression1: () -> T, @autoclosure _ expression2: () throws -> T, _ message: String = "", file: String = __FILE__, line: UInt = __LINE__) {
    do {
        let result2 = try expression2()
        XCTAssertEqual(expression1, result2, message, file: file, line: line)
    } catch let error {
        XCTFail("Caught error: \(error) - \(message)", file: file, line: line)
    }
}

func testSomething() {
    XCTAssertNoThrowEqual("result", try parseSomething())
}
```

Ok, that's equality. But what about >, < and membership checks? Let's add a validator closure then.

```swift
func XCTAssertNoThrowValidateValue<T>(@autoclosure expression: () throws -> T, _ message: String = "", file: String = __FILE__, line: UInt = __LINE__, _ validator: (T) -> Bool) {
    do {
        let result = try expression()
        XCTAssert(validator(result), "Value validation failed - \(message)", file: file, line: line)
    } catch let error {
        XCTFail("Caught error: \(error) - \(message)", file: file, line: line)
    }
}

func testSomething() {
    XCTAssertNoThrowValidateValue(try parseSomething()) { $0 > 5 }
}
```

This covers my use cases for now!