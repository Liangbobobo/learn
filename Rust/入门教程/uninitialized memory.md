# Working With Uninitialized Memory
All runtime-allocated memory in a Rust program begins its life as uninitialized.    
Rust provides mechanisms to work with uninitialized memory in checked (safe) and unchecked (unsafe) ways.   


# Checked Uninitialized Memory
Like C, all stack variables in Rust are uninitialized until a value is explicitly assigned to them.     
Unlike C, Rust statically prevents you from ever reading them until you do
```
fn main() {
    let x: i32;
    println!("{}", x);
}

  |
3 |     println!("{}", x);
  |                    ^ use of possibly uninitialized `x`
```


# Drop Flags 
However types with destructors are a different story: Rust needs to know whether to call a destructor whenever a variable is assigned to, or a variable goes out of scope.   
Note that this is not a problem that all assignments need worry about. In particular, assigning through a dereference unconditionally drops, and assigning in a let unconditionally doesn't drop:   
```
let mut x = Box::new(0); // let makes a fresh variable, so never need to drop
let y = &mut x;
*y = Box::new(1); // Deref assumes the referent is initialized, so always drops

```

## This is only a problem when overwriting a previously initialized variable or one of its subfields.

It turns out that Rust actually tracks whether a type should be dropped or not at runtime. As a variable becomes initialized and uninitialized, a drop flag for that variable is toggled. When a variable might need to be dropped, this flag is evaluated to determine if it should be dropped.

Of course, it is often the case that a value's initialization state can be statically known at every point in the program. If this is the case, then the compiler can theoretically generate more efficient code! For instance, straight- line code has such static drop semantics:    
```

let mut x = Box::new(0); // x was uninit; just overwrite.
let mut y = x;           // y was uninit; just overwrite and make x uninit.
x = Box::new(0);         // x was uninit; just overwrite.
y = x;                   // y was init; Drop y, overwrite it, and make x uninit!
                         // y goes out of scope; y was init; Drop y!
                         // x goes out of scope; x was uninit; do nothing.
```

Similarly, branched code where all branches have the same behavior with respect to关于 initialization has static drop semantics:   
```

let mut x = Box::new(0);    // x was uninit; just overwrite.
if condition {
    drop(x)                 // x gets moved out; make x uninit.
} else {
    println!("{}", x);
    drop(x)                 // x gets moved out; make x uninit.
}
x = Box::new(0);            // x was uninit; just overwrite.
                            // x goes out of scope; x was init; Drop x!
```
??   
However code like this requires runtime information to correctly Drop:   
```

let x;
if condition {
    x = Box::new(0);        // x was uninit; just overwrite.
    println!("{}", x);
}
                            // x goes out of scope; x might be uninit;
                            // check the flag!
```

Of course, in this case it's trivial to retrieve static drop semantics:   
```
if condition {
    let x = Box::new(0);
    println!("{}", x);
}
```
The drop flags are tracked on the stack and no longer stashed in types that implement drop.??

# Unchecked Uninitialized Memory
One interesting exception to this rule is working with arrays.   
Safe Rust doesn't permit you to partially initialize an array.When you initialize an array, you can either set every value to the same thing with ```let x = [val; N]```, or you can specify each member individually with ```let x = [val1, val2, val3]```.    
Unfortunately this is pretty rigid, especially if you need to initialize your array in a more incremental增加 or dynamic way.    

Unsafe Rust gives us a powerful tool to handle this problem: MaybeUninit. This type can be used to handle memory that has not been fully initialized yet.  




# arrary