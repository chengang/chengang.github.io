---
id: 4668
title: '[rust]三个方法解决error: use of moved value'
date: 2015-06-04T00:02:14+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=4668
permalink: /4668.html
categories:
  - 备忘
tags:
  - borrow
  - Debug
  - Drop
  - 'error: use of moved value'
  - impl
  - RefCell
  - rust
  - std::rc:Rc
  - use of moved value
---
## 概述

&#8220;error: use of moved value&#8221;，相信最近开始玩rust的同学看到这个报错都能会心一笑了。

rust做到了不依赖运行期垃圾回收的安全内存管理，但这个特别爽快的特性也引入了一些不便，这报错就是常见的麻烦之一。

这报错要是想展开说清楚，需要完整解释rust的ownership、borrowing、lifetime等概念，那是一篇太长的文章。

我们先暂时放下概念，用三个不同的方法动手解决这个报错。

## 错误

我们以下面这段程序为基础展开我们的讨论，这里面主要定义的就是一个Info结构体。

<pre>struct Info {
    s: String,
}

fn fn_a(info: Info) {
    println!("in fn_a");
}

fn main() {
    let foo = Info {s : "abc".to_string() };
    fn_a(foo);
}
</pre>

首先，我们要制造出报错“use of moved value”。
  
很简单，我们只需要以foo为参数再调用一次fn_a()就好。

<pre>struct Info {
    s: String,
}

fn fn_a(info: Info) {
    println!("in fn_a");
}

fn main() {
    let foo = Info {s : "abc".to_string() };
    fn_a(foo);
    fn_a(foo); // 只有这行是新加入的
}
</pre>

现在，我们得到了编译器报错。

<pre>src/main.rs:12:10: 12:13 error: use of moved value: `foo`
src/main.rs:12     fn_a(foo);
                        ^~~
src/main.rs:11:10: 11:13 note: `foo` moved here because it has type `Info`, which is non-copyable
src/main.rs:11     fn_a(foo);
                        ^~~
</pre>

编译器说，我们新加入的行里用了“移动了的值”。
  
啥叫“移动了的值”呢？
  
说白了就是用过了的值，foo已经给第一个fn\_a()用过了，到了第二个fn\_a()的时候就是moved value了。然后就不让用了。
  
至于为什么要制定这样的规则，我另撰文解释。

现在我们开始动手来解决这个问题。

## 方法一：引用

因为我们传给函数的是value，所以value被move了。
  
我们可以通过传引用给函数来解决这问题。
  
代码如下：

<pre>struct Info {
    pub s: String,
}

fn fn_a(info: &mut Info) {
    info.s = "bbb".to_string();
}

fn main() {
    let mut foo = Info {s : "abc".to_string() };
    println!("1: {}", foo.s);
    fn_a(&mut foo);
    println!("2: {}", foo.s);
    fn_a(&mut foo);
    println!("3: {}", foo.s);
}
</pre>

运行结果如下：

<pre>1: abc
2: bbb
3: bbb
</pre>

我们把这段程序用impl改写一下。
  
下面这段程序和上面的程序其实是等价的：

<pre>struct Info {
    pub s: String,
}

impl Info {
    fn fn_a(&mut self) {
        self.s = "bbb".to_string();
    }
}

fn main() {
    let mut foo = Info {s : "abc".to_string() };
    println!("1: {}", foo.s);
    foo.fn_a();
    println!("2: {}", foo.s);
    foo.fn_a();
    println!("3: {}", foo.s);
}
</pre>

运行结果显然是一样的：

<pre>1: abc
2: bbb
3: bbb
</pre>

改写成这样之后是不是很眼熟了呢。

没错，大多数的标准库就是用的这种方法来避免moved value问题的。

impl里面的第一个参数其实就是struct的引用，所以我们用struct.fn()这种写法的时候，传递给方法实际都是引用。

## 方法二：引用计数

rust在标准库里提供了引用计数，它是另一个可以解决move value的方法。
  
先列出来代码吧。

<pre>use std::rc::Rc;
use std::cell::RefCell;

struct Info {
    s: String,
}

impl Info {
    fn new(a: &str) -> Info {
        Info {
            s: a.to_string(),
        }
    }
}

fn abc(a: Rc&lt;RefCell&lt;Info>>) {
    a.borrow_mut().s = "bbbbb".to_string();
}

fn main() {
    let bar = Rc::new(RefCell::new(Info::new("abc")));
    println!("1 : {}", bar.borrow().s);
    abc(bar.clone());
    println!("2 : {}", bar.borrow().s);
    abc(bar.clone());
    println!("3 : {}", bar.borrow().s);
}
</pre>

这段代码有点稍稍复杂。
  
其实需要注意的地方只有两个：
  
1、使用了Rc之后，传递变量都要使用.clone()方法来增加引用计数；减少引用计数不用管，rust会根据作用域自己搞定；
  
2、变量不用声明为mut了。使用的时候，如果不更改，使用.borrow()方法得到真正的struct；如果需要更改，则使用.borrow_mut()方法得到真正的struct。

另外还有两点需要再说明一下：
  
1、如果变量根本不需要改变，则不用套里面的RefCell::new()那层；
  
2、如果涉及多线程之间的传参，要放弃Rc，使用Arc。

## 方法三：实现Trait Clone

从方法二中可以看到，我们使用了Rc，Rc就帮助我们实现了一个.clone()方法，让我们得以避免错误。

那我们能不能自己实现.clone()呢，答案当然是肯定的。

Rc做的是把数据和计数都放到了堆上，它提供的.clone()实际是对计数的复制。

我们不搞那么复杂，我们做个简单点的，我们直接做对struct的复制。

代码如下：

<pre>struct Info {
    s: i32,
}

impl Info {
    fn new(a: i32) -> Info {
        Info {
            s: a,
        }
    }
}

impl Clone for Info {
    fn clone(&self) -> Info {
        Info {s: self.s}
    }
}

fn abc(a: Info) -> Info {
    Info {s: a.s + 1}
}

fn main() {
    let mut foo = Info::new(111);
    println!("1 : {}", foo.s);
    abc(foo);
    println!("2 : {}", foo.s);
    abc(foo);
    println!("3 : {}", foo.s);
}
</pre>

运行结果：

<pre>1 : 111
2 : 111
3 : 111
</pre>

为什么我们这里没有写foo.clone()而直接写foo编译器也没抱怨什么呢？

我们可以看看最初编译器说的话：note: \`foo\` moved here because it has type \`Info\`, which is non-copyable

它说，foo因为不能复制，所以它被move了。

所以当我们让Info变得可复制了之后，它就不会被move了。

当然要写成foo.clone()，那也是没问题的。

但是注意因为我们的值是复制进去的，所以最原始的foo不会被改变了。

## 最后

以上我们用了三种方法解决了move value的报错。

实际还有很多种方法可以用于在不同的场景中避免这个错误。

有兴趣的同学可以继续玩耍，最后提供一个跟踪变量生命周期的方法以供愉快玩耍。

实现Drop Trait在变量被释放或重分配空间时打印日志，代码如下：

<pre>use std::fmt;
use std::rc::Rc;
use std::cell::RefCell;

struct Info {
    s: String,
}

impl Info {
    fn new(a: &str) -> Info {
        println!{"new:{}", a};
        Info {
            s: a.to_string(),
        }
    }
}

impl Drop for Info {
    fn drop(&mut self) {
        println!("drop:{}", self.s);
    }
}

impl fmt::Display for Info {
    fn fmt(&self, f:&mut fmt::Formatter) -> fmt::Result {
        write!(f, "Info.s = {}", self.s)
    }
}

impl fmt::Debug for Info {
    fn fmt(&self, f:&mut fmt::Formatter) -> fmt::Result {
        write!(f, "Info.s = {}", self.s)
    }
}

fn main() {
    let foo = Info::new("abc");
    let mut bar = Rc::new(RefCell::new(Info::new("abc")));
    println!("1 : {:?}", bar);
    println!("code exit");
}
</pre>