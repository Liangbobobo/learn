# 哈希 map（hash map）
HashMap<K, V> 类型储存了一个键类型 K 对应一个值类型 V 的映射。它通过一个 哈希函数（hashing function）来实现映射，决定如何将键和值放入内存中。

哈希 map 可以用于需要任何类型作为键来寻找数据的情况，而不是像 vector 那样通过索引

## 新建一个哈希 map
哈希 map 将它们的数据储存在堆上  

注意必须首先 use 标准库中集合部分的 HashMap。在这三个常用集合中，HashMap 是最不常用的，所以并没有被 prelude 自动引用。标准库中对 HashMap 的支持也相对较少，例如，并没有内建的构建宏。

类似于 vector，哈希 map 是同质的：所有的键必须是相同类型，值也必须都是相同类型。
1. 可以使用 new 创建一个空的 HashMap，并使用 insert 增加元素
```

use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```



2. 另一个构建哈希 map 的方法是使用一个元组的 vector 的 collect 方法，
```

use std::collections::HashMap;

let teams  = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
collect 方法可以将数据收集进一系列的集合类型
可以使用 zip 方法来创建一个元组的 vector，其中 “Blue” 与 10 是一对，依此类推。接着就可以使用 collect 方法将这个元组 vector 转换成一个 HashMap.

```

这里 HashMap<_, _> 类型注解是必要的，因为可能 collect 很多不同的数据结构，而除非显式指定否则 Rust 无从得知你需要的类型。但是对于键和值的类型参数来说，可以使用下划线占位，而 Rust 能够根据 vector 中数据的类型推断出 HashMap 所包含的类型。

## 哈希 map 和所有权
对于像 i32 这样的实现了 Copy trait 的类型，其值可以拷贝进哈希 map。对于像 String 这样拥有所有权的值，其值将被移动而哈希 map 会成为这些值的所有者

```
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// 这里 field_name 和 field_value 不再有效，
// 尝试使用它们看看会出现什么编译错误！
```
当 insert 调用将 field_name 和 field_value 移动到哈希 map 中后，将不能使用这两个绑定。  
如果将值的引用插入哈希 map，这些值本身将不会被移动进哈希 map。但是这些引用指向的值必须至少在哈希 map 有效时也是有效的。

## 访问哈希 map 中的值
可以通过 get 方法并提供对应的键来从哈希 map 中获取值
```
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name);
```

score 是与蓝队分数相关的值，应为 Some(10)。因为 get 返回 Option<V>，所以结果被装进 Some；如果某个键在哈希 map 中没有对应的值，get 会返回 None。这时就要用某种第六章提到的方法之一来处理 Option。

可以使用与 vector 类似的方式来遍历哈希 map 中的每一个键值对，也就是 for 循环：
```
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

## 更新哈希 map
尽管键值对的数量是可以增长的，不过任何时候，每个键只能关联一个值。当我们想要改变哈希 map 中的数据时，必须决定如何处理一个键已经有值了的情况。可以选择完全无视旧值并用新值代替旧值。可以选择保留旧值而忽略新值，并只在键 没有 对应值时增加新值。或者可以结合新旧两值。让我们看看这分别该如何处理！

### 覆盖一个值

如果我们插入了一个键值对，接着用相同的键插入一个不同的值，与这个键相关联的旧值将被替换。即便示例 8-24 中的代码调用了两次 insert，哈希 map 也只会包含一个键值对，因为两次都是对蓝队的键插入的值：
```

use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

这会打印出 {"Blue": 25}。原始的值 10 则被覆盖了。
```

### 只在键没有对应值时插入

我们经常会检查某个特定的键是否有值，如果没有就插入一个值。为此哈希 map 有一个特有的 API，叫做 entry，它获取我们想要检查的键作为参数。entry 函数的返回值是一个枚举，Entry，它代表了可能存在也可能不存在的值。比如说我们想要检查黄队的键是否关联了一个值。如果没有，就插入值 50，对于蓝队也是如此。
```
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
```
Entry 的 or_insert 方法在键对应的值存在时就返回这个值的可变引用，如果不存在则将参数作为新值插入并返回新值的可变引用。这比编写自己的逻辑要简明的多，另外也与借用检查器结合得更好。

### 根据旧值更新一个值:  
```

use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map);

这会打印出 {"world": 2, "hello": 1, "wonderful": 1}，or_insert 方法事实上会返回这个键的值的一个可变引用（&mut V）。这里我们将这个可变引用储存在 count 变量中，所以为了赋值必须首先使用星号（*）解引用 count。这个可变引用在 for 循环的结尾离开作用域，这样所有这些改变都是安全的并符合借用规则。
```

## 哈希函数
HashMap 默认使用一种 “密码学安全的”（“cryptographically strong” ）1 哈希函数，它可以抵抗拒绝服务（Denial of Service, DoS）攻击。然而这并不是可用的最快的算法，不过为了更高的安全性值得付出一些性能的代价。如果性能监测显示此哈希函数非常慢，以致于你无法接受，你可以指定一个不同的 hasher 来切换为其它函数。hasher 是一个实现了 BuildHasher trait 的类型。第十章会讨论 trait 和如何实现它们。你并不需要从头开始实现你自己的 hasher；crates.io 有其他人分享的实现了许多常用哈希算法的 hasher 的库。