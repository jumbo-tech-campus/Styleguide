# Unit test guidelines

Writing unit tests can be done in many ways, but all of them have some default rules to adhere to. Further more we recommend using [Quick](https://github.com/Quick/Quick) & [Nimble](https://github.com/Quick/Nimble) for unit tests in Swift, with writing mocks as described below.

## General guidelines

- Never use a shared instance of a class in any part of a unit test, i.e.
```swift
UserDefaults.standard
UIApplication.shared
```
- Never use any form of waiting directly in the tests
```swift
sleep()
NSRunLoop.current
while (value != expected) { }
```

To wait for code to finish use `XCTestExpectation` when writing a `XCTest` or `expectEventually` when writing a spec using `Quick & Nimble`

## Quick & Nimble guidelines

Quick & Nimble provide 5 important DSL keywords, below follow which should be used in different scenarios.
Each of these keywords takes a string, which is a description of the block to follow.
All of these describing strings will be used by the framework to generate the test function, i.e
```
describe("MyClass") {
    describe("execute the function") {
        it("sets the variable") {
        }
    }
}
```
Would become `MyClass__executing_the_function__sets_the_variable`. 

These test functions read in one go
 - Which object is being tested
 - Which functionality is being tested
 - What is expected

To make sure these test functions all read in a similar fassion Always use **present tense** for DSL keyword descriptions.
You can check this by replacing the first `describe` keyword with 'when':

`when` MyClass execute the function it sets the variable


### Describe

Use `describe` when **starting a spec function**. This is also the place where the `sut` is defined.
The `sut` should always be the first variable defined in a Spec.

```swift
class BoolSpec: QuickSpec {
    override func spec() {
        describe("Bool") {
            var sut: Bool!

            ...
        }
    }
}

```

The other use of describe is to describe **a piece functionality within the class** under test. 

**Rule of thumb**: When adding a `beforeEach` statement in a `describe` it should only prepare for `context` calls to follow. Don't execute any code on the class under test. 

For example a `negate()` function within a `Bool` class;

```swift
class BoolSpec: QuickSpec {
    override func spec() {
        describe("Bool") {
            var sut: Bool!

            context("with a false value") {
                beforeEach {
                    sut = false

                    ...
                }

                afterEach {
                    ...

                    sut = nil
                }
            }
        }
    }
}

```

### Context

Use `context` to define the different flows of the a certain functionality within the `describe` closure, for example:

```swift
class BoolSpec: QuickSpec {
    override func spec() {
        describe("Bool") {
            var sut: Bool!

            describe("negating the value") {
                context("negating a true value") {
                    ...
                }

                context("negating a false value") {
                    ...
                }
            }
        }
    }
}

```

### BeforeEach

`beforeEach` should only be used within `describe` or `context` closures. Use beforeEach for;
1. Preparing mocks & test input data
2. Creating or modifying the `sut`
3. Execute the code under test (as defined in the `describe` closure)

```swift
class BoolSpec: QuickSpec {
    override func spec() {
        describe("Bool") {
            var sut: Bool!

            describe("negating the value") {
                context("negating a true value") {
                    beforeEach {
                        sut = true
                        sut.negate()
                    }
                }

                context("negating a false value") {
                    beforeEach {
                        sut = false
                        sut.negate()
                    }
                }
            }
        }
    }
}

```

### It

`it` is where assertions are executed in the form of an `expect()` call.

**Rule of thumb**: Only execute one `expect` call per `it` closure

```swift
class BoolSpec: QuickSpec {
    override func spec() {
        describe("Bool") {
            var sut: Bool!

            describe("negating the value") {
                context("negating a true value") {
                    beforeEach {
                        sut = true
                        sut.negate()
                    }

                    it("sets the value to false") {
                        expect(sut) == false
                    }
                }

                context("negating a false value") {
                    beforeEach {
                        sut = false
                        sut.negate()
                    }

                    it("sets the value to true") {
                        expect(sut) == true
                    }
                }
            }
        }
    }
}

```

### AfterEach

`afterEach` should only be used within `describe` or `context` closures. Use afterEach for;
1. Nullify all variables instantiated in the current `describe` / `context`
2. Any other cleanup your test specifically need

**Note**: Setting all variables to `nil` technically isn't necessary, but it's a small effort to add them and have certainty all values are reset correctly.


```swift
class BoolSpec: QuickSpec {
    override func spec() {
        describe("Bool") {
            var sut: Bool!

            afterEach {
                sut = nil
            }

            describe("negating the value") {
                context("negating a true value") {
                    beforeEach {
                        sut = true
                        sut.negate()
                    }

                    it("sets the value to false") {
                        expect(sut) == false
                    }
                }

                context("negating a false value") {
                    beforeEach {
                        sut = false
                        sut.negate()
                    }

                    it("sets the value to true") {
                        expect(sut) == true
                    }
                }
            }
        }
    }
}

```

## Writing Swift mocks

### Terminology

When it comes to writing mocks in Swift theres many ways for developers to do this, but we can never agree upon the naming.

Imagine a protocol `MyProtocol`, some would create `MyProtocolMock` where others use `MyProtocolStub`. Which of them is correct? Neither, see below.

#### Mock

Mocks are objects that register calls they receive.
In test assertion we can verify on Mocks that all expected actions were performed.
![](https://cdn-images-1.medium.com/max/1600/0*k7mwTF60slyMxRlm.png)

#### Stub

Stub is an object that holds predefined data and uses it to answer calls during tests. It is used when we cannot or don’t want to involve objects that would answer with real data or have undesirable side effects.
![](https://cdn-images-1.medium.com/max/1600/0*KdpZaEVy6GNnrUpB.png)

#### Fake

Fakes are objects that have working implementations, but not same as production one. Usually they take some shortcut and have simplified version of production code.
![](https://cdn-images-1.medium.com/max/1600/0*snrzYwepyaPu3uC9.png)

### Example mock from protocol

As you can read, neither `...Stub` nor `...Mock` would cover the full load of a object we want to write. Our test-objects should be able to mimic a protocol by both setting stubs and registering received method calls. We don't want multiple files for a single Protocol in order to both mock & stub them, thus we want one object which can both stub and mock a protocol.

The `...Double` suffix is coined as wel for this goal, derived from body-double, but this term on it's own only adds confusion. In the end calling your 'minimal' implementation of a class a `Mock` feels most natural, where a mock is build up from two parts;
 - Captures: Basically the Mocks as described above. They register the captured method calls
 - Stubs: Stubs as described above
 
 Both captures and stubs should be stored in a struct which can be instantiated without passing parameters. 
 `Stubs` speaks for itself. It contains stubs for both variables & method calls. Stubs should either be defined as an optional or with a default value.
 Within `Captures` nested structs should be defined for each method to capture, in these nested structs the parameters of a function call should be stored. The definition of each variable within `Captures` should be optional. This way by checking the `Captures` variable for nil you can see wether or not a function is called, plus you can check the individual parameters passed to the call. 

This way we have one object to wrap a full protocol in, having both mocking and stubbing capabilities.

```swift
protocol ExampleProtocol {
    var testVar: String { get }

    func doFunction(var: String)
    func returnFunction(number: Int, otherNumber: Int) -> String
}
```

```swift
class ExampleProtocolMock {
    var Stubs {
        var testVar = String()
        var returnFunction = String()
    }

    var Captures {
        var doFunction: DoFunction?
        var returnFunction: ReturnFunction?

        struct DoFunction {
            let var: String
        }

        struct ReturnFunction {
            let number: Int
            let otherNumber: Int
        }
    }

    var stubs = Stubs()
    var captures = Captures()
}

extension ExampleProtocolDouble: ExampleProtocol {
    var testVar: String { 
        return stubs.testVar
    }

    func doFunction(var: String) {
        captures.doFunction = Captures.DoFunction(var: var)
    }

    func returnFunction(number: Int, otherNumber: Int) -> String {
        captures.returnFunction = Captures.ReturnFunction(number: number, otherNumber: otherNumber)

        return stubs.returnFunction
    }
}

```
