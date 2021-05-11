# trait：定义共享的行为
trait 告诉 Rust 编译器某个特定类型拥有可能与其他类型共享的功能。  
通过 trait 以一种抽象的方式定义共享的行为。  
可以使用 trait bounds 指定泛型是任何拥有特定行为的类型。  
trait 类似于其他语言中的常被称为 接口（interfaces）的功能，虽然有一些不同

## 定义 trait
一个类型的行为由其可供调用的方法构成。如果可以对不同类型调用相同的方法的话，这些类型就可以共享相同的行为了。  
trait 定义是一种将方法签名组合起来的方法，目的是定义一个实现某些目的所必需的行为的集合。   
```
pub trait Summary {
    fn summarize(&self) -> String;
}
Summary trait 定义，它包含由 summarize 方法提供的行为
使用 trait 关键字来声明一个 trait，后面是 trait 的名字，
大括号中声明描述实现这个 trait 的类型所需要的行为的方法签名
在这个例子中是 fn summarize(&self) -> String。

在方法签名后跟分号，而不是在大括号中提供其实现。
接着每一个实现这个 trait 的类型都需要提供其自定义行为的方法体，
编译器也会确保任何实现 Summary trait 的类型都拥有与这个签名的定义完全一致的 summarize 方法

trait 体中可以有多个方法：一行一个方法签名且都以分号结尾。
```

现在我们定义了 Summary trait,接着可以在类型上使用trait   
```
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
在类型上实现 trait 类似于实现与 trait 无关的方法。
区别在于 impl 关键字之后，我们提供需要实现 trait 的名称，接着是 for 和需要实现 trait 的类型的名称。
在 impl 块中，使用 trait 定义中的方法签名，不过不再后跟分号，
而是需要在大括号中编写函数体来为特定类型实现 trait 方法所拥有的行为。
```