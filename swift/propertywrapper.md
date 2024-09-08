# **A Simple Use Case for Property Wrappers**

Let’s start with a familiar example: managing your app’s small data with `UserDefaults`. Here’s a common pattern for using `UserDefaults` to store and retrieve simple values:

```swift
extension UserDefaults {

    public enum Keys {
        static let appVersion = "app_version"
        static let isLoggedIn = "is_logged_in"
    }

    var appVersion: Int {
        set {
            set(newValue, forKey: Keys.appVersion)
        }
        get {
            return integer(forKey: Keys.appVersion)
        }
    }

    var isLoggedIn: Bool {
        set {
            set(newValue, forKey: Keys.isLoggedIn)
        }
        get {
            return bool(forKey: Keys.isLoggedIn)
        }
    }
}
```

In this example, we extend `UserDefaults` to include computed properties `appVersion` and `isLoggedIn`. These properties provide an easy way to access and update specific user data. The getter and setter methods encapsulate the logic for interacting with `UserDefaults`.

However, this approach has a downside: it can be repetitive and boilerplate-heavy if you have multiple such properties. Each property requires its own getter and setter logic, potentially cluttering your code.

This is where Property Wrappers come into play. Property Wrappers provide a way to encapsulate and reuse this boilerplate code in a more elegant and maintainable way.

# **What is a Property Wrapper?**

Property Wrappers are a feature introduced in Swift 5.1 that allows you to define a reusable piece of logic for managing the state of properties. They encapsulate the logic for getting and setting a value, and they help to reduce boilerplate code by centralizing this logic into a single, reusable component.

# **How Property Wrappers Work**

A **Property Wrapper** is essentially a `struct`, `class`, or `enum` that wraps a property and provides a `wrappedValue` to manage its actual value, though it is commonly defined using a `struct`. By applying a Property Wrapper to a property, you can add custom logic for setting and retrieving the value while keeping the property's interface clean and simple.

Here’s a simple example of how you can use a Property Wrapper to manage `UserDefaults`, as shown in the first example, in a more elegant way:

```
@propertyWrapper
struct UserDefault<T> {
    private let key: String
    private let defaultValue: T

    init(key: String, defaultValue: T) {
        self.key = key
        self.defaultValue = defaultValue
    }

    var wrappedValue: T {
        get {
            return UserDefaults.standard.object(forKey: key) as? T ?? defaultValue
        }
        set {
            UserDefaults.standard.set(newValue, forKey: key)
        }
    }
}
```

In this `UserDefault` Property Wrapper:

- `key` is the key used to store and retrieve the value from `UserDefaults`.
- `defaultValue` is the value to use if the key does not exist in `UserDefaults`.
- The `wrappedValue` property manages getting and setting the value.

Now, you can use this Property Wrapper in your code like this:

```
extension UserDefaults {

    @UserDefault(key: "app_version", defaultValue: 1)
    var appVersion: Int

    @UserDefault(key: "is_logged_in", defaultValue: false)
    var isLoggedIn: Bool
}
```

By applying the `@UserDefault` Property Wrapper, you eliminate the need to write repetitive getter and setter methods. Instead, the Property Wrapper handles the logic of accessing and updating `UserDefaults`, making your code more concise and easier to maintain.

With Property Wrappers, you can centralize common logic, reduce boilerplate code, and improve code readability. This approach not only makes your code cleaner but also makes it easier to implement consistent behaviors across your application.

# **Property Wrapper Syntax**

To use Property Wrappers in Swift, you’ll need to understand the key components and their syntax:

# **@propertyWrapper**

The `@propertyWrapper` attribute is used to mark a `struct`, `class`, or `enum` as a Property Wrapper. This attribute tells Swift that the type is designed to manage the storage and logic for a property. Here’s a basic example of a Property Wrapper definition:

```
@propertyWrapper
struct ExampleWrapper {
    // Internal storage for the value
    private var internalValue: Int

    // The `wrappedValue` property manages access to the actual value
    var wrappedValue: Int {
        get {
            return internalValue
        }
        set {
            // Add some logic to transform/update the actual value
            internalValue = newValue
        }
    }

    // Initializer to set up the initial value
    init(wrappedValue: Int) {
        self.internalValue = wrappedValue
    }
}
```

# **wrappedValue**

The `wrappedValue` property is required in every Property Wrapper. It is the main interface through which the wrapped property’s value is accessed and modified. The `wrappedValue` property provides the getter and setter methods to manage how the underlying value is retrieved and updated. For instance:

```
@propertyWrapper
struct Uppercase {
    private var value: String

    var wrappedValue: String {
        get { return value.uppercased() }
        set { value = newValue.uppercased() }
    }

    init(wrappedValue: String) {
        self.value = wrappedValue
    }
}
```

In this example, `wrappedValue` ensures that the value is always uppercased when accessed or set.

# **Getter/Setter**

The `getter` and `setter` methods within `wrappedValue` allow you to define custom logic for accessing and updating the property’s value. These methods control how the property interacts with its underlying storage:

- **Getter**: Retrieves the current value of the property. This is where you can add logic to transform or calculate the value before returning it.
- **Setter**: Updates the value of the property. Here, you can add logic to transform or validate the new value before storing it.

# **Internal Value**

Property Wrappers often use an internal value to manage the updated value. This internal value is usually stored in a private property and is accessed and modified through the `wrappedValue`. Here’s an example:

```
@propertyWrapper
struct Clamped {
    private var internalValue: Int
    private let minValue: Int
    private let maxValue: Int

    var wrappedValue: Int {
        get { return internalValue }
        set { internalValue = min(max(newValue, minValue), maxValue) }
    }

    init(wrappedValue: Int, minValue: Int, maxValue: Int) {
        self.minValue = minValue
        self.maxValue = maxValue
        self.internalValue = min(max(wrappedValue, minValue), maxValue)
    }
}
```

In this example, `internalValue` is used to store the actual value, while `wrappedValue` ensures the value is clamped within a specified range.

# **How Swift Synthesizes Property Wrappers: An Example**

When you declare a property with a Property Wrapper, for example, `@MyPropertyWrapper var propertyA`, Swift synthesizes your code into two properties:

1. `_propertyA`: This provides access to the instance of the property wrapper (in this case, `MyPropertyWrapper`).
2. `propertyA`: A computed property that accesses the value of the wrapped property.

Let’s take a look at how Swift handles Property Wrappers behind the scenes, using the `TwelveOrLess` wrapper as an example.

# **Property Wrapper Definition**

Here’s the `TwelveOrLess` Property Wrapper:

```
@propertyWrapper
struct TwelveOrLess {
    private var _value = 0

    var wrappedValue: Int {
        get {
            return _value
        }
        set {
            _value = min(newValue, 12)
        }
    }
}
```

This wrapper ensures that any value assigned to it doesn’t exceed 12.

# **Using the Property Wrapper**

In a `SmallRectangle` struct, you can use the `TwelveOrLess` wrapper like this:

```
struct SmallRectangle {
    @TwelveOrLess var height: Int
    @TwelveOrLess var width: Int

    func someMethod() {
        self.width // Int
        self._width // TwelveOrLess
        self._width.wrappedValue // Int
    }
}
```

# **Synthesized Code**

When Swift compiles this code, it automatically generates additional code to manage the Property Wrapper. Here’s what Swift creates behind the scenes:

```
struct SmallRectangle {
    private var _height = TwelveOrLess(wrappedValue: 0)
    private var _width = TwelveOrLess(wrappedValue: 0)

    var height: Int {
        get { return _height.wrappedValue }
        set { _height.wrappedValue = newValue }
    }

    var width: Int {
        get { return _width.wrappedValue }
        set { _width.wrappedValue = newValue }
    }
}
```

There are two types of properties to manage the values:

- **Private Storage**: Swift creates private variables `_height` and `_width` to hold instances of `TwelveOrLess`. These are used internally to store the values. You can directly access it if you want.
- **Computed Properties**: The `height` and `width` properties interact with these private variables through the `wrappedValue` property of the `TwelveOrLess` instances.

# **Accessing the Property Wrapper**

To use Property Wrappers effectively, you need to know the different ways to access their values. In the `someMethod` function:

- `self.width` accesses the `width` property directly, using the synthesized getter and setter.
- `self._width` allows you to access the underlying `TwelveOrLess` instance directly.
- `self._width.wrappedValue` lets you get or set the actual value managed by the `TwelveOrLess` wrapper. It has the same value with `self.width`

This synthesis simplifies your code by handling the boilerplate of managing property values and encapsulating the logic in one place. If you work with SwiftUI, you’ll encounter this more frequently.

# **Setting Initial Values for Wrapped Properties**

When working with Property Wrappers, you might encounter situations where you need to set default values for your wrapped properties. Here’s how you can handle initialization effectively.

# **Default Initialization Issue**

In the previous example, you might have noticed that you can’t define a default value for a Property Wrapper directly, resulting in an error like `Argument passed to call that takes no arguments`.

```
struct SmallRectangle {
    // Argument passed to call that takes no arguments
    @TwelveOrLess var height: Int = 1
    @TwelveOrLess var width: Int
}
```

To solve this issue, you need to provide an initializer (`init()`) in the Property Wrapper.

# **Initializers in Property Wrappers**

By default, if you don’t assign any initial value to the wrapped property, Swift will call the default initializer `init()` of the Property Wrapper. Here’s how you can enhance the `TwelveOrLess` Property Wrapper to support initial values and custom configuration:

```
@propertyWrapper
struct SmallNumber {
    private var maximum: Int = 12
    private var _value: Int = 0

    var wrappedValue: Int {
        get {
            return _value
        }
        set {
            _value = min(newValue, maximum)
        }
    }

    // Default initializer
    init(wrappedValue: Int) {
        _value = min(wrappedValue, maximum)
    }

    // Initializer with maximum value
    init(wrappedValue: Int, maximum: Int) {
        self.maximum = maximum
        _value = min(wrappedValue, maximum)
    }
}
```

# **Applying the Initializers**

**Case 1**: No initializer parameters provided.

When you don’t provide initial values for the wrapped property, the default initializer `init()` is called:

```
struct UnitRectangle {
    @SmallNumber var height: Int
    @SmallNumber var width: Int
}
```

**Case 2**: Initializer with custom maximum value.

You can specify both the `wrappedValue` and the `maximum.` It will call the initializer `init(wrappedValue: Int, maximum: Int)`

```
struct NarrowRectangle {
    @SmallNumber(wrappedValue: 2, maximum: 5) var height: Int
    @SmallNumber(wrappedValue: 3, maximum: 4) var width: Int
}
```

**Example 2**: Mixed initializers.

You can mix different ways of initialization. The compiler will use the appropriate initializer based on the provided values:

```
struct MixedRectangle {
    @SmallNumber(maximum: 9) var width: Int = 2
}
```

In this example, Swift will call the initializer `init(wrappedValue: Int, maximum: Int)`

# **projectedValue**

In addition to exposing the `wrappedValue` of a Property Wrapper, Swift also supports an optional `projectedValue` property. This property is accessed using the `$` syntax (if you’ve used `@Binding` in SwiftUI, you're likely familiar with it) and can provide additional insights or functionality related to the Property Wrapper.

When using a Property Wrapper in your class or struct, you can access the `projectedValue` in two ways:

1. Using the `$` syntax, e.g., `$yourPropertyName`
2. Accessing the instance of the property and calling `projectedValue`, e.g., `_yourPropertyName.projectedValue` (this approach is rarely used).

# **Example 1: `projectedValue` as a Publisher**

In this example, we will implement behavior similar to the `@Published` property wrapper, where the `projectedValue` is a Publisher that emits updates whenever the `wrappedValue` changes.

```
@propertyWrapper
struct UserDefault<Value> {
    private let key: String
    private let defaultValue: Value
    private var container: UserDefaults = .standard
    private let publisher = PassthroughSubject<Value, Never>()

    var wrappedValue: Value {
        get {
            return container.object(forKey: key) as? Value ?? defaultValue
        }
        set {
            container.set(newValue, forKey: key)
            publisher.send(newValue)
        }
    }

    var projectedValue: AnyPublisher<Value, Never> {
        return publisher.eraseToAnyPublisher()
    }
}
```

Here’s an example of how to use it:

```
class SettingsViewModel {
    @UserDefault(key: "isDarkMode", defaultValue: false)
    var isDarkMode: Bool

    private var cancellables = Set<AnyCancellable>()

    init() {
        // Subscribe to changes using the projected value ($isDarkMode)
        $isDarkMode
            .sink { newValue in
                print("Dark Mode is now: \(newValue)")
            }
            .store(in: &cancellables)
    }
}
```

# **Example 2: `projectedValue` in SwiftUI**

1. The `projectedValue` of a `@State` property wrapper is a property wrapper instance with type of `Binding<Value>`

```
struct ParentView: View {
    @State private var counter: Int = 0  // State property

    var body: some View {
        VStack {
            Text("Parent Counter: \(counter)")
                .padding()

            // Passing $counter as a Binding to ChildView
            // Binding is projected with the $ syntax
            ChildView(counter: $counter)

            Button("Increase in Parent") {
                counter += 1
            }
            .padding()
        }
    }
}
```

2. The `projectedValue` of `@Binding` is simply itself, as `@Binding` is already a `Binding<Value>`.

```
struct SliderView: View {
    // Receiving the projectedValue (Binding) from ParentViewWithSlider
    @Binding var sliderValue: Double

    var body: some View {
        VStack {
            Text("Child Slider Value: \(sliderValue)")
                .padding()

            // A slider that is bound to the parent's state
            Slider(value: $sliderValue, in: 0...1)
                .padding()
        }
    }
}
```

# **Conclusion**

In this detailed exploration of Property Wrappers in Swift, we have covered their fundamental concepts, practical applications, and the enhanced capabilities they offer. Property Wrappers are a powerful feature introduced in Swift 5.1 that streamline the management of property values, reduce boilerplate code, and centralize logic for property access and mutation.

# **Key Takeaways:**

1. **Simplifying UserDefaults Management**: We began by demonstrating how Property Wrappers can simplify the interaction with `UserDefaults`. The traditional approach involves repetitive boilerplate code for each property. By using a Property Wrapper like `UserDefault`, you can encapsulate the logic for accessing and updating `UserDefaults` in a single reusable component, making your code cleaner and more maintainable.
2. **Defining and Using Property Wrappers**: Property Wrappers are defined with the `@propertyWrapper` attribute and include the `wrappedValue` property to manage the actual value. This approach allows for custom logic in the getter and setter methods, which can be used to transform or validate property values. Examples like `Uppercase` and `Clamped` showcase how you can enforce constraints or transformations on property values.
3. **Initialization Flexibility**: We discussed how to handle initial values for wrapped properties effectively. Property Wrappers require initialization through their defined initializers, and by using custom initializers, you can configure default values and additional settings, such as maximum limits.
4. **Exploring `projectedValue`**: The `projectedValue` property is a powerful feature that provides additional insights or functionality related to the wrapped property. By exposing `projectedValue`, you can offer more than just the updated value, such as bindings or publishers. This is particularly useful in frameworks like SwiftUI, where `projectedValue` is used to create bindings for state management.
5. **Behind-the-Scenes Synthesis**: Swift handles Property Wrappers by synthesizing code that manages both the wrapped value and the Property Wrapper instance. Understanding this synthesis helps you appreciate how Property Wrappers simplify the management of property values while hiding the underlying complexity.

Overall, Property Wrappers in Swift provide a clean and efficient way to manage properties with reusable logic, enhancing code readability and maintainability. Whether you’re managing user defaults, enforcing value constraints, or integrating with frameworks like SwiftUI, Property Wrappers offer a versatile and elegant solution to common programming challenges.

# **References**

- https://docs.swift.org/swift-book/documentation/the-swift-programming-language/properties/
- https://www.avanderlee.com/swift/property-wrappers/