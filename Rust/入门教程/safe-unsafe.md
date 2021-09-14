# safe and unsafe 笔记
## the relationship between Safe Rust and Unsafe Rust
1. controlled with the unsafe keyword，The unsafe keyword has two uses: to declare the existence of contracts the compiler can't check, and to declare that a programmer has checked that these contracts have been upheld维护.    
unsafe function  And  unsafe trail

You can use unsafe on a block to declare that all unsafe actions performed within are verified to uphold the contracts of those operations. For instance, the index passed to slice::get_unchecked is in-bounds.


You can use unsafe on a trait implementation to declare that the implementation upholds the trait's contract. For instance, that a type implementing Send is really safe to move to another thread.

The standard library has a number of unsafe functions, including:    
slice::get_unchecked, which performs unchecked indexing, allowing memory safety to be freely violated.   
mem::transmute reinterprets some value as having a given type, bypassing type safety in arbitrary ways以任意方式绕过类型安全 (see conversions for details).    
Every raw pointer to a sized type has an offset method that invokes Undefined Behavior if the passed offset is not "in bounds".   
All FFI (Foreign Function Interface) functions are unsafe to call because the other language can do arbitrary operations that the Rust compiler can't check.   

As of Rust 1.29.2 the standard library defines the following unsafe traits ：   
Send is a marker trait (a trait with no API) that promises implementors are safe to send (move) to another thread.   
Sync is a marker trait that promises threads can safely share implementors through a shared reference.   
GlobalAlloc allows customizing the memory allocator of the whole program.   

## No matter what, Safe Rust can't cause Undefined Behavior.
The decision of whether to mark a trait unsafe is an API design choice. A safe trait is easier to implement, but any unsafe code that relies on it must defend against incorrect behavior. Marking a trait unsafe shifts this responsibility to the implementor. Rust has traditionally avoided marking traits unsafe because it makes Unsafe Rust pervasive, which isn't desirable.  

The decision of whether to mark your own traits unsafe depends on the same sort of consideration. If unsafe code can't reasonably expect to defend against a broken implementation of the trait, then marking the trait unsafe is a reasonable choice.

Send and Sync are marked unsafe because thread safety is a fundamental property that unsafe code can't possibly hope to defend against in the way it could defend against a buggy Ord implementation. Similarly, GlobalAllocator is keeping accounts of all the memory in the program and other things like Box or Vec build on top of it. If it does something weird (giving the same chunk of memory to another request when it is still in use), there's no chance to detect that and do anything about it.       
As an aside, while Send and Sync are unsafe traits, they are also automatically implemented for types when such derivations are provably safe to do. Send is automatically derived for all types composed only of values whose types also implement Send. Sync is automatically derived for all types composed only of values whose types also implement Sync. This minimizes the pervasive unsafety of making these two traits unsafe. And not many people are going to implement memory allocators (or use them directly, for that matter).

# What Unsafe Rust Can Do
 Unsafe Rus can:   
1. Dereference raw pointers
2. Call unsafe functions (including C functions, compiler intrinsics, and the raw allocator)
3. Implement unsafe traits
4. Mutate statics
5. Access fields of unions

The reason these operations are relegated降级 to Unsafe is that misusing any of these things will cause the ever dreaded Undefined Behavior. Invoking Undefined Behavior gives the compiler full rights to do arbitrarily bad things to your program. You definitely should not invoke Undefined Behavior.    

Unlike C, Undefined Behavior is pretty limited in scope in Rust. All the core language cares about is preventing the following things:   
1. Dereferencing (using the * operator on) dangling or unaligned pointers (see below) 
2. Breaking the pointer aliasing rules
3. Calling a function with the wrong call ABI or unwinding from a function with the wrong unwind ABI.
4. Causing a data race
5. Executing code compiled with target features that the current thread of execution does not support
6. Producing invalid values (either alone or as a field of a compound复合类型字段 type such as enum/struct/array/tuple):   
a bool that isn't 0 or 1   
an enum with an invalid discriminant无效判别的枚举  
a null fn pointer  
a char outside the ranges [0x0, 0xD7FF] and [0xE000, 0x10FFFF]   
a ! (all values are invalid for this type)   
an integer (i*/u*), floating point value (f*), or raw pointer read from uninitialized memory, or uninitialized memory in a str.从未初始化的内存或 str 中的未初始化内存读取的整数 (i*/u*)、浮点值 (f*) 或原始指针   
a reference/Box that is dangling, unaligned, or points to an invalid value.   
a wide reference, Box, or raw pointer that has invalid metadata:   
dyn Trait metadata is invalid if it is not a pointer to a vtable for Trait that matches the actual dynamic trait the pointer or reference points to   
slice metadata is invalid if the length is not a valid usize (i.e., it must not be read from uninitialized memory)   
a type with custom invalid values that is one of those values, such as a NonNull that is null. (Requesting custom invalid values is an unstable feature, but some stable libstd types, like NonNull, make use of it.)   

https://doc.rust-lang.org/reference/types/trait-object.html
## Trait objects 
A trait object is an opaque value of another type that implements a set of traits.trait 对象是另一种类型的不透明值，它实现了一组 trait    
The set of traits is made up of an object safe base trait plus any number of auto traits.这组trail由一个对象安全的基本特征加上任意数量的自动特征组成    
Trait objects implement the base trait, its auto traits, and any supertraits of the base trait.

用法：    
Trait objects are written as the optional keyword dyn followed by a set of trait bounds, but with the following restrictions on the trait bounds.     
All traits except the first trait must be auto traits, there may not be more than one lifetime, and opt-out bounds (e.g. ?Sized) are not allowed. 除了第一个特征之外的所有特征都必须是自动特征，生命周期不能超过一个，并且不允许选择退出边界     
Furthermore此外, paths to traits may be parenthesized括号括起来.    
```

    Trait
    dyn Trait
    dyn Trait + Send
    dyn Trait + Send + Sync
    dyn Trait + 'static
    dyn Trait + Send + 'static
    dyn Trait +
    dyn 'static + Trait.
    dyn (Trait)

```
Two trait object types alias each other if the base traits alias each other and if the sets of auto traits are the same and the lifetime bounds are the same. For example, dyn Trait + Send + UnwindSafe is the same as dyn Trait + UnwindSafe + Send.



Due to the opaqueness of which concrete type the value is of, trait objects are dynamically sized types.     
Like all DSTs, trait objects are used behind some type of pointer; for example &dyn SomeTrait or ```Box<dyn SomeTrait>```. Each instance of a pointer to a trait object includes:    
```

    a pointer to an instance of a type T that implements SomeTrait
    a virtual method table, often just called a vtable, which contains, for each method of SomeTrait and its supertraits that T implements, a pointer to T's implementation (i.e. a function pointer).

```

The purpose of trait objects is to permit "late binding" of methods. Calling a method on a trait object results in virtual dispatch at runtime: that is, a function pointer is loaded from the trait object vtable and invoked indirectly. The actual implementation for each vtable entry can vary on an object-by-object basis.     
An example of a trait object:    
```
trait Printable {
    fn stringify(&self) -> String;
}

impl Printable for i32 {
    fn stringify(&self) -> String { self.to_string() }
}

fn print(a: Box<dyn Printable>) {
    println!("{}", a.stringify());
}

fn main() {
    print(Box::new(10) as Box<dyn Printable>);
}
```
In this example, the trait Printable occurs as a trait object in both the type signature of print, and the cast expression in main.   

Trait Object Lifetime Bounds   
Since a trait object can contain references, the lifetimes of those references need to be expressed as part of the trait object. This lifetime is written as Trait + 'a. There are defaults that allow this lifetime to usually be inferred with a sensible choice一个明智的选择.




## dyn：  
dyn is a prefix of a trait object’s type.  
The dyn keyword is used to highlight强调 that calls to methods on the associated Trait are dynamically dispatched. To use the trait this way, it must be ‘object safe’.

Unlike generic parameters or impl Trait, the compiler does not know the concrete type具体类型 that is being passed. That is, the type has been erased. As such, a dyn Trait reference contains two pointers. One pointer goes to the data (e.g., an instance of a struct). Another pointer goes to a map of method call names to function pointers另一个指针指向方法调用名称到函数指针的映射 (known as a virtual method table or vtable).

At run-time, when a method needs to be called on the dyn Trait, the vtable is consulted to get the function pointer and then that function pointer is called.

The above indirection is the additional runtime cost of calling a function间接调用产生额外的运行时开销 on a dyn Trait. Methods called by dynamic dispatch generally cannot be inlined by the compiler.    
However, dyn Trait is likely to produce smaller code than impl Trait / generic parameters as the method won’t be duplicated重复 for each concrete type具体类型.

