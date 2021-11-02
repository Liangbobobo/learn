2021.9.18 真TM难     
https://doc.rust-lang.org/nomicon/lifetimes.html
# Lifetimes
Rust enforces these rules through lifetimes，this is the onely reason why we need lifetimes:      
### Lifetimes are named regions of code that a reference must be valid for.

There may even be holes in these paths of execution, as it's possible to invalidate a reference as long as it's reinitialized before it's used again.    
Types which contain references (or pretend to) may also be tagged with lifetimes so that Rust can prevent them from being invalidated as well.    
生命周期适用于引用以及所有包含引用的地方，比如含有引用的type


### In most of our examples, the lifetimes will coincide with scopes. This is because our examples are simple. 当出现 anonymous scopes and temporaries时，lifetime变得复杂，that is  once you cross the function boundary, you need to start talking about lifetimes. 

Within a function body, Rust generally doesn't let you explicitly name the lifetimes involved. This is because it's generally not really necessary to talk about lifetimes in a local context; Rust has all the information and can work out everything as optimally as possible.      
Many anonymous scopes and temporaries that you would otherwise have to write are often introduced to make your code Just Work.   


###  each let statement implicitly introduces a scope. 
For the most part, this doesn't really matter. However it does matter for variables that refer to each other.     
The borrow checker always tries to minimize the extent of a lifetime   
Actually passing references to outer scopes will cause Rust to infer a larger lifetime    
```

let x = 0;
let z;
let y = &x;
z = y;

desugar:

'a: {
    let x: i32 = 0;
    'b: {
        let z: &'b i32;
        'c: {
            // Must use 'b here because this reference is
            // being passed to that scope.
            let y: &'b i32 = &'b x;
            z = y;
        }
    }
}
```

## references that outlive referents
```

fn as_str(data: &u32) -> &str {
    let s = format!("{}", data);
    &s
}

desugars to:

fn as_str<'a>(data: &'a u32) -> &'a str {
    'b: {
        let s = format!("{}", data);
        return &'a s;
    }
}
```
This signature of as_str takes a reference to a u32 with some lifetime, and promises that it can produce a reference to a str that can live just as long.   
That basically implies基本上意味着 that we're going to find a str somewhere in the scope the reference to the u32 originated in, or somewhere even earlier. That's a bit of a tall order.   
```
fn as_str<'a>(data: &'a u32) -> &'a str {
    'b: {
        let s = format!("{}", data);
        return &'a s
    }
}

fn main() {
    'c: {
        let x: u32 = 0;
        'd: {
            // An anonymous scope is introduced because the borrow does not
            // need to last for the whole scope x is valid for. The return
            // of as_str must find a str somewhere before this function
            // call. Obviously not happening.
            println!("{}", as_str::<'d>(&'d x));
            由于as_str处于单独的{} scope内，x在此scope无效
        }
    }
}
```
after let s 语句之后
We then proceed to compute the string s, and return a reference to it.       
Since the contract of our function says the reference must outlive 'a, that's the lifetime we infer for the reference. Unfortunately, s was defined in the scope 'b, so the only way this is sound is if 'b contains 'a -- which is clearly false since 'a must contain the function call itself. We have therefore created a reference whose lifetime outlives its referent, which is literally the first thing we said that references can't do. The compiler rightfully blows up in our face.

Of course, the right way to write this function is as follows:   
```
fn to_string(data: &u32) -> String {
    format!("{}", data)
}
```
We must produce an owned value inside the function to return it! The only way we could have returned an &'a str would have been if it was in a field of the &'a u32, which is obviously not the case.    
(Actually we could have also just returned a string literal, which as a global can be considered to reside at the bottom of the stack; though this limits our implementation just a bit.)??

## aliasing a mutable reference
```
let mut data = vec![1, 2, 3];
let x = &data[0];
data.push(4);
println!("{}", x);

desugars:

'a: {
    let mut data: Vec<i32> = vec![1, 2, 3];
    'b: {
        // 'b is as big as we need this borrow to be
        // (just need to get to `println!`)
        let x: &'b i32 = Index::index::<'b>(&'b data, 0);
        'c: {
            // Temporary scope because we don't need the
            // &mut to last any longer.
            Vec::push(&'c mut data, 4);
        }
        println!("{}", x);
    }
}
```
### The reason Rust reject this program:   
1. violate the reference can not alias   
We have a live shared reference x to a descendant of data when we try to take a mutable reference to data to push. This would create an aliased mutable reference

2. Rust doesn't understand that x is a reference to a subpath of data. It doesn't understand Vec at all. What it does see is that x has to live for 'b to be printed. The signature of Index::index subsequently demands that the reference we take to data has to survive for 'b. When we try to call push, it then sees us try to make an &'c mut data. Rust knows that 'c is contained within 'b, and rejects our program because the &'b data must still be alive!

Here we see that the lifetime system is much more coarse than the reference semantics we're actually interested in preserving. For the most part, that's totally ok, because it keeps us from spending all day explaining our program to the compiler. However it does mean that several programs that are totally correct with respect to Rust's true semantics are rejected because lifetimes are too dumb.


## The area covered by a lifetime 生命周期覆盖
### A reference (sometimes called a borrow) is alive from the place it is created to its last use. 
The borrowed thing needs to outlive only borrows that are alive. This looks simple, but there are few subtleties微妙.    
```
this is ok

let mut data = vec![1, 2, 3];
let x = &data[0];
println!("{}", x);
// This is OK, x is no longer needed
data.push(4);
```
after printing x, it is no longer needed, so it doesn't matter if it is dangling or aliased (even though the variable x technically exists to the very end of the scope).

However, if the value has a destructor, the destructor is run at the end of the scope. And running the destructor is considered a use ‒ obviously the last one. So, this will not compile.
```

#[derive(Debug)]
struct X<'a>(&'a i32);

impl Drop for X<'_> {
    fn drop(&mut self) {}
}

let mut data = vec![1, 2, 3];
let x = X(&data[0]);
println!("{:?}", x);
data.push(4);
// Here, the destructor is run and therefore this'll fail to compile.

```
One way to convince the compiler that x is no longer valid is by using drop(x) before data.push(4).

Furthermore, there might be multiple possible last uses of the borrow, for example in each branch of a condition.
```
let mut data = vec![1, 2, 3];
let x = &data[0];

if some_condition() {
    println!("{}", x); // This is the last use of `x` in this branch
    data.push(4);      // So we can push here
} else {
    // There's no use of `x` in here, so effectively the last use is the
    // creation of x at the top of the example.
    data.push(5);
}
```


### And a lifetime can have a pause in it.生命周期内部可以暂停
Or you might look at it as two distinct borrows just being tied to the same local variable. This often happens around loops (writing a new value of a variable at the end of the loop and using it for the last time at the top of the next iteration).

```

let mut data = vec![1, 2, 3];
// This mut allows us to change where the reference points to
let mut x = &data[0];

println!("{}", x); // Last use of this borrow
data.push(4);
x = &data[3]; // We start a new borrow here
println!("{}", x);
```


# Limits of Lifetimes


# Lifetime Elision
每一个输出的生命周期，是self或者输入的生命周期，否则就是临时变量的周期，是错误的。表述可能不对，参见rust程序设计
Elision rules are as follows:   
1. Each elided lifetime in input position becomes a distinct lifetime parameter.每一个被省略的输入生命周期都有一个不同的生命周期参数
2. If there is exactly one input lifetime position (elided or not), that lifetime is assigned to all elided output lifetimes.如果只有一个输入位置，且该位置只有一个生命周期，那么这个生命周期将分配给所有的输出生命周期
3. If there are multiple input lifetime positions, but one of them is &self or &mut self, the lifetime of self is assigned to all elided output lifetimes.


Otherwise, it is an error to elide an output lifetime.


# Unbounded Lifetimes
Unsafe code can often end up最终 producing references or lifetimes out of thin air.不安全的代码最终会凭空产生引用和liftime， Such lifetimes come into the world as unbounded.这样的生命周期是无限的。     
The most common source来源 of this is dereferencing a raw pointer, which produces a reference with an unbounded lifetime.Such a lifetime becomes as big as context demands.      
This is in fact more powerful than simply becoming 'static事实上unbound比 'stactic更具有破坏力, because for instance &'static &'a T will fail to typecheck, but the unbound lifetime will perfectly mold into &'a &'a T as needed.     
However for most intents and purposes, such an unbounded lifetime can be regarded as 'static.
Almost no reference is 'static, so this is probably wrong. transmute and transmute_copy are the two other primary offenders.   
One should endeavor to bound an unbounded lifetime as quickly as possible, especially across function boundaries.人们应该尽其所快的努力限制unbounded lifetime   
```
any output lifetimes that don't derive from inputs are unbounded.
fn get_str<'a>() -> &'a str;
```
this will produce an &str with an unbounded lifetime.
The easiest way to avoid unbounded lifetimes is to use lifetime elision at the function boundary. If an output lifetime is elided, then it must be bounded by an input lifetime.    
Of course it might be bounded by the wrong lifetime, but this will usually just cause a compiler error, rather than allow memory safety to be trivially violated.

Within a function, bounding lifetimes is more error-prone容易出错.   
The safest and easiest way to bound a lifetime is to return it from a function with a bound lifetime.    
However if this is unacceptable, the reference can be placed in a location with a specific lifetime. Unfortunately it's impossible to name all lifetimes involved in a function.


# Higher-Rank高阶 Trait Bounds (HRTBs)




https://doc.rust-lang.org/nomicon/obrm.html    
# The Perils危险 Of Ownership Based Resource Management (OBRM)
OBRM AKA RAII: Resource Acquisition Is Initialization.The most common "resource" this pattern manages is simply memory. Box, Rc, and basically everything in std::collections is a convenience to enable correctly managing memory.     
Roughly speaking the pattern is as follows:     
to acquire a resource, you create an object that manages it. To release the resource, you simply destroy the object, and it cleans up the resource for you.      
Which is the point, really: Rust is about control.真正的重点是，Rust就是控制   
However we are not limited to just memory.然而不仅仅是刚才提到的内存 Pretty much every other system resource like a thread, file, or socket is exposed暴露 through this kind of API.其他一些系统资源比如 线程、文件、套接字都是通过这些APi暴露的。

## Constructors
There is exactly one way to create an instance of a user-defined type: name it, and initialize all its fields at once.创建一个用户自定义实列的唯一方式是，命名，并初始化他的所有字段。   
Every other way you make an instance of a type is just calling a totally vanilla普通 function that does some stuff工作 and eventually bottoms out见底 to The One True Constructor.

Unlike C++, Rust does not come with a slew of大量 built-in kinds of constructor. There are no Copy, Default, Assignment, Move, or whatever constructors. The reasons for this are varied, but it largely boils down to Rust's philosophy of being explicit.

Move constructors are meaningless in Rust because we don't enable types to "care" about their location in memory. ？？怎么理解？  简单的如整形是直接复制的，复杂的如box是new初始化，drop释放内存的，也就是说Rust中的类型值和其所在的内存这两者之间的操作是分开的，值有单独的trait来控制，所在的内存有new和drop来控制，Rust is control   
Every type must be ready for it to be blindly memcopied to somewhere else in memory. This means pure on-the-stack-but- still-movable intrusive linked lists are simply not happening in Rust (safely).

Assignment and copy constructors similarly don't exist because move semantics are the only semantics in Rust. At most x = y just moves the bits of y into the x variable. Rust does provide two facilities for providing C++'s copy- oriented semantics: Copy and Clone. Clone is our moral equivalent of a copy constructor, but it's never implicitly invoked. You have to explicitly call clone on an element you want to be cloned. Copy is a special case of Clone where the implementation is just "copy the bits". Copy types are implicitly cloned whenever they're moved, but because of the definition of Copy this just means not treating the old copy as uninitialized -- a no-op.

While Rust provides a Default trait for specifying the moral equivalent of a default constructor, it's incredibly rare for this trait to be used. This is because variables aren't implicitly initialized. Default is basically only useful for generic programming. In concrete contexts, a type will provide a static new method for any kind of "default" constructor. This has no relation to new in other languages and has no special meaning. It's just a naming convention.

## Destructors
