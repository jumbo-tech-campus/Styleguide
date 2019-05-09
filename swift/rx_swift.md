# RxSwift conventions

This document contains all conventions decided upon by our team on how to implement RxSwift in our MVVM architecture.

## Table Of Contents

- [Protocols](#protocols)
   + [ReactiveConnectable](#reactiveconnectable)
- [ViewControllers](#viewcontrollers)
   + [Binding to ReactiveConnectable](#binding-to-reactiveconnectable)

## Protocols

In order to assure all (new) MVVM powered views are implemented in the same way 

### ReactiveConnectable

`ReactiveConnectable` is the base protocol for any `-Model` type class, for example a `ViewModel` or a `TrackingModel`. It requires any implementation to define an `Input` & `Output` type, most of the time implemented in the form of a struct.
In most cases the `Input` will contain defaultly instantiated `PublishSubject` subjects to any event generator can bind to it, causing events to flow into your types.

The gist of `ReactiveConnectable` is as follows;
```swift
protocol ReactiveConnectable {
    associatedtype Input
    associatedtype Output

    var input: Input { get }
    var output: Output { get }

    func transform(input: Input) -> Output
}
```
Please note: The gist above might be outdated, always check the production project for the latest version.

#### Conventions
- The `Input` / `Output` properties should not be renamed in the implementation, even if the type can be inferred
- The `Input` / `Output` properties should not be assignable by after getting an initial value
- The `transform` function should only be called when initializing the `output` variable, see example below.
- The conformance to `ReactiveConnectabale` should always be implemented in a extension of your type
   + The only exceptions are the `input` and `output` stored properties

#### Example implementation

```swift
class ExampleViewModel {
    let input = Input()
    lazy var output: Output = { return transform(input: input) }()
}

extension ExampleViewModel: ReactiveConnectable {
    struct Input { }
    struct Output { }

    func transform(input: Input) -> Output {
        return Output()
    }
}
```

## ViewControllers

### Binding to ReactiveConnectable

Binding UI to the `ReactiveConnectable` protocol should always happen by a `-Binding` protocol. For example the protocol definitions to bind a `ViewModel` or a `TrackingModel`;
```swift
protocol ViewModelBindable {
    typealias ViewModelType: ReactiveConnectable

    func bind(to viewModel: ViewModelType)
    func bind(to input: ViewModelType.Input)
    func bind(to output: ViewModelType.Output)
}

extension ViewModelBindable {
    func bind(to viewModel: ViewModelType) {
        bind(to: viewModel.input)
        bind(to: viewModel.output)
    }
}
```

```swift
protocol TrackingModelBindable {
    typealias TrackingModelType: ReactiveConnectable

    func bind(to trackingModel: TrackingModelType)
    func bind(to input: TrackingModelType.Input)
    func bind(to output: TrackingModelType.Output)
}

extension TrackingModelBindable {
    func bind(to trackingModel: TrackingModelType) {
        bind(to: viewModel.input)
        bind(to: viewModel.output)
    }
}
```

This way we assure multiple `ReactiveConnectable` types can be bound to from a view controller, or any other type binding to `ReactiveConnectable` for that matter, and we limit the amount of `ViewModel`s and `TrackingModel`s which can bound to a single view controller at the same time.
The view controller should always implement a `-Bindable` protocol in an extension and initiate the binding by calling `bind(to connectable: ReactiveConnectable)` (thus not the input & output separately).

For example binding our `ExampleViewModel` to a `ExampleViewController` class:

```swift 
class ExampleViewController: UIViewController {
    let viewModel: ExampleViewModel

    init(viewModel: ExampleViewModel) {
        self.viewModel = viewModel
    }

    override func viewDidLoad() {
        super.viewDidLoad()

        bind(to: viewModel)
    }
}

extension ExampleViewController: ViewModelBindable {
    func bind(to input: ExampleViewModel.Input) {
        ...
    }

    func bind(to output: ExampleViewModel.Output) {
        ...
    }
}
```

As soon as analytics would be added to this view controller simply add another extension

```
class ExampleTrackingModel {
    let input = Input()
    lazy var output: Output = { return transform(input: input) }()
}

extension ExampleTrackingModel: ReactiveConnectable {
    struct Input { }
    struct Output { }

    func transform(input: Input) -> Output {
        return Output()
    }
}

class ExampleViewController: UIViewController {
    let viewModel: ExampleViewModel
    let trackingModel: ExampleTrackingModel
    init(viewModel: ExampleViewModel, trackingModel: ExampleTrackingModel) {
        self.viewModel = viewModel
        self.trackingModel = trackingModel
    }
    override func viewDidLoad() {
        super.viewDidLoad()
        
        bind(to: viewModel)
        bind(to: trackingModel)
    }
}

extension ExampleViewController: TrackingModelBindable {
    func bind(to input: ExampleTrackingModel.Input) {
        ...
    }

    func bind(to output: ExampleTrackingModel.Output) {
        ...
    }
}

```
