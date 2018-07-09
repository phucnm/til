Swift has default observers on properties of an object. Unfortunately, "They are not called while a class is setting its own properties, before the superclass initializer has been called." ([Apple docs](https://docs.swift.org/swift-book/LanguageGuide/Properties.html#//apple_ref/doc/uid/TP40014097-CH14-XID_368))

Swift developers have their own reason behind this. But you still may want the `didSet` or `willSet` will be triggered when you are initializing your instance. Here's a litte trick to achieve this using `defer`:

```
class MyClass {
    var myProperty: AnyObject {
        didSet {
            //do something
        }
    }

    init(someProperty: AnyObject) {
        defer { self.someProperty = someProperty }
    }
}
```

