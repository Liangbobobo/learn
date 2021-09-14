https://doc.rust-lang.org/nomicon/ownership.html   
# ownership and lifetimes

## the motivation of this design


## References
There are two kinds of reference:

    1.Shared reference: &
    2.Mutable reference: &mut

Which obey the following rules:   

    1.A reference cannot outlive its referent
    2.A mutable reference cannot be aliased（可变引用不能被别名引用）

# Aliasing
First off首先, let's get some important caveats（警告） out of the way（注意事项）:   
1. We will be using the broadest possible definition of aliasing for the sake of discussion（为了讨论）.   
Rust's definition will probably be more restricted to factor（因素） in mutations and liveness.

2. We will be assuming a single-threaded, interrupt-free无中断, execution.    
We will also be ignoring things like memory-mapped hardware.    
 Rust assumes these things don't happen unless you tell it otherwise. For more details, see the Concurrency Chapter.   

With that said, here's our working definition: variables and pointers alias if they refer to overlapping regions of memory.  

## Why Aliasing Matters
```
fn compute(input: &u32, output: &mut u32) {
    if *input > 10 {
        *output = 1;
    }
    if *input > 5 {
        *output *= 2;
    }
}

We would like to be able to optimize it to the following function:

fn compute(input: &u32, output: &mut u32) {
    let cached_input = *input; // keep *input in a register
    if cached_input > 10 {
        *output = 2;  // x > 10 implies x > 5, so double and exit immediately
    } else if cached_input > 5 {
        *output *= 2;
    }
}

```

In Rust, this optimization should be sound. For almost any other language, it wouldn't be (barring global analysis). This is because the optimization relies on knowing that aliasing doesn't occur, which most languages are fairly liberal with. Specifically, we need to worry about function arguments that make input and output overlap, such as compute(&x, &mut x).

```
                    //  input ==  output == 0xabad1dea
                    // *input == *output == 20
if *input > 10 {    // true  (*input == 20)
    *output = 1;    // also overwrites *input, because they are the same
}
if *input > 5 {     // false (*input == 1)
    *output *= 2;
}
                    // *input == *output == 1
```
Our optimized function would produce *output == 2 for this input, so the correctness of our optimization relies on this input being impossible.   

In Rust we know this input should be impossible because &mut isn't allowed to be aliased. So we can safely reject its possibility and perform this optimization. In most other languages, this input would be entirely possible, and must be considered.

we used the fact that &mut u32 can't be aliased to prove that writes to *output can't possibly affect *input. This let us cache *input in a register, eliminating a read.

By caching this read, we knew that the write in the > 10 branch couldn't affect whether we take the > 5 branch, allowing us to also eliminate a read-modify-write (doubling *output) when *input > 10.

## alias analysis is important it lets the compiler perform useful optimizations! Some examples:
1. keeping values in registers by proving no pointers access the value's memory   
通过证明没有指针访问值的内存来将值保存在寄存器中

2. eliminating reads by proving some memory hasn't been written to since last we read it   
通过证明自上次读取以来尚未写入某些内存来消除读取

3. eliminating writes by proving some memory is never read before the next write to it   
通过证明某些内存在下一次​​写入之前从未被读取来消除写入

4. moving or reordering reads and writes by proving they don't depend on each other  
通过证明它们不相互依赖来移动或重新排序读取和写入


These optimizations also tend to prove the soundness of bigger optimizations such as loop vectorization, constant propagation, and dead code elimination.

The key thing to remember about alias analysis is that writes are the primary hazard for optimizations.   
That is, the only thing that prevents us from moving a read to any other part of the program is the possibility of us re-ordering it with a write to the same location.  
也就是说，阻止我们将读取移动到程序的任何其他部分的唯一原因是我们可以通过写入到同一位置来重新排序它。

## 通过将写入放在最后，来消除重排影响
For instance, we have no concern for aliasing in the following modified version of our function, because we've moved the only write to *output to the very end of our function. This allows us to freely reorder the reads of *input that occur before it:   
例如，在我们的函数的以下修改版本中，我们不担心别名，因为我们已将对 *output 的唯一写入移动到我们函数的最后。这允许我们自由地重新排序发生在它之前的 *input 读取：   
```
fn compute(input: &u32, output: &mut u32) {
    let mut temp = *output;
    if *input > 10 {
        temp = 1;
    }
    if *input > 5 {
        temp *= 2;
    }
    *output = temp;
}
```
We're still relying on alias analysis to assume that temp doesn't alias input, but the proof is much simpler: the value of a local variable can't be aliased by things that existed before it was declared. This is an assumption every language freely makes, and so this version of the function could be optimized the way we want in any language.   
我们仍然依赖别名分析来假设 temp 不会对输入进行别名，但证明要简单得多：局部变量的值不能被声明之前存在的变量使用别名。这是每种语言都自由做出的假设，因此可以在任何语言中以我们想要的方式优化此版本的函数。

This is why the definition of "alias" that Rust will use likely involves some notion of liveness and mutation: we don't actually care if aliasing occurs if there aren't any actual writes to memory happening.   
这就是为什么 Rust 将使用的“别名”的定义可能涉及一些outlive和mutable的概念：如果没有任何实际的内存写入发生，我们实际上并不关心是否发生别名。   

Of course, a full aliasing model for Rust must also take into consideration things like function calls (which may mutate things we don't see), raw pointers (which have no aliasing requirements on their own), and UnsafeCell (which lets the referent of an & be mutated).  
当然，一个完整的 Rust 别名模型还必须考虑到函数调用（可能会改变我们看不到的东西）、原始指针（它们本身没有别名要求）和 UnsafeCell（允许引用对象的 & 被突变）。  



# Lifetimes
Rust enforces these rules（ownership） through lifetimes.   
Lifetimes are named regions of code that a reference must be valid for.     
Those regions may be fairly complex, as they correspond to paths of execution in the program.   
There may even be holes in these paths of execution, as it's possible to invalidate a reference as long as it's reinitialized before it's used again.（这些执行路径中甚至可能存在漏洞，因为只要在再次使用之前重新初始化引用，就有可能使引用无效。）    
Types which contain references (or pretend to) may also be tagged with lifetimes so that Rust can prevent them from being invalidated as well.   

In most of our examples, the lifetimes will coincide with scopes. This is because our examples are simple. The more complex cases where they don't coincide are described below.   

Within a function body, Rust generally doesn't let you explicitly name the lifetimes involved. This is because it's generally not really necessary to talk about lifetimes in a local context;   
Many anonymous scopes and temporaries that you would otherwise have to write are often introduced to make your code Just Work.

### However once you cross the function boundary, you need to start talking about lifetimes.
Lifetimes are denoted表示 with an apostrophe: 'a, 'static.     

One particularly interesting piece of sugar is that each let statement implicitly introduces a scope.    
For the most part, this doesn't really matter. However it does matter for variables that refer to each other.     

```
let x = 0;
let y = &x;
let z = &y;

The borrow checker always tries to minimize the extent of a lifetime,
so it will likely desugar to the following:
// NOTE: `'a: {` and `&'b x` is not valid syntax!
'a: {
    let x: i32 = 0;
    'b: {
        // lifetime used is 'b because that's good enough.
        let y: &'b i32 = &'b x;
        'c: {
            // ditto on 'c
            let z: &'c &'b i32 = &'c y;
        }
    }
}


```
Actually passing references to outer scopes will cause Rust to infer a larger lifetime:   
```
let x = 0;
let z;
let y = &x;
z = y;

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

## Example: references that outlive referents
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

This signature of as_str takes a reference to a u32 with some lifetime, and promises that it can produce a reference to a str that can live just as long. Already we can see why this signature might be trouble. That basically implies that we're going to find a str somewhere in the scope the reference to the u32 originated in, or somewhere even earlier. That's a bit of a tall order.   

### 为什么上面返回一个引用在rust中是错误的
We then proceed to compute the string s, and return a reference to it. Since the contract of our function says the reference must outlive 'a, that's the lifetime we infer for the reference. Unfortunately, s was defined in the scope 'b, so the only way this is sound is if 'b contains 'a -- which is clearly false since 'a must contain the function call itself. We have therefore created a reference whose lifetime outlives its referent, which is literally the first thing we said that references can't do. The compiler rightfully blows up in our face.

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
        }
    }
}
```

### the right way to write this function is as follows:
```

fn to_string(data: &u32) -> String {
    format!("{}", data)
}
```

We must produce an owned value inside the function to return it! The only way we could have returned an &'a str would have been if it was in a field of the &'a u32, which is obviously not the case.

(Actually we could have also just returned a string literal, which as a global can be considered to reside at the bottom of the stack; though this limits our implementation just a bit.)
（实际上，我们也可以只返回一个字符串文字，它作为一个全局变量可以被认为位于堆栈的底部；尽管这会稍微限制我们的实现。）


## aliasing a mutable reference
```

let mut data = vec![1, 2, 3];
let x = &data[0];
data.push(4);
println!("{}", x);

desugar:
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

The problem here is a bit more subtle and interesting. We want Rust to reject this program for the following reason: We have a live shared reference x to a descendant of data when we try to take a mutable reference to data to push. This would create an aliased mutable reference, which would violate违反 the second rule of references.

However this is not at all how Rust reasons that this program is bad.       
Rust doesn't understand that x is a reference to a subpath of data. It doesn't understand Vec at all. What it does see is that x has to live for 'b to be printed. The signature of Index::index subsequently demands that the reference we take to data has to survive for 'b. When we try to call push, it then sees us try to make an &'c mut data. Rust knows that 'c is contained within 'b, and rejects our program because the &'b data must still be alive!


Here we see that the lifetime system is much more coarse than the reference semantics we're actually interested in preserving. For the most part, that's totally ok, because it keeps us from spending all day explaining our program to the compiler. However it does mean that several programs that are totally correct with respect to Rust's true semantics are rejected because lifetimes are too dumb.


## The area covered by a lifetime
