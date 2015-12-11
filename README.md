# Swift — Subclass Initializers

## Objectives

1. Call a super class's initializer using the `super` keyword.
2. Correctly set up the subclass's additional properties within an initializer.
3. Override a superclass's default property value.
4. Use the `override` keyword when writing a default initializer for a subclass.
5. Correctly handle setting a constant property's value on the superclass from the subclass's initializers.

## Introduction

An important aspect of properly building an inheritance chain is providing initializers on subclasses. There are several particulars around initializer methods in Swift due to its strongly-typed nature.

For starters, initializers are not implicitly inherited.

**Important:** A subclass's initializer must call a *designated* initializer on the superclass.

Let's set up our superclass `Item` with two properties and a designated initializer:

```swift
class Item {
    let name: String
    var priceInCents: Int
       
    init(name: String, priceInCents: Int) {
        self.name = name
        self.priceInCents = priceInCents
    }
}
```

And let's set up a subclass named `GroceryItem` that adds no additional properties, the compiler allows us to use the superclass's initializers:

```swift
class GroceryItem: Item {
    // no additional properties
}
```

```swift
let grocery = GroceryItem()
print(grocery.name)
print(grocery.priceInCents)

let pizza = GroceryItem(name: "pizza", priceInCents: 899)
print(pizza.name)
print(pizza.priceInCents)
```

These will print:

```
item
0
pizza
899
```

**Top-tip:** *If you find yourself creating a subclassing without adding new properties, then you probably don't need to be subclassing. Consider writing an extension instead.*

However, adding a new non-optional property to our subclass without providing a default value will cause the compiler to complain:

```swift
class GroceryItem: Item {
    var expiration: NSDate  // error
}
```

![](https://curriculum-content.s3.amazonaws.com/swift/swift-subclass-initializers/error_subclass_property_not_initialized.png)

The solution is to write an initializer on the subclass that covers the property.

### Calling A Superclass's Initializer

So let's try to write an initializer on our subclass that sets all three properties:

```swift
class GroceryItem: Item {
    var expiration: NSDate
    
    init(name: String, priceInCents: Int, expiration: NSDate) {
        self.name = name  // error
        self.priceInCents = priceInCents
        self.expiration = expiration
    }
}
```

![](https://curriculum-content.s3.amazonaws.com/swift/swift-subclass-initializers/error_cannot_reassign_constant_property_from_subclass.png)

We can't take this approach if any of the properties covered by the superclass's initializers is immutable. This means that we'll have to call the superclass's initializer from within our subclass's initializer. We can do this by using the `super` keyword to call the superclass's initializer, passing along each of the arguments.

If you're a developer in Objective-C, you might assume that the pattern for this in Swift would follow things in this order:

```swift
class GroceryItem: Item {
    var expiration: NSDate
    
    init(name: String, priceInCents: Int, expiration: NSDate) {
        super.init(name: name, priceInCents: priceInCents)  // error
        self.expiration = expiration
    }
}
```

![](https://curriculum-content.s3.amazonaws.com/swift/swift-subclass-initializers/error_subclass_property_must_be_set_before_super_init.png)

But you'd be wrong. In Swift, the compiler expects new properties on the subclass to be set *before* the call to the superclass's initializer. This is correct:

```swift
class GroceryItem: Item {
    var expiration: NSDate
    
    init(name: String, priceInCents: Int, expiration: NSDate) {
        self.expiration = expiration
        super.init(name: name, priceInCents: priceInCents)
    }
}
```

### Overriding A Superclass's Default Property Value

Let's take an altered version of our `Item` superclass that sets a default value for `priceInCents` of `0` instead of accepting an initializer argument:

```swift
class Item {
    let name: String
    var priceInCents: Int = 0
       
    init(name: String) {
        self.name = name
    }
}
```

While subclassing `Item` into `GroceryItem`, we can alter this default value within an initializer *following* the call to the superclass's initializer:

```swift
class GroceryItem: Item {
    var expiration: NSDate
    
    init(name: String, expiration: NSDate) {
        self.expiration = expiration
        super.init(name: name)
        self.priceInCents = 199
    }
}
```

Attempting to set a mutable property on the superclass *before* the call to the superclass's initializer will produce a compiler error:

```swift
class GroceryItem: Item {
    var expiration: NSDate
    
    init(name: String, expiration: NSDate) {
        self.expiration = expiration
        self.priceInCents = 199  // error
        super.init(name: name)
    }
}
```

![](https://curriculum-content.s3.amazonaws.com/swift/swift-subclass-initializers/error_cannot_set_superclass_property_before_super_init.png)

### Overriding `init()`

Let's set up a default initializer on our superclass `Item`:

```swift
class Item {
    let name: String
    var priceInCents: Int
    
    init() {
        self.name = "item"
        self.priceInCents = 0
    }
}
```

**Note:** *On the surface, this is equivalent to just assigning default values to both properties—except that constant properties with default values cannot also be set in other initializers:*

```swift
class Item {
    let name: String = "item"
    var priceInCents: Int = 0
}
```
**:End of Note**

There are no arguments to accept and assign, just default values to hand to properties.

Attempting to set these properties within a default initializer declared on the subclass actually produces two distinct errors:

```swift
class GroceryItem: Item {
    var expiration: NSDate
    
    init() {  // error
        self.name = "grocery item"  // error
        self.priceInCents = 0
        self.expiration = NSDate()
    }
}
```

![](https://curriculum-content.s3.amazonaws.com/swift/swift-subclass-initializers/errors_override_keyword_cannot_set_constant_property.png)

The first error tells us to use the `override` keyword. Whether or not we explicitly define a default initializer for any class, it will be compiled with a default initializer known as `init()`. If we manually declare it in a subclass, we are forced to use the `override` keyword to denote its true nature as an overridden initializer.

The second error tells us that `self.name` is a `let` constant that is already set. Our choices are to either leave the superclass's immutable property alone:

```swift
class GroceryItem: Item {
    var expiration: NSDate
    
    override init() {
        self.priceInCents = 0
        self.expiration = NSDate()
    }
}
```

Or to write a designated initializer on the superclass:

```swift
class Item {
    let name: String
    var priceInCents: Int = 0
    
    init() {
        self.name = "item"
    }
    
    init(name: String) {
        self.name = name
    }
}
```

And then call the superclass's designated initializer within the subclass's default initializer, passing in an argument for the desired property value for the subclass:

```swift
class GroceryItem: Item {
    var expiration: NSDate
    
    override init() {
        self.expiration = NSDate()
        super.init(name: "grocery item")
        self.priceInCents = 199
    }
}
```
