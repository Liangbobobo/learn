https://doc.rust-lang.org/stable/rust-by-example/macros.html
# Macro
However, unlike macros in C and other languages, Rust macros are expanded into abstract syntax trees, rather than string preprocessing, so you don't get unexpected precedence bugs.

Macros are created using the macro_rules! macro.     
```
// This is a simple macro named `say_hello`.
macro_rules! say_hello {
    // `()` indicates that the macro takes no argument.
    () => {
        // The macro will expand into the contents of this block.
        println!("Hello!");
    };
}

fn main() {
    // This call will expand into `println!("Hello");`
    say_hello!()
}
```

why are macros useful?
1. Don't repeat yourself. There are many cases where you may need similar functionality in multiple places but with different types. Often, writing a macro is a useful way to avoid repeating code. (More on this later)
2. Domain-specific languages. Macros allow you to define special syntax for a specific purpose. (More on this later)
3. Variadic可变的 interfaces. Sometimes you want to define an interface that takes a variable number of arguments. An example is println! which could take any number of arguments, depending on the format string!. (More on this later)

# Syntax
## Designators 代号
The arguments of a macro are prefixed by a dollar sign $ and type annotated with a designator:  
```
macro_rules! create_function {
    // This macro takes an argument of designator `ident` and
    // creates a function named `$func_name`.
    // The `ident` designator is used for variable/function names.
    ($func_name:ident) => {
        fn $func_name() {
            // The `stringify!` macro converts an `ident` into a string.
            println!("You called {:?}()",
                     stringify!($func_name));
        }
    };
}

// Create functions named `foo` and `bar` with the above macro.
create_function!(foo);
create_function!(bar);

// run the code 
You called "foo"()
You called "bar"()

macro_rules! print_result {
    // This macro takes an expression of type `expr` and prints
    // it as a string along with its result.
    // The `expr` designator is used for expressions.
    ($expression:expr) => {
        // `stringify!` will convert the expression *as it is* into a string.
        println!("{:?} = {:?}",
                 stringify!($expression),
                 $expression);
    };
}

fn main() {
    foo();
    bar();

    print_result!(1u32 + 1);

    // Recall that blocks are expressions too!
    print_result!({
        let x = 1u32;

        x * x + 2 * x - 1
    });
}

// run the code 
"1u32 + 1" = 2
"{ let x = 1u32; x * x + 2 * x - 1 }" = 2
```
These are some of the available designators:   
https://doc.rust-lang.org/reference/macros-by-example.html



### Macro std::stringify,Stringifies its arguments.
```
macro_rules! stringify {
    ($($t : tt) *) => { ... };
}
```
This macro will yield产生 an expression of type &'static str which is the stringification of all the tokens passed to the macro.    
No restrictions are placed on the syntax of the macro invocation调用 itself.    
Examples：    
```
let one_plus_one = stringify!(1 + 1);
assert_eq!(one_plus_one, "1 + 1");
```

## Overload,重载
Macros can be overloaded to accept different combinations of arguments. In that regard, macro_rules! can work similarly to a match block:   
```
// `test!` will compare `$left` and `$right`
// in different ways depending on how you invoke it:
macro_rules! test {
    // Arguments don't need to be separated by a comma逗号.
    // Any template can be used!
    ($left:expr; and $right:expr) => {
        println!("{:?} and {:?} is {:?}",
                 stringify!($left),
                 stringify!($right),
                 $left && $right)
    };
    // ^ each arm must end with a semicolon.
    ($left:expr; or $right:expr) => {
        println!("{:?} or {:?} is {:?}",
                 stringify!($left),
                 stringify!($right),
                 $left || $right)
    };
}
// 重载的单个参数表达式中不用逗号，match多个参数中才用逗号
fn main() {
    test!(1i32 + 1 == 2i32; and 2i32 * 2 == 4i32);
    test!(true; or false);
}
```
run the code    
"1i32 + 1 == 2i32" and "2i32 * 2 == 4i32" is true
"true" or "false" is true

## Repeat
Macros can use + in the argument list to indicate that an argument may repeat at least once, or *, to indicate that the argument may repeat zero or more times.

In the following example, surrounding the matcher with $(...),+ will match one or more expression, separated by commas. Also note that the semicolon is optional on the last case.
```
// `find_min!` will calculate the minimum of any number of arguments.
macro_rules! find_min {
    // Base case:
    ($x:expr) => ($x);
    // `$x` followed by at least one `$y,`
    ($x:expr, $($y:expr),+) => (
        // Call `find_min!` on the tail `$y`
        std::cmp::min($x, find_min!($($y),+))
    )
}

fn main() {
    println!("{}", find_min!(1u32));
    println!("{}", find_min!(1u32 + 2, 2u32));
    println!("{}", find_min!(5u32, 2u32 * 3, 4u32));
}
```
run the code   
1
2
4

