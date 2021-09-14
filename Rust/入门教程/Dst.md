https://doc.rust-lang.org/reference/dynamically-sized-types.html
# Dynamically Sized Types
Most types have a fixed size that is known at compile time and implement the trait Sized.   
A type with a size that is known only at run-time is called a dynamically sized type (DST) or, informally非正式的, an unsized type.   
Slices and trait objects are two examples of DSTs.    
Such types can only be used in certain一些 cases:   
1. Pointer types to DSTs are sized but have twice the size of pointers to sized types 指向 DST 的指针类型是有大小的，但其大小是指向大小类型的指针的两倍   
Pointers to slices also store the number of elements of the slice.   
Pointers to trait objects also store a pointer to a vtable.    
2. DSTs can be provided as type arguments to generic type parameters having the special ?Sized bound. They can also be used for associated type definitions when the corresponding相应的 associated type declaration has a ?Sized bound. By default, any type parameter or associated type has a Sized bound, unless it is relaxed using ?Sized.
3. Traits may be implemented for DSTs. Unlike with generic type parameters, Self: ?Sized is the default in trait definitions.
4. Structs may contain a DST as the last field, this makes the struct itself a DST.

## Note: variables, function parameters, const items, and static items must be Sized.


https://doc.rust-lang.org/reference/special-types-and-traits.html#auto-traits
# Special types and traits
Certain types and traits that exist in the standard library are known to the Rust compiler

1. ```Box<T>```
```Box<T>``` has a few special features that Rust doesn't currently allow for user defined types.   
The dereference operator for ```Box<T> ```produces a place which can be moved from. This means that the * operator and the destructor of``` Box<T>``` are built-in to the language.   
Methods can take ```Box<Self>``` as a receiver.  
A trait may be implemented for```Box<T> ```in the same crate as T, which the orphan rules prevent for other generic types.

2. ```Rc<T> Arc<T> Pin<P>```
Methods can take them as a receiver

3. ```UnsafeCell<T>```
``` std::cell::UnsafeCell<T> is used for interior mutability. It ensures that the compiler doesn't perform optimisations that are incorrect for such types. It also ensures that static items which have a type with interior mutability aren't placed in memory marked as read only.```

4. 