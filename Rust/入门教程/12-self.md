## self和Self
self 与Self

self与Self都是rust的关键字，看着像，用途不一样

self在方法参数中代表这个方法的接收对象，在模块中代表本模块的作用域。

Self是trait与impl代码块中的语法糖，用来代表当前方法的接收类型，支持泛型。




PATH 中：   
self resolves the path relative to the current module. self can only be used as the first segment, without a preceding ::   
```
fn foo() {}
fn bar() {
    self::foo();
}
```

Self, with a capital "S", is used to refer to the implementing type within traits and implementations.     
```

trait T {
    type Item;
    const C: i32;
    // `Self` will be whatever type that implements `T`.
    fn new() -> Self;
    // `Self::Item` will be the type alias in the implementation.
    fn f(&self) -> Self::Item;
}
struct S;
impl T for S {
    type Item = i32;
    const C: i32 = 9;
    fn new() -> Self {           // `Self` is the type `S`.
        S
    }
    fn f(&self) -> Self::Item {  // `Self::Item` is the type `i32`.
        Self::C                  // `Self::C` is the constant value `9`.
    }
}
```

