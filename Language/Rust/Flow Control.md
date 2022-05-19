流程控制结构包括：
-   if条件判断结构
-   loop循环
-   while循环
-   for..in迭代

需要说明的是，Rust中这些结构都是表达式，它们都有默认的返回值`()`，且if结构和loop循环结构可以指定返回值。
> 注：【这些结构的默认返回值是`()`】的说法是不严谨的
> 之所以可以看作是默认返回`()`，是因为Rust会在每个分号结尾的语句后自动加上小括号`()`，使得语句看上去也有了返回值。 为了行文简洁，下文将直接描述为默认返回值。

## 1. if
The basic grammar of  `if` , you could notice that COND needn't  a parenthesis to be wrapped, at the same time it must be the bool value, not any others.
``` rust
if COND1 {
  ...
} else if COND2 {
  ...
} else {
  ...
}
```

The if structure is a expression, so it could be assigned to a variable.
1. Every branch need have return value at last line，and not ending up with semicolon.
2. 2.The return values must be the same type .
``` rust
let x = if real { 1.1_f32 } else { 2.2_f32 };
println!("{}", x);
```

if there is no else branch within "if structure", if or else if branch will return default value '()', and it leads to a compile error that the variable y returns a mismatched type

``` rust 
let y = if real {
	1.1
};
--------------
error[E0317]: `if` may be missing an `else` clause
  --> src/main.rs:31:13
   |
31 |       let y = if real {
   |  _____________^
32 | |         1.1
   | |         --- found here
33 | |     };
   | |_____^ expected `()`, found floating-point number
   |
   = note: `if` expressions without `else` evaluate to `()`
   = help: consider adding an `else` block that evaluates to the expected type

For more information about this error, try `rustc --explain E0317`.
```

## 2.while
simple grammar, the rule of COND is the same as the `if`
```rust
while COND { ... }
```

if you wanna break up the while , use the `break` keyword.
if you wanna go into the next while immediately , use the `continue` keyword.
``` rust
let mut x = 0;
while x < 5 {
	x += 1;
	if x == 3 {
		println!("x is {}, continue!", x);
		continue;
	}
	if x == 4 {
		println!("x is {}, break!", x);
		break;
	}
	println!("x is {}", x);
}
```

ususlly we don't care about the return value of `while`

## 3.loop
1. 无限循环，可以使用`break`关键字终止循环。
2. 同时loop也有默认返回值`()`，作为表达式也可以将其赋值给变量。
3. 在`loop`中使用break，可以指定一个loop的返回值
```rust
let mut count = 0;
loop {
	if count ==5 {
		break;
	}
	println!("count is {}", count);
	count += 1;
}
```

## 4.for
1. Rust中的for只具备迭代功能。迭代是一种特殊的循环，每次从数据的集合中取出一个元素是一次迭代过程，直到取完所有元素，才终止迭代。
2. Range类型是支持迭代的数据集合，Slice类型也是支持迭代的数据集合。
3. Rust数组不支持迭代？？？
```rust
let array = [1, 2, 3, 4, 5];
for item in array {
	println!("The item is {}", item);
}
```