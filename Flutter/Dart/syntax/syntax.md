# Important concepts：  
所有变量引用的都是 对象，每个对象都是一个 类 的实例。数字、函数以及 null 都是对象。所有的类都继承于 Object 类。  

尽管 Dart 是强类型语言，但是在声明变量时指定类型是可选的，因为 Dart 可以进行类型推断。在上述代码中，变量 number 的类型被推断为 int 类型。如果想显式地声明一个不确定的类型，可以使用特殊类型 dynamic。  

Dart 没有类似于 Java 那样的 public、protected 和 private 成员访问限定符。如果一个标识符以下划线 (_) 开头则表示该标识符在库内是私有的。可以查阅 库和可见性 获取更多相关信息。  

标识符 可以以字母或者下划线 (_) 开头，其后可跟字符和数字的组合。  

##  Variables  变量  
```
var name = 'Bob';  
变量仅存储对象的引用。这里名为 name 的变量存储了一个 String 类型对象的引用，“Bob” 则是该对象的值。

name 变量的类型被推断为 String，但是你可以为其指定类型。如果一个对象的引用不局限于单一的类型，可以根据设计指南将其指定为 Object 或 dynamic 类型。  
dynamic name = 'Bob';  
除此之外你也可以指定类型：  

String name = 'Bob';

```

## Default value  默认值  
在 Dart 中，未初始化的变量拥有一个默认的初始化值：null。即便数字也是如此，因为在 Dart 中一切皆为对象，数字也不例外。  

在 Dart 中，未初始化的变量拥有一个默认的初始化值：null。即便数字也是如此，因为在 Dart 中一切皆为对象，数字也不例外。  

## Final and const  
如果你不想更改一个变量，可以使用关键字 final 或者 const 修饰变量，这两个关键字可以替代 var 关键字或者加在一个具体的类型前。一个 final 变量只可以被赋值一次；一个 const 变量是一个编译时常量（const 变量同时也是 final 的）。顶层的 final 变量或者类的 final 变量在其第一次使用的时候被初始化。
1. 实例变量可以是 final 的但不可以是 const 的， final 实例变量必须在构造器开始前被初始化，比如在声明实例变量时初始化，或者作为构造器参数，或者将其置于构造器的 初始化列表中。
2. 使用关键字 const 修饰变量表示该变量为 编译时常量。如果使用 const 修饰类中的变量，则必须加上 static 关键字，即 static const（注意：顺序不能颠倒（译者注））。在声明 const 变量时可以直接为其赋值，也可以使用其它的 const 变量为其赋值
3. const 关键字不仅仅可以用来定义常量，还可以用来创建 常量值，该常量值可以赋予给任何变量。你也可以将构造函数声明为 const 的，这种类型的构造函数创建的对象是不可改变的。``` var foo = const [];  
final bar = const [];
const baz = []; // 相当于 `const []` (Equivalent to `const []`)```   如果使用初始化表达式为常量赋值可以省略掉关键字 const，比如上面的常量 baz 的赋值就省略掉了 const。
4. 没有使用 final 或 const 修饰的变量的值是可以被更改的，即使这些变量之前引用过 const 的值。  
5. 你可以在常量中使用 类型检查和强制类型转换 (is 和 as)、 集合中的 if 以及 展开操作符 (... 和 ...?)：  
```
const Object i = 3; // Where i is a const Object with an int value...
const list = [i as int]; // Use a typecast.
const map = {if (i is int) i: "int"}; // Use is and collection if.
const set = {if (list is List<int>) ...list}; // ...and a spread.
```

## Built-in types 内置类型  
可以直接使用字面量来初始化上述类型。例如 'This is a string' 是一个字符串字面量，true 是一个布尔字面量。

由于 Dart 中每个变量引用都指向一个对象（一个 类 的实例），你通常也可以使用 构造器 来初始化变量。一些内置的类型有它们自己的构造器。例如你可以使用 Map() 来创建一个 map 对象。


### Numbers  
Dart 支持两种 Number 类型：int，double  
1.整数值；长度不超过 64位，具体取值范围依赖于不同的平台。在 DartVM 上其取值位于 -263 至 263 - 1 之间。编译成 JavaScript 的 Dart 使用 JavaScript 数字，其允许的取值范围在 -253 至 253 - 1 之间。  
2. double；64位的双精度浮点数字，且符合 IEEE 754 标准。

int 和 double 都是 num 的子类。 num 中定义了一些基本的运算符比如 +、-、*、/ 等，还定义了 abs()、ceil() 和 floor() 等方法（位运算符，比如 » 定义在 int 中）。如果 num 及其子类不满足你的要求，可以查看 dart:math 库中的 API。

整数是不带小数点的数字，如果一个数字包含了小数点，那么它就是浮点型的。整型字面量将会在必要的时候自动转换成浮点数字面量

字符串和数字之间转换的方式：
```
// String -> int
var one = int.parse('1');
assert(one == 1);

// String -> double
var onePointOne = double.parse('1.1');
assert(onePointOne == 1.1);

// int -> String
String oneAsString = 1.toString();
assert(oneAsString == '1');

// double -> String
String piAsString = 3.14159.toStringAsFixed(2);
assert(piAsString == '3.14');
```

整型支持传统的位移操作，比如移位（<<、>>）、按位与（&）、按位或（ 	），例如：
```
assert((3 << 1) == 6); // 0011 << 1 == 0110
assert((3 >> 1) == 1); // 0011 >> 1 == 0001
assert((3 | 4) == 7); // 0011 | 0100 == 0111
```

数字字面量为编译时常量。很多算术表达式只要其操作数是常量，则表达式结果也是编译时常量。

## string 
Dart 字符串是 UTF-16 (unicode)编码的字符序列。可以使用单引号或者双引号来创建字符串：
```
var s1 = '使用单引号创建字符串字面量。';
var s2 = "双引号也可以用于创建字符串字面量。";
var s3 = '使用单引号创建字符串时可以使用斜杠来转义那些与单引号冲突的字符串：\'。';
var s4 = "而在双引号中则不需要使用转义与单引号冲突的字符串：'";
```

可以在字符串中以 ${表达式} 的形式使用表达式，如果表达式是一个标识符，可以省略掉 {}。如果表达式的结果为一个对象，则 Dart 会调用该对象的 toString 方法来获取一个字符串。
```
var s = '字符串插值';

assert('Dart 有$s，使用起来非常方便。' == 'Dart 有字符串插值，使用起来非常方便。');
assert('使用${s.substring(3,5)}表达式也非常方便' == '使用插值表达式也非常方便。');
```

== 运算符判断两个对象的内容是否一样，如果两个字符串包含一样的字符编码序列，则表示相等。

可以使用 + 运算符将两个字符串连接为一个，也可以将多个字符串挨着放一起变为一个

可以使用三个单引号或者三个双引号创建多行字符串：
```
var s1 = '''
你可以像这样创建多行字符串。
''';

var s2 = """这也是一个多行字符串。""";
```

在字符串前加上 r 作为前缀创建 “raw” 字符串（即不会被做任何处理（比如转义）的字符串）：```var s = r'在 raw 字符串中，转义字符串 \n 会直接输出 “\n” 而不是转义为换行。'; 

字符串字面量是一个编译时常量，只要是编译时常量都可以作为字符串字面量的插值表达式：
```
// 可以将下面三个常量作为字符串插值拼接到字符串字面量中。(These work in a const string.)
const aConstNum = 0;
const aConstBool = true;
const aConstString = 'a constant string';

// 而下面三个常量则不能作为字符串插值拼接到字符串字面量。
var aNum = 0;
var aBool = true;
var aString = 'a string';
const aConstList = [1, 2, 3];
```

## Booleans  
Dart 使用 bool 关键字表示布尔类型，布尔类型只有两个对象 true 和 false，两者都是编译时常量。
Dart 的类型安全不允许你使用类似 if (nonbooleanValue) 或者 assert (nonbooleanValue) 这样的代码检查布尔值。相反，你应该总是显示地检查布尔值，比如像下面的代码这样：
```
// 检查是否为空字符串 (Check for an empty string).
var fullName = '';
assert(fullName.isEmpty);

// 检查是否小于等于零。
var hitPoints = 0;
assert(hitPoints <= 0);

// 检查是否为 null。
var unicorn;
assert(unicorn == null);

// 检查是否为 NaN。
var iMeantToDoThis = 0 / 0;
assert(iMeantToDoThis.isNaN);
```

## Lists
在 Dart 中数组由 List 对象表示。通常称之为 List。