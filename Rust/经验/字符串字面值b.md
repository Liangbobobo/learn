b"..."	字节字符串字面值; 构造一个 ```[u8]``` 类型而非字符串
b'...'	ASCII 码字节字面值
'...'	字符字面值
```
fn main() {
//依次输出字节字符串字面值    
let a =b"0xF0";
for i in a {
    println!("{}", i );
}

//输出ASCII 码字节字面值
let b=b'a'
 println!("{}", b);
}
 ```