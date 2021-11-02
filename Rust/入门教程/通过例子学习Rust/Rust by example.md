https://doc.rust-lang.org/stable/rust-by-example/index.html
# 2. Primitives
Scalar Types标量类型：  
1. char Unicode scalar values like 'a', 'α' and '∞' (4 bytes each)
2. and the unit type (), whose only possible value is an empty tuple: ()    
Despite the value of a unit type being a tuple, it is not considered a compound type because it does not contain multiple values. 
等等详细查看原文

Compound Types 符合类型：   
1. arrays like [1, 2, 3]
2. tuples like (1, true)
Variables can always be type annotated.    
Numbers may additionally be annotated via a suffix or by default. Integers default to i32 and floats to f64    
Note that Rust can also infer types from context
```
fn main() {
    // Variables can be type annotated.
    let logical: bool = true;

    let a_float: f64 = 1.0;  // Regular annotation
    let an_integer   = 5i32; // Suffix annotation

    // Or a default will be used.
    let default_float   = 3.0; // `f64`
    let default_integer = 7;   // `i32`
    
    // A type can also be inferred from context 
    let mut inferred_type = 12; // Type i64 is inferred from another line
    inferred_type = 4294967296i64;
    
    // A mutable variable's value can be changed.
    let mut mutable = 12; // Mutable `i32`
    mutable = 21;
    
    // Error! The type of a variable can't be changed.
    mutable = true;
    
    // Variables can be overwritten with shadowing.
    let mutable = true;
}
```

## 2.1 Literals and operators
## 2.2 Tuples 
A tuple is a collection of values of different types.也可是类型相同的values
Functions can use tuples to return multiple values, as tuples can hold any number of values.   
```
// Tuples can be used as function arguments and as return values
fn reverse(pair: (i32, bool)) -> (bool, i32) {
    // `let` can be used to bind the members of a tuple to variables
    let (integer, boolean) = pair;

    (boolean, integer)
}

// The following struct is for the activity.
#[derive(Debug)]
struct Matrix(f32, f32, f32, f32);

fn main() {
    // A tuple with a bunch of different types
    let long_tuple = (1u8, 2u16, 3u32, 4u64,
                      -1i8, -2i16, -3i32, -4i64,
                      0.1f32, 0.2f64,
                      'a', true);

    // Values can be extracted from the tuple using tuple indexing
    println!("long tuple first value: {}", long_tuple.0);
    println!("long tuple second value: {}", long_tuple.1);

    // Tuples can be tuple members
    let tuple_of_tuples = ((1u8, 2u16, 2u32), (4u64, -1i8), -2i16);

    // Tuples are printable
    println!("tuple of tuples: {:?}", tuple_of_tuples);
    
    // But long Tuples cannot be printed  ？？？
    // let too_long_tuple = (1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13);
    // println!("too long tuple: {:?}", too_long_tuple);
    // TODO ^ Uncomment the above 2 lines to see the compiler error

    let pair = (1, true);
    println!("pair is {:?}", pair);

    println!("the reversed pair is {:?}", reverse(pair));

    // To create one element tuples, the comma is required to tell them apart
    // from a literal surrounded by parentheses
    println!("one element tuple: {:?}", (5u32,));
    println!("just an integer: {:?}", (5u32));

    //tuples can be destructured to create bindings
    let tuple = (1, "hello", 4.5, true);

    let (a, b, c, d) = tuple;
    println!("{:?}, {:?}, {:?}, {:?}", a, b, c, d);

    let matrix = Matrix(1.1, 1.2, 2.1, 2.2);
    println!("{:?}", matrix);

}
```

## 2.3 Arrays and Slices
An array is a collection of objects of the same type T, stored in contiguous memory.    
Arrays are created using brackets [], and their length, which is known at compile time, is part of their type signature [T; length].   

Slices are similar to arrays, but their length is not known at compile time.    
Instead, a slice is a two-word object, the first word is a pointer to the data, and the second word is the length of the slice.    
The word size is the same as usize, determined by the processor architecture eg 64 bits on an x86-64. Slices can be used to borrow a section of an array, and have the type signature``` &[T]```.