---
title: Rust官方指南摘抄（八）并发任务
date: 2015-01-06T18:13:30+00:00

---

## 并发原语

1、Rust使用“任务”来描述并发。任务是轻量级的原语，抛弃了不安全的内存共享模式，使用消息传递作为交流手段。

2、任务是作为一个库被实现的，并不是语言的一部分。所以今后可能会出现更多的并发库去适应各种并发环境。

3、任务使用spawn()方法来表达。

<pre>spawn(proc() {
    println!("Hello from a task!");
});
</pre>

4、spawn接收一个proc，凡是proc用过的变量，往后都不能使用了。
  
这是为了并发安全而考虑的，虽然其他很多语言允许这种用，但是Rust决定禁止这种方法，并选择在编译器直接报错。

<pre>let mut x = vec![1i, 2i, 3i];

spawn(proc() {
    println!("The value of x[0] is: {}", x[0]);
});

println!("The value of x[0] is: {}", x[0]); // 错误：x已经被借走了
</pre>

## 消息传递

5、如果任务想和外面通信，可以使用频道（channel）。
  
很像linux里的pipe是不。

<pre>let (tx, rx) = channel();

spawn(proc() {
    tx.send("Hello from a task!".to_string());
});

let message = rx.recv();
println!("{}", message);
</pre>

注意.recv()方法是阻塞的，它的非阻塞版本是.try_recv()。

6、和pipe一样，如果需要双向通信。
  
那就需要两个频道。

<pre>let (tx1, rx1) = channel();
let (tx2, rx2) = channel();

spawn(proc() {
    tx1.send("Hello from a task!".to_string());
    let message = rx2.recv();
    println!("{}", message);
});

let message = rx1.recv();
println!("{}", message);

tx2.send("Goodbye from main!".to_string());
</pre>

7、Sender都是Receiver泛型定义的，所以频道可以传输各种数据类型。
  
但只能是一种数据类型，如果往一个频道里先传了int，后续又传string是不行的。

## Future

1、从基本的原语可以衍生出很多并发模型，Rust的标准库中也有一些。

2、比如说频道之外的另一种沟通方式Future。

<pre>use std::sync::Future;

let mut delayed_value = Future::spawn(proc() { // mut关键字不可或缺。因为计算出结果后，需要重新赋值
    // just return anything for examples' sake

    12345i
});

// do staff...

println!("value = {}", delayed_value.get()); //跑到需要值的这句的时候，如果proc已经跑完了，就直接拿到值了。如果还没跑完，会block到跑完。
</pre>

## 错误处理

1、fail!宏可以帮助我们传递失败消息。

<pre>spawn(proc() {
    fail!("Nope.");
});
</pre>

2、失败的任务不可恢复。但是它可以提醒其它任务它失败了。
  
task::try会帮助我们。

<pre>use std::task;
use std::rand;

let result = task::try(proc() { // 成功或者失败会存在result变量里
    if rand::random() {
        println!("OK");
    } else {
        fail!("oops!");
    }
});
</pre>

## 完
