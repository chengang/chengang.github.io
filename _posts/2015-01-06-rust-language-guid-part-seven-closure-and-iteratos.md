---
title: Rust官方指南摘抄（七）闭包和迭代器
date: 2015-01-06T17:22:48+00:00

---

## 闭包

1、Rust中的匿名函数被称为闭包。

<pre>let add_one = |x| { 1i + x };

println!("The sum of 5 plus 1 is {}.", add_one(5i));
</pre>

2、闭包可以读取它被定义的那个作用域内的变量，比如下面这个不接受参数的闭包。

<pre>fn main() {
    let x = 5i;

    let printer = || { println!("x is: {}", x); };

    printer(); // prints "x is: 5"
}
</pre>

3、闭包借走了它从环境中读取的变量，在它归还之间，要遵守Rust的拥有者原则。

<pre>fn main() {
    let mut x = 5i;

    let printer = || { println!("x is: {}", x); };

    x = 6i; // 这里会报错，因为x已经被前面的闭包借走了
}

</pre>

4、闭包特性可以帮助我们获取函数式编程两大优点之一高阶函数（另一个是惰性计算），造出“返回函数的函数”。

<pre>fn twice(x: int, f: |int| -> int) -> int {
    f(x) + f(x)
}

fn main() {
    let square = |x: int| { x * x };

    twice(5i, square); // 结果是50
}
</pre>

<pre>fn twice(x: int, f: |int| -> int) -> int {
    f(x) + f(x)
}

fn main() {
    twice(5i, |x: int| { x * x }); // 结果是50
}

</pre>

## 过程

1、Rust中的闭包还有一种形式被称为过程，使用proc关键字来定义。

<pre>let x = 5i;

let p = proc() { x * x };
println!("{}", p()); // prints 25
</pre>

2、过程是为并发而设计的，我们会在并发的章节继续谈到它。目前我们只需要知道，同一个过程不可以被调用两次。

<pre>let x = 5i;

let p = proc() { x * x };
println!("{}", p());
println!("{}", p()); // 第二次调用会报错
</pre>

## 迭代器

1、rang函数的返回，就是一个迭代器。所谓迭代器，都可以调用.next()方法。

<pre>for x in range(0i, 10i) {
    println!("{:d}", x);
}
</pre>

其实等价于显式的.next()调用如下：

<pre>let mut range = range(0i, 10i);

loop {
    match range.next() {
        Some(x) => {
            println!("{}", x);
        },
        None => { break }
    }
}
</pre>

2、rang()在Rust中是反模式的。下面这种写法并不被提倡。

<pre>let nums = vec![1i, 2i, 3i];

for i in range(0u, nums.len()) {
    println!("{}", nums[i]);
}
</pre>

下面这样写更好一些。代码更清晰，而且执行效率更高。
  
因为我们不再依赖vec的索引，.iter()帮助我们避免了不必要的边界检查。

<pre>let nums = vec![1i, 2i, 3i];

for num in nums.iter() {
    println!("{}", num);
}
</pre>

3、值得注意的是，迭代器给我们的是数据的引用而不是数据的拷贝。
  
它临时借来了数据，避免了数据拷贝的开销。
  
当然，无论把哪个交给println!()，它都会帮我们搞定它。

## 消费者

1、进一步阐明为什么range()是反模式的，我们需要明确三个概念。
  
a> 迭代器：返回一组数据；（range()就是这类）
  
b> 迭代适配器：作用在迭代器之上，返回一个数据被处理过的新的迭代器；
  
c> 消费者：作用在迭代器之上，返回最终数据。

2、消费者之一 collect()，最常用的消费者，把迭代器里的数据弄成一组返回回来。
  
需要指明数据类型。

<pre>let one_to_one_hundred = range(0i, 100i).collect::&lt;Vec&lt;int>>();
</pre>

3、消费者之二 find()，找出符合条件的值。它可以接受一个闭包。

<pre>let greater_than_forty_two = range(0i, 100i)
                             .find(|x| *x >= 42);

match greater_than_forty_two {
    Some(_) => println!("We got some numbers!"),
    None    => println!("No numbers found :("),
}
</pre>

4、消费者之三 fold()。带状态存储的迭代器消费者。

<pre>let total = range(1i, 100i)
              .fold(0i, |sum, x| sum + x);
</pre>

就这段代码而言。
  
每一轮的计算结果会存放在sum里面。最终total变量的值等于 fold中第一个参数0与迭代器中所有值的合。

5、借助迭代器和消费者，我们可以实现函数式编程的另一个重要优点“惰性计算”。
  
值只在真正被下一个流程需要的时候才真正被生成和计算。
  
这使得我们直接申请一个1到无穷大的数组而完全不用考虑内存和CPU资源的问题。

6、按照上面说的，再来一个高级迭代器。

<pre>std::iter::count(1i, 5i); // 返回从1、6、11、16、21直到无限
</pre>

## 迭代适配器

1、迭代适配器之一 filter()。

<pre>// 会输出1~100之间所有偶数
for i in range(1i, 100i).filter(|x| x % 2 == 0) {
    println!("{}", i); 
}
</pre>

2、迭代适配器之二 map()。

<pre>range(1i, 100i).map(|x| println!("{}", x));
</pre>

但这会得到warning

<pre>src/hello_world.rs:2:2: 2:45 warning: unused result which must be used: iterator adaptors are lazy and do nothing unless consumed, #[warn(unused_must_use)] on by default
src/hello_world.rs:2 	range(1i, 100i).map(|x| println!("{}", x));
                     	^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
</pre>

因为是惰性的，迭代器要求后续操作一定要明确把值取走。

3、迭代适配器之三 take()。下例明确取5个。

<pre>for i in std::iter::count(1i, 5i).take(5) {
    println!("{}", i);
}

// 输出
// 1
// 6 
// 11
// 16
// 21
</pre>

4、我们可以让迭代器打头、迭代适配器放在中间，消费者放在最后结合起来用。就像下面这个例子。

<pre>range(1i, 1000i)
    .filter(|x| x % 2 == 0)
    .filter(|x| x % 3 == 0)
    .take(5)
    .collect::&lt;Vec&lt;int>>(); // 我们会得到一个6, 12, 18, 24, 30的Vector
</pre>
