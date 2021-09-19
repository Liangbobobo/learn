https://doc.rust-lang.org/std/marker/index.html    
# Module std::marker
Primitive traits and types representing basic properties of types.   
Rust types can be classified in various useful ways according to their intrinsic内在 properties. These classifications are represented as traits.

# Macros

# Structs

# Trail
## 一、copy--Types whose values can be duplicated simply by copying bits.   
By default, variable bindings have ‘move semantics.However, if a type implements Copy, it instead has ‘copy semantics’
```
// We can derive a `Copy` implementation. `Clone` is also required, as it's
// a supertrait of `Copy`.
#[derive(Debug, Copy, Clone)]
struct Foo;

let x = Foo;

let y = x;

// `y` is a copy of `x`

println!("{:?}", x); // A-OK!
```

### How can I implement Copy?    
1.  use derive    
```
#[derive(Copy, Clone)]
struct MyStruct;
```

2.  implement Copy and Clone manually   
```
struct MyStruct;

impl Copy for MyStruct { }

impl Clone for MyStruct {
    fn clone(&self) -> MyStruct {
        *self
    }
}
```

There is a small difference between the two: the derive strategy will also place a Copy bound on type parameters, which isn’t always desired.??

### the difference between Copy and Clone
Copies happen implicitly, for example as part of an assignment y = x. The behavior of Copy is not overloadable; it is always a simple bit-wise copy.   

Cloning is an explicit action, x.clone(). The implementation of Clone can provide any type-specific behavior necessary to duplicate values safely. For example, the implementation of Clone for String needs to copy the pointed-to string buffer in the heap. A simple bitwise copy of String values would merely copy the pointer, leading to a double free down the line. For this reason, String is Clone but not Copy.

Clone is a supertrait of Copy, so everything which is Copy must also implement Clone. If a type is Copy then its Clone implementation only needs to return *self (see the example above).

### When can my type be Copy?
A type can implement Copy if all of its components implement Copy    
``` Vec<T> is not Copy.```    
Shared references (&T) are  Copy, so a type can be Copy, even when it holds shared references of types T that are not Copy.

### When can’t my type be Copy?
Because Some types can’t be copied safely.   
 1. copying &mut T would create an aliased mutable reference.    
 2. Copying String would duplicate responsibility for managing the String’s buffer, leading to a double free.Generalizing the  case概括这种情况，any type implementing Drop can’t be Copy, because it’s managing some resource besides its own ``` size_of::<T>``` bytes.?   
 3.  try to implement Copy on a struct(struct 内的字段不全是copy的) or enum containing non-Copy data, you will get the error E0204.

### When should my type be Copy?
Generally speaking, if your type can implement Copy, it should. 不过请记住Keep in mind, though, that implementing Copy is part of the public API of your type. If the type might become non-Copy in the future, it could be prudent小心 to omit the Copy implementation now, to avoid a breaking API change.


### In addition to the implementors listed below, the following types also implement Copy:
1. Function item types (i.e., the distinct types defined for each function)
2. Function pointer types (e.g., fn() -> i32)
3. Array types, for all sizes, if the item type also implements Copy (e.g., [i32; 123456])
4. Tuple types, if each component also implements Copy (e.g., (), (i32, bool))
5. Closure types, if they capture no value from the environment or if all such captured values implement Copy themselves. Note that variables captured by shared reference always implement Copy (even if the referent doesn’t), while variables captured by mutable reference never implement Copy.