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