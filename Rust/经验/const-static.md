https://doc.rust-lang.org/std/keyword.const.html    
# Keyword const
Compile-time constants and compile-time evaluable functions.  
## Compile-time constants
Sometimes a certain value is used many times throughout a program, and it can become inconvenient to copy it over and over. What’s more, it’s not always possible or desirable to make it a variable that gets carried around to each function that needs it. In these cases, the const keyword provides a convenient alternative to code duplication:  
```
const THING: u32 = 0xABAD1DEA;

let foo = 123 + THING;
```
Constants must be explicitly typed; unlike with let, you can’t ignore their type and let the compiler figure it out. Any constant value can be defined in a const, which in practice happens to be most things that would be reasonable to have in a constant (barring const fns). For example, you can’t have a File as a const.

The only lifetime allowed in a constant is 'static, which is the lifetime that encompasses all others in a Rust program. For example, if you wanted to define a constant string, it would look like this:   
```
const WORDS: &'static str = "hello rust!";
```
 
Thanks to static lifetime elision, you usually don’t have to explicitly use 'static:   
``` const WORDS: &str = "hello convenience!"; ```

const items looks remarkably similar to static items, which introduces some confusion as to which one should be used at which times. To put it simply, constants are inlined wherever they’re used, making using them identical to simply replacing the name of the const with its value. Static variables, on the other hand, point to a single location in memory, which all accesses share. This means that, unlike with constants, they can’t have destructors, and act as a single value across the entire codebase.

Constants, like statics, should always be in SCREAMING_SNAKE_CASE.

## Compile-time evaluable functions
The other main use of the const keyword is in const fn. This marks a function as being callable in the body of a const or static item and in array initializers (commonly called “const contexts”). const fn are restricted in the set of operations they can perform, to ensure that they can be evaluated at compile-time. See the Reference for more detail.

Turning a fn into a const fn has no effect on run-time uses of that function.

## Other uses of const
The const keyword is also used in raw pointers in combination with mut, as seen in *const T and *mut T. More about const as used in raw pointers can be read at the Rust docs for the pointer primitive.   

https://doc.rust-lang.org/reference/items/constant-items.html
# Constant items

https://doc.rust-lang.org/reference/const_eval.html   
# Constant evaluation

https://rust-lang.github.io/rfcs/2000-const-generics.html?highlight=const#when-a-const-variable-can-be-used   
# const_generics

https://rust-lang.github.io/rfcs/2920-inline-const.html?highlight=const#lints-about-placement-of-inline-const
# inline_const

https://rust-lang.github.io/rfcs/2342-const-control-flow.html?highlight=const#require-intermediate-const-fns-to-break-the-eager-const-evaluation
# const-control-flow

https://rust-lang.github.io/rfcs/2341-const-locals.html
# const_locals

https://rust-lang.github.io/rfcs/2344-const-looping.html
# const_looping

https://rust-lang.github.io/rfcs/2345-const-panic.html
# const_panic


https://rust-lang.github.io/rfcs/0911-const-fn.html   
# const_fn  
## Summary
Allow marking free functions and inherent methods as const, enabling them to be called in constants contexts, with constant arguments.    
## Motivation
```
#[lang="unsafe_cell"]
struct UnsafeCell<T> { pub value: T }
struct AtomicUsize { v: UnsafeCell<usize> }
const ATOMIC_USIZE_INIT: AtomicUsize = AtomicUsize {
    v: UnsafeCell { value: 0 }
};
```
As it is right now, UnsafeCell is a stabilization and safety hazard: the field it is supposed to be wrapping is public.   
This is only done out of the necessity出于需要 to initialize static items containing atomics, mutexes, etc.     
But，This approach is fragile and doesn't compose well - consider having to initialize an AtomicUsize static with usize::MAX - you would need a const for each possible value.   

Also, types like ```AtomicPtr<T> or Cell<T> ```have no way at all to initialize them in constant contexts, leading to overuse过度使用 of UnsafeCell or static mut, disregarding type safety and proper abstractions.

During implementation, the worst offender I've found was std::thread_local: all the fields of std::thread_local::imp::Key are public, so they can be filled in by a macro - and they're also marked "stable" (due to the lack of stability hygiene in macros).

A pre-RFC for the removal移动 of the dangerous (and often misused) static mut received positive feedback积极反馈, but only under the condition that abstractions could be created and used in const and static items.

Another concern另一个问题是 is the ability to use certain intrinsics一些内部函数, like size_of, inside constant expressions, including fixed-length array固定长度数组 types. Unlike keyword-based alternatives关键字替代方案, const fn provides an extensible and composable可组合 building block for such features.

The design should be as simple as it can be, while keeping enough functionality to solve the issues mentioned above.

The intention意图 of this RFC is to introduce a minimal change that enables safe abstraction resembling类似 the kind of code that one writes outside of a constant. Compile-time pure constants (the existing const items) with added parametrization over types and values (arguments) should suffice足够了.

This RFC explicitly does not introduce a general CTFE mechanism. In particular, conditional branching and virtual dispatch are still not supported in constant expressions, which imposes a severe limitation on what one can express.

## Detailed design
Traits, trait implementations and their methods cannot be const - this allows us to properly design a constness/CTFE system that interacts well with traits - for more details, see Alternatives.   

Functions and inherent methods can be marked as const:   
```
const fn foo(x: T, y: U) -> Foo {
    stmts;
    expr
}
impl Foo {
    const fn new(x: T) -> Foo {
        stmts;
        expr
    }

    const fn transform(self, y: U) -> Foo {
        stmts;
        expr
    }
}
```
Only simple by-value bindings are allowed in arguments, e.g. x: T. While by-ref bindings and destructuring can be supported, they're not necessary and they would only complicate the implementation.参数中只允许简单的按值绑定，例如x: T。虽然可以支持 by-ref 绑定和解构，但它们不是必需的，它们只会使实现复杂化。 








https://rust-lang.github.io/rfcs/0246-const-vs-static.html?highlight=const#static--const      
# Summary
Divide global declarations into two categories:   
1. constants declare constant values. These represent a value, not a memory address. This is the most common thing one would reach for and would replace static as we know it today in almost all cases.   
2. statics declare global variables. These represent a memory address. They would be rarely used: the primary use cases are global locks, global atomic counters, and interfacing with legacy C libraries.   

## Motivation
We have been wrestling with the best way to represent globals for some times.    
一段时间以来，我们一直在努力寻找代表全局变量的最佳方式。   
There are a number of interrelated issues:     






# const fn
```
    // a normal function
    fn foo(n: usize) -> usize {
        n + 1
    }

    fn main() {
        const BAR: usize = foo(5);
        let array = [0_u8; foo(7)];
    }

const BAR: usize = foo(5)应改为
    // a constant function
    const fn foo(n: usize) -> usize {
        n + 1
    }
```
const fn is currently much more limited than a normal function.   

 you can also now use loop and if in constant functions but there's a lot that won't work. Perhaps most notably,iterators won't work at all

 You can use it in more places than an ordinary function. It can be used as a static variable, a const variable, as a length for an array, etc.

 So should you mark every function as const fn so long as it compiles? No! const fn is a contract between you and the people that call your function. You are declaring that you will never change the function in ways that are invalid for const fn. This contract may prevent some future optimizations so think carefully before using it.

 # 0911
 ## Detailed design
 Traits, trait implementations and their methods cannot be const(this allows us to properly design a constness/CTFE system that interacts well with traits -as they are never const in this design)      
 Only simple by-value bindings are allowed in arguments, e.g. x: T. by-ref bindings and destructuring are't support   
 Functions and inherent methods can be marked as const:   
 ```
const fn foo(x: T, y: U) -> Foo {
    stmts;
    expr
}
impl Foo {
    const fn new(x: T) -> Foo {
        stmts;
        expr
    }

    const fn transform(self, y: U) -> Foo {
        stmts;
        expr
    }
}
 ```


  the meaning of static   


 ```static global_variable: i32 = 5;```
 space for the variable is allocated in a separate segment of the memory, existing for the whole execution of the program.



 ```fn foo<F: Human + 'static>(param: F) {} ```
 ```
fn main() {
    let kate = Kate { age: 30 };
    foo(kate);
}
 ```

 Putting a constraint like T: 'a means that all lifetime parameters of the type T must satisfy the lifetime constraint 'a (thus, must outlive it).

 For example, if I have this struct:  
 ```
struct Kate<'a, 'b> {
    address: &'a str,
    lastname: &'b str
}
 ```
Kate<'a, 'b> will satisfy the constraint F: Human + 'static only if 'a == 'static and 'b == 'static.

However, a struct without any lifetime parameter will always satisfy any lifetime constraint.  
>So as a summary, a constraint like F: 'static means that either:  
    1.  F has no lifetime parameter   
    2. all lifetime parameters of F are 'static


