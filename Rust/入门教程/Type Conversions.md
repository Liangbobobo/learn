https://doc.rust-lang.org/nomicon/conversions.html
# Type Conversions 类型转换
There are two common problems with typing bits:    
1. needing to reinterpret those exact bits as a different type
2. needing to change the bits to have equivalent meaning for a different type

# Rust consequently gives you several ways to solve them：   
一、 the ways that Safe Rust gives you to reinterpret values. The most trivial简单 way to do this is to just destructure a value into its constituent构成 parts and then build a new type out of them.    
二、 Coercions   
Types can implicitly be coerced to change in certain contexts.    
These changes are generally just weakening of types, largely focused around pointers and lifetimes.    
They mostly exist to make Rust "just work" in more cases, and are largely harmless.

https://doc.rust-lang.org/reference/type-coercions.html#coercion-types   
Type coercions are implicit operations that change the type of a value.   
They happen automatically at specific locations and are highly restricted in what types actually coerce.   
Any conversions allowed by coercion can also be explicitly performed by the type cast operator, as.
## Coercion sites 强制转换的位置
A coercion can only occur at certain coercion sites in a program   
these are typically places where the desired type is explicit所需的类型是显式的 or can be derived by propagation from explicit types (without type inference没有类型推断)：    
1. 


