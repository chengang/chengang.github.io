---
title: '[rust]使struct可被打印'
date: 2015-06-03T00:49:38+00:00

---
这块变化挺大的。
  
在1.0里的做法是实现Struct的Trait——std::fmt::Display，代码如下：

<pre>use std::fmt;

struct Info {
    s: String,
}

impl fmt::Display for Info {
    fn fmt(&self, f:&mut fmt::Formatter) -> fmt::Result {
        write!(f, "Info.s = {}", self.s)
    }
}

fn main() {
    let bar = Info {s: "foobar".to_string() };
    println!("b:{}", bar);
}
</pre>

一般来说打印struct这就够了。
  
但是如果这个struct还会被包含在别的struct里面的话，可能需要用到Debug打印，也就是{:?}。
  
代码如下：

<pre>use std::fmt;

struct Info {
    s: String,
}

impl fmt::Debug for Info {
    fn fmt(&self, f:&mut fmt::Formatter) -> fmt::Result {
        write!(f, "Info.s = {}", self.s)
    }
}

fn main() {
    let bar = Info {s: "foobar".to_string() };
    println!("b:{:?}", bar);
}
</pre>
