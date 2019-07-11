# Modules in the Jumbo iOS app

## What is a module?

A module within the Jumbo iOS project is either a core module or a feature module, represented as a (development) (cocoa)pod. A core module on it's own doesn't provide any user value but is used by other modules (both core- & feature modules) to create feature modules. 

Our JumboUI module is a great example. It contains everything needed to build an app based on the Jumbo Online Styleguide but on it's own doesn't provide our customers with much.

### Core modules

At time of writing we have identified 5 core modules:

#### JumboCore

The core or core. This module contains our most shared code, extensions for example. Along with that it contains all base logic to implement paginated infinite scrollers, analytics tracking abstraction and much more.

#### JumboEntities

A collection of domain entity objects used by all of the modules. These entities form the link between different feature modules and the core or all UI components shared between the different feature modules. Examples of these entities are:

- Promotion.swift
- Product.swift
- Recipe.swift

Do note; Each feature module which needs to use these entities will have to define it's on `...Raw.swift` variant of that module. Each API call will be parsed into the Raw entities within the module and is converted into the JumboEntity whenever nessecary.

A short example with the Products modules (defined below) as a base;

```swift
extension JumboEntities.Product {
    init(raw: Products.ProductRaw) {
        ...
    }
}
```

#### JNetworkingKit

Short for JumboNetworkingKit. An [open source module](https://github.com/jumbo-tech-campus/JNetworkingKit) which is used by all our feature modules do do network calls in a structured way. Designed, developed and maintained by the Jumbo Mobile iOS development team.

### Feature modules

#### Foodcoach

Foodcoach is where it all started, it was our first feature module. It used JumboCore for it's analytics capabilities & JumboEcom for basket interactions.

#### Products (upcoming)

#### Promotions (upcoming)