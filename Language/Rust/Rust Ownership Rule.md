**[Rust所有权](https://blog.zongwu233.com/talk-about-rust-ownership/)**
# 1.前言
所有权(Ownership)是 Rust 最与众不同的特性，让Rust实现了保障内存安全以及无GC，并且运行时高性能的目标。一些语言提供垃圾回收机制（garbage collection,GC），例如：Java、Python、Golang、Lisp、Haskell、JavaScript 等等。 另一些语言需要编程人员手工管理内存，进行分配和释放内存。例如：C、C++。

两种机制各有优势和缺点：
1. 带GC的编程语言，自动管理内存，消除人工管理带来的内存管理安全性问题，降低编程语言学习复杂度和使用复杂度，但是带来了额外的运行时性能开销，无法保证高性能和高实时性。
2. 手工管理内存，运行时高性能， 但是增加编程人员的使用心智负担，容易造成内存管理安全性上的bug。

**Rust选择了第三种方式，通过（自己创立的）所有权系统进行内存管理。**

关于所有权系统，编译器会在**编译期**依据所有权规则对代码进行检查。不会给运行期带来额外的开销。（缺点是编译期阶段的时间变的很长，编译后的代码体积会膨胀，就所有权而言，实际上跟范型化的生命周期标记的实现相关）。

# 2. 所有权规则
-   Each value in Rust has a variable that’s called its _owner_.
-   There can only be one owner at a time.
-   When the owner goes out of scope, the value will be dropped.

Rust中的变量值，要么分配在栈上，要么分配在堆上。栈上的数据会随着入栈出栈操作被自动清理，编程语言主要是对堆进行内存管理。

### 2.1 move 特性

如果将一个变量赋值给另外一个变量会如何？考虑下面这种情况：
```rust
fn main() {
    let x = String::from("hello");
    let y = x;
    println!("{}, world!", x);
    println!("{}, world!", y);
}
```

编译就会报错
```rust
error[E0382]: borrow of moved value: `x`
 --> src/main.rs:4:28
  |
2 |     let x = String::from("hello");
  |         - move occurs because `x` has type `String`, which does not implement the `Copy` trait
3 |     let y = x;
  |             - value moved here
4 |     println!("{}, world!", x);
  |                            ^ value borrowed here after move
  |
  = note: this error originates in the macro `$crate::format_args_nl` (in Nightly builds, run with -Z macro-backtrace for more info)

For more information about this error, try `rustc --explain E0382`.
```

在`let y = x;`那行编译器提示`value moved here`，在此处值被move了。

我们对`let` 关键字的理解更加深入：对于形如 `let x = y;` 的语句，如果`y`是变量，**`let` 会引发变量的所有权转移。** 即`let`操作默认是`move`语义。 从而保障了规则2的约束。

### 2.2 Copy 特性

新的问题出现，下面这段代码没有遵循`move`语义，但编译成功：
``` rust
fn main() {
    let x = 1;
    let y = x;
    println!("{}", x);
    println!("{}", y);
}
```

因为考虑到使用上的便利性，在有些场景下并不希望应用默认的`move`语义。**Rust的基础数据类型都实现了 std::marker::Copy `trait`，所以在进行`let`操作时，实际上发生了`copy`。**

哪些类型是满足`Copy`特性的？类似整型这样的存储在栈上的类型、不需要分配内存或某种形式资源的类型。常见的`Copy` 类型如下：
-   所有整数类型，比如 u32。
-   布尔类型，bool，它的值是 true 和 false。
-   所有浮点数类型，比如 f64。
-   字符类型，char。
-   元组，当且仅当其包含的类型也都是 Copy 的时候。比如，(i32, i32) 是 Copy 的，但 (i32, String) 就不是。

### 2.3 Clone 特性
保留原来的变量，并且将值内容拷贝给新的变量，对简单的数字类型很容易理解接受，那如果是复杂类型或者说是自定义类型呢？ 如果不希望发生所有权转移，可以使用`clone`。如下：
```rust
fn main() {
    let x = String::from("hello");
    let y = x.clone();
    println!("{}, world!", x);
    println!("{}, world!", y);
}

```

### 2.4 所有权与函数
调用函数的时候，会将实际参传递给形式参数，这个操作在语义上与给变量赋值相似。可能会移动或者复制，遵循与赋值语句一样的规则。例如：

```rust
fn main() {
    let s = String::from("hello");  // s 进入作用域

    takes_ownership(s);             // s 的值移动到函数里 ...
                                    // ... 所以到这里不再有效

    let x = 5;                      // x 进入作用域

    makes_copy(x);                  // x 应该移动函数里，
                                    // 但 i32 是 Copy 的，所以在后面可继续使用 x

} // 这里, x 先移出了作用域，然后是 s。但因为 s 的值已被移走，
  // 所以不会有特殊操作

fn takes_ownership(some_string: String) { // some_string 进入作用域
    println!("{}", some_string);
} // 这里，some_string 移出作用域并调用 `drop` 方法。占用的内存被释放

fn makes_copy(some_integer: i32) { // some_integer 进入作用域
    println!("{}", some_integer);
} // 这里，some_integer 移出作用域。不会有特殊操作
```


### 2.5 引用与借用

再看如下的代码：
```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() 返回字符串的长度

    (s, length)
}
```

在每一次函数调用的时候，都获取所有权，然后在函数调用完成的时候，再返回所有权，这种方式略显繁琐。而这种调用在实际场景中极其常见，Rust提供了引用（references）功能来解决这个问题。
利用引用，上面的代码可以改写成：

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

传给`calculate_length()`的是`&s1`而不是`s1`。函数的声明是`calculate_length(s: &String)`参数是`s: &String`。

**`&`就是引用。它允许你使用值但不获取其所有权。** 毕竟我们需要遵循规则2。**我们把采用引用作为函数参数称为借用（borrowing）**

这里并不想展开讨论不可变引用和可变引用的相关规则，那是并发编程场景下需要关注的点。

关于引用还需要特别注意的一个点是作用域。一个**引用的作用域**从声明的地方开始一直持续到**最后一次使用为止**。如下代码是可以编译的：

```rust
fn main(){
    let mut s = String::from("hello");

    let r1 = &mut s; // 没问题
    println!("{}", r1); //r1最后使用的地方

    let r2 = &mut s; // 没问题
    println!("{}", r2);
}
```

编译器负责推断引用最后一次使用的代码位置。

如何在合适的时机清理回收内存？当持有堆内存数据的变量离开作用域的时候，堆内存的数据将被`drop`掉。编译器会自动插入`drop`相关的代码，在运行时需要`drop` 的时候调用。

这条规则的重点就是作用域了。

### 2.6 作用域

作用域是一个项（item）在程序中有效的范围。最常见的就是**函数作用域**，比如

```rust
fn main(){// s无效

    let s = "hello world";// s进入作用域
    println!("{}",s);
    
}// s离开作用域
```

还有块作用域，例如：

```rust
fn main(){// s无效

    let s = "hello world";// s进入作用域
    println!("{}",s);
    {
        let a = 1; //a 进入作用域
         println!("{}",a);
    }// a离开作用域
    
    //println!("{}",a); 这里会出错
    
}// s离开作用域
```

实际上，`let`关键字会隐式地开启一个作用域，对于以下代码：

```rust
fn main(){ 

    let s1 = "hello"; 
    let s2 ="world";
    
} 
```

利用 https://play.rust-lang.org/ 的 SHOW MIR 功能可以看到：

```rust
fn main() -> () {
    let mut _0: ();                      // return place in scope 0 at src/main.rs:1:10: 1:10
    let _1: &str;                        // in scope 0 at src/main.rs:3:9: 3:11
    scope 1 {
        debug s1 => _1;                  // in scope 1 at src/main.rs:3:9: 3:11
        let _2: &str;                    // in scope 1 at src/main.rs:4:9: 4:11
        scope 2 {
            debug s2 => _2;              // in scope 2 at src/main.rs:4:9: 4:11
        }
    }

...
```

`main`函数的作用域`scope 0`，在其中，`let s1 = "hello";` 创建一个作用域`scope 1`，然后`let s2 ="world";` 又在`scope 1`里面创建了`scope 2`。

关于引用的特殊作用域问题，上一小节已经说明，这里不再重复。

### 2.7 闭包带来的问题
闭包（closures）可以从环境捕获变量，并在闭包体中使用。有三种使用变量的方式：获取所有权、可变引用、不可变引用。Rust 提供了3种`Fn trait` 以便编译器能够更清晰直接地处理不同场景下闭包对变量所有权的操作问题。

-   FnOnce 获取变量所有权，Once表明闭包不能多次获取同一个变量的所有权，所以只能调用一次。
-   FnMut 获取变量可变的借用值
-   Fn 获取变量的不可变的借用值

注意：`FnOnce`是`FnMut` `Fn`的`supertrait`；`FnMut`是`Fn`的`supertrait`。 `FnOnce`的例子：

```rust
fn do_action<F>(func:F)where F:FnOnce()->usize{
    println!("disappeared variable length : {}", func());
}

pub fn main(){
    let a = String::from("hello world");
    let useless_warpper = move ||->usize{a.len() } ;
    do_action(useless_warpper);
    
    //println!("{}",a);  //这里再使用a会报错
}

```

使用了一个关键字`move` 将闭包需要捕获的变量的所有权`move`到闭包内。 `FnMut`的例子：

```rust
fn do_action<F>(mut func:F)where F:FnMut()->usize{
    println!("mutable result {}", func());
}

pub fn main(){
    let mut a = 12;
    let mut mutable_warpper = ||->usize{ a+=2;a } ;
    do_action(mutable_warpper);
    
    println!("now {}",a);
}
```