# Swift Style Guide

This style guide follows the [Apple's API Design Guidelines](https://swift.org/documentation/api-design-guidelines/): be sure to read them.

This guide will describe the swift style guide with a top-to-bottom approach, starting from the file naming conventions and arriving to code style conventions.

A special chapter is reserved from _special practise_ that could be implemented to help us in our daily work.

This guide was last updated on October 11, 2018 and it's based on the following sources:

* [Apple's API Design Guidelines](https://swift.org/documentation/api-design-guidelines/)
* [LinkedIn Guidelines](https://github.com/linkedin/swift-style-guide/blob/master/README.md)
* [RayWenderlich Guidelines](https://github.com/raywenderlich/swift-style-guide)

## Table Of Contents

- [Swift Style Guide](#swift-style-guide)
  * [Table Of Contents](#table-of-contents)
  * [1. Source file](#1-source-file)
    + [1.1 File names](#11-file-names)
    + [1.2 Characters](#12-characters)
  * [2. File structure](#2-file-structure)
    + [2.1 Import statements](#21-import-statements)
    + [2.2 Enums, Protocols and Classes](#22-enums--protocols-and-classes)
    + [2.3 Variables and Functions](#23-variables-and-functions)
  * [3. Naming](#3-naming)
    + [3.1 General](#31-general)
    + [3.2 Descriptive and unambiguous names](#32-descriptive-and-unambiguous-names)
    + [3.3 Make an English phrase](#33-make-an-english-phrase)
    + [3.4 Initializer](#34-initializer)
    + [3.5 Protocols](#35-protocols)
    + [3.6 Delegates and DataSource](#36-delegates-and-datasource)
    + [3.7 Generics](#37-generics)
  * [4. Coding Style](#4-coding-style)
    + [4.1 General formatting](#41-general-formatting)
    + [4.2 Error Handling](#42-error-handling)
    + [4.3 Guard Statements](#43-guard-statements)
    + [4.4 Access Modifiers](#44-access-modifiers)
    + [4.5 Custom Operators](#45-custom-operators)
    + [4.6 Switch Statements and enums](#46-switch-statements-and-enums)
    + [4.7 Optionals](#47-optionals)
    + [4.8 Properties](#48-properties)
    + [4.9 Closures](#49-closures)
    + [4.10 Arrays](#410-arrays)

## 1. Source file

### 1.1 File names

All Swift source files end with the extension **.swift**.

In general, the name of a source file should best describe the primary entity that it contains. Based on the entity itself, the name of the file changes:

* A file containing a single type `MyClass` is named **MyClass.swift**.
* A file containing a single class `MyClass` and some top-level helper methods is named **MyClass.swift**.
* A file containing a single extension to a class `MyClass` that adds conformance to a protocol `MyProtocol` is named **MyClass+MyProtocol.swift**.
* A file containing multiple extensions or constants to a class `MyClass` that adds conformances, nested types, or other functionality can be named in a generic way like **MyType+Additions.swift**.

### 1.2 Characters

* Source files are encoded in UTF-8.
* Tab characters are **not** used for indentation.
* File should start without any empty lines.
* File should start without a copyright notice.
* File should end with a single empty line.


## 2. File structure

### 2.1 Import statements

Import statements are located at the top of a source file. They are grouped, and each group has one blank line between it and the next group:

1. Modules imported with `@testable` (only present in test sources)
2. Module/submodule imports not under test
3. Individual declaration imports (class, enum, func, struct, var)

A source file imports exactly the top-level modules that it needs, nothing more and nothing less. If a source file uses definitions from both UIKit and Foundation, it imports both explicitly and it does not rely on the fact that some Apple frameworks transitively import others as an implementation detail.

Imports of whole modules are preferred to imports of individual declarations or submodules.

### 2.2 Enums, Protocols and Classes

A file should define a single type. For the dependency injection pattern, a class file should define a protocol type and confirm to it.

If a file uses an `Enum` and it is not private, it should be added to a new file.

```swift
// FILE EyesColor.swift
enum EyesColor {
    case brown
    case green
    case blue
}
```

```swift
// FILE Person.swift
protocol PersonType {
    var name: String { get }
    var eyes: EyesColor { get }
    func talk()
}

class Person: PersonType {
    var name: String

    init(name: String, eyes: EyesColor) {
        self.name = name
        self.eyes = eyes
    }

    func talk() {
        print("Hello, I'm \(name)!")
    }
}
```

Exceptions are allowed when it makes sense to include multiple related types in a single file.

For example:

1. A class and its helper methods
2. A class and *typealiases* used in the class itself

### 2.3 Variables and Functions

The ordering of variables and functions in a source file can have a great effect on readability. However, there is no single correct recipe for how to do it: different files may order their contents in different ways.

What is important is that **each file and type uses some logical order**, which is easy to understand when reading the code.

When deciding on the logical order of members, please use `// MARK: -` to group functions related to the same functionality. These comments are also interpreted by Xcode and provide bookmarks in the source window’s navigation bar.

Some rules apply:

* Attributes to the **top**, following this order:
    1. Type aliasses
    2. Static properties
    3. private IBOutlets
    4. public IBOutlets (try to avoid them at all)
    5. Private variables
    6. Public variables

* Functions are defined **after** properties.
* `Delegates` and `DataSource` functions should be defined in a separate `extension`

  ```swift
    class MyViewController: UIViewController {

        // MARK: - Static variables

        static let identifier = "MyViewControllerIdentifier"

        // MARK: - IBOutlets variables

        @IBOutlet private var headerView: UIView!
        @IBOutlet private var tableView: UITableView!
        @IBOutlet private var footerView: UIView!

        // MARK: - Private variables

        private let title = "My Personal List"
        private var data: [String] = []

        // MARK: - View controller lifecycle

        override func viewDidLoad() {
            // ...
        }

        override func viewWillAppear(_ animated: Bool) {
            // ...
        }
    }

    extension MyViewController: UITableViewDelegate {
        // ...
    }
  ```


## 3. Naming

**Clarity is more important than brevity**. Although Swift code can be compact, naming used in the code should be clear enough to be understood without any doubt by other members of the team.

Golden rule:

> If you are having trouble describing your API’s functionality in simple terms, **you may have designed the wrong API**.

### 3.1 General

* There is no need for Objective-C style prefixing in Swift: use `MyClass` instead of `JMBMyClass`
* Use **PascalCase** with initial capital case letter for type names (e.g. *struct*, *enum*, *class*, *typedef*, *generics*, *associatedtype*, etc.).
* Use **camelCase** with initial lower case letter for everything else (e.g. *functions*, *property*, *constant*, *variable*, *argument names*, *enum cases*, etc.).
* When dealing with an acronym or other name that is usually written in all caps, use all caps in any names that use this in code. The exception is if this word is at the start of a name that needs to start with lowercase - in this case, use all lowercase for the acronym.
* Prefer using *ID* rather than *Id*.

    ```swift
    // "HTML" is at the start of a constant name, so we use lowercase "html"
    let htmlBodyContent: String = "<p>Hello, World!</p>"

    // Prefer using ID to Id
    let profileID: Int = 1

    // Prefer URLFinder to UrlFinder
    class URLFinder {
        /* ... */
    }
    ```

### 3.2 Descriptive and unambiguous names

* Do not abbreviate, use shortened names, or single letter names.

    ```swift
    // PREFERRED
    class RoundAnimatingButton: UIButton {
        let animationDuration: NSTimeInterval

        func startAnimating() {
            let firstSubview = subviews.first
        }
    }

    // NOT PREFERRED
    class RoundAnimating: UIButton {
        let aniDur: NSTimeInterval

        func srtAnimating() {
            let v = subviews.first
        }
    }
    ```

* Include type information in constant or variable names when it is not obvious otherwise.

    ```swift
    // PREFERRED
    class ConnectionTableViewCell: UITableViewCell {

        // make sure to specify the outlet type in the property name.
        @IBOutlet weak var submitButton: UIButton!
        @IBOutlet weak var emailTextField: UITextField!
        @IBOutlet weak var nameLabel: UILabel!

	     // make sure to add the type in the property name
        let personImageView: UIImageView
        let popupViewController: UIViewController

        // it's obvious that it's a time interval
        let animationDuration: TimeInterval

        // it's obvious that it's a string
        let firstName: String
    }

    // NOT PREFERRED

    // use the complete type, not just a part
    class ConnectionCell: UITableViewCell {

        // this isn't a `UIImage`
        let personImage: UIImageView

        // this isn't a `String`, so it should be `textLabel`
        let text: UILabel

        // `animation` is not clearly a time interval
        let animation: TimeInterval

        // this is not obviously a `String` use `transitionString` instead
        let transition: String

        // this is a view controller - not a view
        let popupView: UIViewController

        // don't use abbreviations
        let popupVC: UIViewController

        // it's not just a `UIViewController`, it's a *Table* View Controller
        let popupViewController: UITableViewController
    }
    ```

### 3.3 Make an English phrase

* Prefer method and function names that make use sites form **grammatical English phrases**.

```swift
// PREFERRED
x.insert(y, at: z)          “x, insert y at z”
x.subViews(havingColor: y)  “x's subviews having color y”
x.capitalizingNouns()       “x, capitalizing nouns”

// NOT PREFERRED
x.insert(y, position: z)
x.subViews(color: y)
x.nounCapitalize()
```

* The first argument to initializer and factory methods calls should not form a phrase starting with the base name

    ```swift
    // PREFERRED
    let foreground = Color(red: 32, green: 64, blue: 128)
    let newPart = factory.makeWidget(gears: 42, spindles: 14)
    let ref = Link(target: destination)

    // NOT PREFERRED
    let foreground = Color(havingRGBValuesRed: 32, green: 64, andBlue: 128)
    let newPart = factory.makeWidget(havingGearCount: 42, andSpindleCount: 14)
    let ref = Link(to: destination)
    ```

* Name **mutating/nonmutating** method pairs consistently. A mutating method will often have a nonmutating variant with similar semantics, but that returns a new value rather than updating an instance in-place.

  * **Mutating**
      * `x.sort()`
      * `x.append(y)`

  * **Nonmutating**    
      * `z = x.sorted()`
      * `z = x.appending(y)`

* Boolean methods and properties should read as an assertions. E.g. `x.isEmpty`

### 3.4 Initializer

For clarity, initializer arguments that correspond directly to a stored property have the same name as the property. Explicit `self.` is used during assignment to disambiguate them.

```swift
// PREFERRED
public struct Person {
    public let name: String
    public let phoneNumber: String

    public init(name: String, phoneNumber: String) {
        self.name = name
        self.phoneNumber = phoneNumber
    }
}

// NOT PREFERRED
public struct Person {
    public let name: String
    public let phoneNumber: String

    public init(name otherName: String, aPhoneNumber: String) {
      name = otherName
      phoneNumber = aPhoneNumber
    }
}
```

### 3.5 Protocols

A protocol can be used for multiple purpose, thought it can be named in different ways:

* A protocol should be named as a nouns if it describes what something is doing.

    ```swift
    // the name is a noun that describes what the protocol does
    protocol TableViewSectionProvider {
        func rowHeight(at row: Int) -> CGFloat
        var numberOfRows: Int { get }
        /* ... */
    }
    ```

* A protocol should have the suffixes `-able`, `-ible`, or `-ing` if it describes a capability.

    ```swift
    // the protocol is a capability, and we name it appropriately
    protocol Loggable {
        func logCurrentState()
        /* ... */
    }
    ```

* If a protocol is created for the purpose of the dependency injection or to generalize some of the functionalities, it should have the suffix `Type` to describe its generic purpose.

    ```swift
    // the protocol describe the `Person` class
    protocol PersonType {
        var name: String { get }
        var eyes: EyesColor { get }
        func talk()
    }

    class Person: PersonType {
	     var name: String

	     init(name: String, eyes: EyesColor) {
	         self.name = name
	         self.eyes = eyes
	     }

	     func talk() {
	         print("Hello, I'm \(name)!")
	     }
    }
    ```

### 3.6 Delegates and DataSource

Methods on delegate protocols and delegate-like protocols, such as data sources, are named using the linguistic syntax described below, which is inspired by Cocoa’s protocols.

> The term “delegate’s source object” refers to the object that invokes methods on the delegate. For example, a UITableView is the source object that invokes methods on the UITableViewDelegate that is set as the view’s delegate property.

* All methods take the delegate’s source object as the first argument.
* For methods that take the delegate’s source object as their **only** argument:

  * If the method returns `Void` (such as those used to notify the delegate that an event has occurred), then the method’s base name is the **delegate’s source type** followed by an **indicative verb phrase** describing the event. The argument is unlabeled.

        func scrollViewDidBeginScrolling(_ scrollView: UIScrollView)

  * If the method returns `Bool` (such as those that make an assertion about the delegate’s source object itself), then the method’s name is the **delegate’s source type** followed by an **indicative or conditional verb phrase** describing the assertion. The argument is **unlabeled**.

        func scrollViewShouldScrollToTop(_ scrollView: UIScrollView) -> Bool

  * If the method returns some other value (such as those querying for information about a property of the delegate’s source object), then the method’s base name is a **noun phrase** describing the property being queried. The argument is **labeled with a preposition or phrase with a trailing preposition** that appropriately combines the noun phrase and the delegate’s source object.

        func numberOfSections(in scrollView: UIScrollView) -> Int

### 3.7 Generics

Generic type parameters should be descriptive, upper camel case names. When a type name doesn't have a meaningful relationship or role, use a traditional single uppercase letter such as `T`, `U`, or `V`.

```swift
// PREFERRED
struct Stack<Element> { ... }
func write<Target: OutputStream>(to target: inout Target)
func swap<T>(_ a: inout T, _ b: inout T)
Not Preferred:

// NOT PREFERRED
struct Stack<T> { ... }
func write<target: OutputStream>(to target: inout target)
func swap<Thing>(_ a: inout Thing, _ b: inout Thing)
```

## 4. Coding Style

### 4.1 General formatting

* Prefer `let` to `var` whenever possible.
* Prefer the composition of `map`, `filter`, `reduce`, etc. over iterating when transforming from one collection to another.

    ```swift
    // PREFERRED
    let stringOfInts = [1, 2, 3].compactMap { String($0) }
    let evenNumbers = [4, 8, 15, 16, 23, 42].filter { $0 % 2 == 0 }

    // NOT PREFERRED
    var stringOfInts: [String] = []
    for integer in [1, 2, 3] {
        stringOfInts.append(String(integer))
    }
    ```

* If a function returns multiple values, prefer returning a tuple instead of to using `inout` arguments
* If you use a certain tuple more than once, consider using a `typealias`
* If you’re returning 3 or more items in a tuple, consider using a `struct` or `class` instead.
* Be aware of **retain cycles** when creating delegates/protocols for your classes, these properties should be declared `weak`.
* Be careful when calling `self` directly from an escaping closure as this can cause a retain cycle. Declare is as `[weak self]`.
* Don't place parentheses around control flow predicates.

    ```swift
    // PREFERRED
    if x == y {
        /* ... */
    }

    // NOT PREFERRED
    if (x == y) {
        /* ... */
    }
    ```

* Avoid writing out an enum type - use shorthand.

    ```swift
    // PREFERRED
    imageView.setImageWithURL(url, type: .person)

    // NOT PREFERRED
    imageView.setImageWithURL(url, type: AsyncImageView.Type.person)
    ```

* Don't write `self`, unless it is required.
* When using a statement such as `else`, `catch`, etc. that follows a block, put this keyword on the same line as the block. Follow the [1TBS style](https://en.m.wikipedia.org/wiki/Indentation_style#1TBS):

    ```swift
    // PREFERRED
    if someBoolean {
        // do something
    } else {
        // do something else
    }

    do {
        let fileContents = try readFile("filename.txt")
    } catch {
        print(error)
    }

    // NOT PREFERRED
    if someBoolean {
        // do something
    }
    else if {
        // do something else }
    else { // do something completely different }

    do {
        let fileContents = try readFile("filename.txt")
    }
    catch {
        print(error)}
    ```

* Prefer `static` to `class` when declaring a function or property that is associated with a class as opposed to an instance of that class. Only use class if you specifically need the functionality of overriding that function or property in a subclass, though consider using a `protocol` to achieve this instead.

* Prefer computed properties over functions

### 4.2 Error Handling

In general, developers tend to return `nil` when something can go wrong. Error handling is a better solution:

* When the result should *semantically* potentially be `nil`, it makes sense to return an optional instead of using error handling.
* If a method can *fail*, and the reason for the failure is not immediately obvious if using an optional return type, it makes sense for the method to throw an error.

Example with a custom `Error`:

```swift
struct Error: Swift.Error {
    public let file: StaticString
    public let function: StaticString
    public let line: UInt
    public let message: String

    public init(message: String, file: StaticString = #file, function: StaticString = #function, line: UInt = #line) {
        self.file = file
        self.function = function
        self.line = line
        self.message = message
    }
}

func readFile(named filename: String) throws -> String {
    guard let file = openFile(named: filename) else {
        throw Error(message: "Unable to open file named \(filename).")
    }

    let fileContents = file.read()
    file.close()
    return fileContents
}

func printSomeFile() {
    do {
        let fileContents = try readFile(named: filename)
        print(fileContents)
    } catch {
        print(error)
    }
}
```

### 4.3 Guard Statements

* Always prefer an "early return" strategy using the `guard` as opposed to nesting code in `if` statements.

    ```swift
    // PREFERRED
    func eatDoughnut(at index: Int) {
        guard index >= 0 && index < doughnuts.count else {
            // return early because the index is out of bounds
            return
        }

        let doughnut = doughnuts[index]
        eat(doughnut)
    }

    // NOT PREFERRED
    func eatDoughnut(at index: Int) {
        if index >= 0 && index < doughnuts.count {
            let doughnut = doughnuts[index]
            eat(doughnut)
        }
    }
    ```

* When unwrapping optionals, prefer guard statements as opposed to if statements to decrease the amount of nested indentation in your code.

    ```swift
    // PREFERRED
    guard let monkeyIsland = monkeyIsland else {
        return
    }
    bookVacation(on: monkeyIsland)
    bragAboutVacation(at: monkeyIsland)

    // NOT PREFERRED
    if let monkeyIsland = monkeyIsland {
        bookVacation(on: monkeyIsland)
        bragAboutVacation(at: monkeyIsland)
    }

    // JUST NO!
    if monkeyIsland == nil {
        return
    }
    bookVacation(on: monkeyIsland!)
    bragAboutVacation(at: monkeyIsland!)
    ```

* If choosing between two different states, it makes more sense to use an `if` statement as opposed to a `guard` statement.

    ```swift
    // PREFERRED
    if isFriendly {
        print("Hello, nice to meet you!")
    } else {
        print("You have the manners of a beggar.")
    }

    // NOT PREFERRED
    guard isFriendly else {
        print("You have the manners of a beggar.")
        return
    }

    print("Hello, nice to meet you!")
    ```

* When unwrapping is **not** involved, the most important thing to keep in mind is the readability of the code

    ```swift
    // an `if` statement is readable here
    if operationFailed {
        return
    }

    // double negative logic like this can get hard to read, don't do this
    guard !operationFailed else {
        return
    }
    ```

* If you need to unwrap multiple optionals, you have 2 possibilities:
  * If a failing unwrapping generates the same error, combine unwraps into a single `guard` statement

     ```swift
       guard 
           let thingOne = thingOne,
           let thingTwo = thingTwo,
           let thingThree = thingThree else {
           return
       }
     ```

  * If different unwrap lead to different errors, separate them

     ```swift
       // separate statements because we handle a specific error in each case
       guard let thingOne = thingOne else {
           throw Error(message: "Unwrapping thingOne failed.")
       }

       guard let thingTwo = thingTwo else {
           throw Error(message: "Unwrapping thingTwo failed.")
       }

       guard let thingThree = thingThree else {
           throw Error(message: "Unwrapping thingThree failed.")
       }
     ```

### 4.4 Access Modifiers

* Write the access modifier keyword first (if it is needed).
* Don't write the `internal` access modifier as it is the default one.
* The access modifier keyword should **not** be on a line by itself, but on the same line of the code it is referring to.

### 4.5 Custom Operators

* Define the custom operator in the file they are referred to. E.g. if you want to define a `==` operator for your `MyClass` class, add it in the same file `MyClass.swift`.
* Custom operators definitions must preserve the semantics of the operator. For example, `==` must always test equality and return a boolean.

### 4.6 Switch Statements and enums

* When using a switch statement that has a finite set of possibilities, do NOT include a `default` case. Instead, place unused cases at the bottom and use the break keyword to prevent execution.

* Since `switch` cases in Swift break by default, do not include the `break` keyword if it is not needed.

* The `case` statements should line up with the switch statement itself as per default Swift standards.

* When defining a case that has an associated value, make sure that this value is appropriately labeled as opposed to just types (e.g. `case hunger(hungerLevel: Int) `instead of `case hunger(Int)`).

    ```swift
    enum Problem {
        case attitude
        case hair
        case hunger(hungerLevel: Int)
    }

    func handleProblem(problem: Problem) {
        switch problem {
        case .attitude:
            print("At least I don't have a hair problem.")
        case .hair:
            print("Your barber didn't know when to stop.")
        case .hunger(let hungerLevel):
            print("The hunger level is \(hungerLevel).")
        }
    }
    ```

* Prefer lists of possibilities (e.g. `case 1, 2, 3:`) to using the `fallthrough` keyword where possible).

* If you have a `default` case that shouldn't be reached, preferably throw an error (or handle it some other similar way such as asserting).

    ```swift
    func handleDigit(_ digit: Int) throws {
        switch digit {
        case 0, 1, 2, 3, 4, 5, 6, 7, 8, 9:
            print("Yes, \(digit) is a digit!")
        default:
            throw Error(message: "The given number was not a digit.")
        }
    }
    ```

### 4.7 Optionals

* Use implicitly unwrapped types declared with `!` only for instance variables that you know will be initialized later before use, such as subviews that will be set up in `viewDidLoad`. In every other case, it is better to use a non-optional or regular optional property. Yes, there are cases in which you can probably "guarantee" that the property will never be `nil` when used, but it is better to be safe and consistent. Similarly, don't use force unwraps. "*It will never happen*" leads to errors and crashes.

* Don't use `as!` or `try!` for production code. You can use them in test as a crash in the test itself means that something went wrong.

* If you don't plan on actually using the value stored in an optional, but need to determine whether or not this value is `nil`, explicitly check this value against `nil` as opposed to using `if let` syntax.

    ```swift
    // PREFERERED
    if someOptional != nil {
        // do something
    }

    // NOT PREFERRED
    if let _ = someOptional {
        // do something
    }
    ```

* Don't use `unowned`. Since we don't ever want to have implicit unwraps, we similarly don't want `unowned` properties.

    ```swift
    // PREFERRED
    weak var parentViewController: UIViewController?

    // NOT PREFERRED
    weak var parentViewController: UIViewController!
    unowned var parentViewController: UIViewController
    ```

* When unwrapping optionals, use the same name for the unwrapped constant or variable where appropriate.

    ```swift
    guard let myValue = myValue else {
        return
    }
    ```

* Declare variables and function return types as optional with `?` where a `nil` value is acceptable.


### 4.8 Properties

* For read-only and computed property, provide the getter without the `get {}` around it.

    ```swift
    var computedProperty: String {
        if someBool {
            return "I'm a mighty pirate!"
        }
        return "I'm selling these fine leather jackets."
    }
    ```

* When using `get {}`, `set {}`, `willSet {}`, and `didSet {}`, indent these blocks.

* Try to stick to the standard `newValue`/`oldValue` identifiers in the `willSet {}` and `didSet {}`, even if you can use your custom name.

    ```swift
    var storedProperty: String = "I'm selling these fine leather jackets." {
        willSet {
            print("will set to \(newValue)")
        }
        didSet {
            print("did set from \(oldValue) to \(storedProperty)")
        }
    }

    var computedProperty: String  {
        get {
            if someBool {
                return "I'm a mighty pirate!"
            }
            return storedProperty
        }
        set {
            storedProperty = newValue
        }
    }
    ```

### 4.9 Closures

* Omit the type of the parameters.

    ```swift
    // PREFERRED
    doSomethingWithClosure() { response in
        print(response)
    }

    // NOT PREFERRED
    doSomethingWithClosure() { response: NSURLResponse in
        print(response)
    }
    ````

* Use shorthand syntax when possible, it makes code clear and compact

    ```swift
    [1, 2, 3].compactMap { String($0) }
    ```

* Don’t wrap the return parameter in parentheses unless it is required (e.g. if the type is optional or the closure is within another closure). Always wrap the arguments in the closure in a set of parentheses - use `()` to indicate no arguments and use `Void` to indicate that nothing is returned.

    ```swift
    let completionBlock: (Bool) -> Void = { (success) in
        print("Success? \(success)")
    }

    let completionBlock: () -> Void = {
        print("Completed!")
    }

    let completionBlock: (() -> Void)? = nil
    ```

* Keep parameter names on same line as the opening brace for closures.

* Use trailing closure syntax unless the meaning of the closure is not obvious without the parameter name (an example of this could be if a method has parameters for success and failure closures).

    ```swift
    // PREFERRED: trailing closure
    doSomething(1.0) { (parameter1) in
        print("Parameter 1 is \(parameter1)")
    }

    // PREFERRED: no trailing closure
    doSomething(1.0, success: { (parameter1) in
        print("Success with \(parameter1)")
    }, failure: { (parameter1) in
        print("Failure with \(parameter1)")
    })
    ```

### 4.10 Arrays

* Avoid accessing an array directly with subscripts. When possible, use accessors such as `.first` or `.last`, which are optional and won’t crash.

* Prefer using a `for item in items` syntax when possible as opposed to something like `for i in 0 ..< items.count`.

* Never use the `+=` or `+` operator to append/concatenate to arrays. Instead, use `.append()` or `.append(contentsOf:)` as these are far more performant.
