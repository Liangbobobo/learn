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


