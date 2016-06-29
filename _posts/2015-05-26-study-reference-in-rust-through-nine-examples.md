---
title: 九个例程认识Rust中的引用
date: 2015-05-26T16:41:45+00:00
layout: post
---
为方便理解，先简述一下Rust中的所有权概念：

### 一个变量同一时刻只能有一个拥有者。

完整的概念参看Rust Book: https://doc.rust-lang.org/stable/book/ownership.html。

## 例一

通过引用把变量借给方法，方法结束后，会自动把变量的拥有权归还给主逻辑。

<pre>fn main() {
    fn yyy(a: &mut String)  {
        a.push_str("push_in_fn");
    }

    let mut b = String::new();
    b.push_str("before[");
    yyy(&mut b);
    b.push_str("]after");

    println!("123{}456", b);
}
</pre>

## 例二

例一之堆版本。

<pre>fn main() {
    fn yyy(a: &mut String)  {
        a.push_str("push_in_fn");
    }

    let mut b = Box::new( String::new());
    b.push_str("before[");
    yyy(&mut *b);
    //yyy(&mut b); this also accpeted.
    b.push_str("]after");

    println!("123{}456", b);
}
</pre>

## 例三

引用没有归还之前，原变量不可使用。否则报错。

<pre>fn main() {
    fn yyy(a: &mut String)  {
        a.push_str("push_in_fn");
    }

    let mut b = Box::new( String::new());
    b.push_str("before[");
    let c = &b; // 这里变量b已经借出了
    yyy(&mut b); // 报错：这里c还没有归还b，把b交给方法yyy会导致编译期错误。
    b.push_str("]after");

    println!("123{}456", b);
}
</pre>

## 例四

例三正确版。
  
给c一个作用域，超出作用域时c会自动归还b。

<pre>fn main() {
    fn yyy(a: &mut String)  {
        a.push_str("push_in_fn");
    }

    let mut b = Box::new( String::new());
    b.push_str("before[");
    {
        let c = &b;
    }
    yyy(&mut b);
    b.push_str("]after");

    println!("123{}456", b);
}
</pre>

## 例五

例四基础上另一个多次借出的错误示范。

<pre>fn main() {
    fn yyy(a: &mut String)  {
        a.push_str("push_in_fn");
    }

    let mut b = Box::new( String::new());
    b.push_str("before[");
    {
        let c = &mut b; // 删掉这句就可以正确运行了。
        let d = &mut b;
        d.push_str("asd");
    }
    yyy(&mut b);
    b.push_str("]after");

    println!("123{}456", b);
}
</pre>

## 例六

函数是可以返回引用的。

<pre>struct MyStruct {
    a: u32,
    b: u32,
}

fn main() {
    fn add (var: &MyStruct) -> &u32 {
        return &var.b;
    }

    let foo = MyStruct {a: 100, b: 99};

    let c = add(&foo);

    println!("a:{}, b:{}, c:{}", foo.a, foo.b, c);
}
</pre>

## 例七

但是返回的引用必须是传入的结构体里的东西。这样可以保证只要传入的变量活着，返回的变量也会一直有效。
  
因为如果在函数内部声明一个变量，返回它的引用的话，出了这个函数那个变量也被回收了。
  
像下面这样是不行的。

<pre>struct MyStruct {
    a: u32,
    b: u32,
}

fn main() {
    fn add (var: &MyStruct) -> &u32 {
        let foo = var.a + var.b;
        return &foo; // 这里会报错：`foo` does not live long enough
    }

    let foo = MyStruct {a: 100, b: 99};

    let c = add(&foo);

    println!("a:{}, b:{}, c:{}", foo.a, foo.b, c);
}
</pre>

## 例八

但是如果传入的变量有两个，编译器就不知道把返回的指针的生命周期和其中哪个绑定了。
  
于是报错，等我们解决。

<pre>struct MyStruct {
    a: u32,
    b: u32,
}

fn main() {
    fn add (var: &MyStruct, var2: &MyStruct) -> &u32 { // 这里会报错。因为传了2个带有生命周期的指针进来，编译器不知道绑定哪个了。
        return &var.a;
    }

    let foo = MyStruct {a: 100, b: 99};
    let goo = MyStruct {a: 50, b: 49};

    let c = add(&foo, &goo);

    println!("a:{}, b:{}, c:{}", foo.a, foo.b, c);
}
</pre>

## 例九

例八正确版。
  
使用“命名生命周期”解决例八的问题，告诉编译器绑定第一个变量。
  
这里第一次出现了“&#8217;a”这个东西，一下就出现了三次。
  
它就是“命名生命周期”。

出现的这三次可以这样理解：
  
第一次：声明&#8217;a，我这个函数接下来会用到；
  
第二次：给&#8217;a赋值，&#8217;a等于第一个参数的生命周期；
  
第三次：把&#8217;a的值赋给返回，返回的引用的声明周期等于&#8217;a。
  
这样就把返回的引用生命周期和第一个传入的参数绑在一起了。

&#8216;a也可以是&#8217;b，&#8217;c，&#8217;d，a-z都可以用的。

改成下面这样就可以运行了。

<pre>struct MyStruct {
    a: u32,
    b: u32,
}

fn main() {
    fn add&lt;'a> (var: &'a MyStruct, var2: &MyStruct) -> &'a u32 {
        return &var.a;
    }

    let foo = MyStruct {a: 100, b: 99};
    let goo = MyStruct {a: 50, b: 49};

    let c = add(&foo, &goo); // 标记A

    println!("a:{}, b:{}, c:{}", foo.a, foo.b, c);
}
</pre>

请这样理解&#8217;a：它和var一样也是函数的一个参数，同样是由调用者赋值的。
  
在这里例子里面，在函数被调用的那一刻（标记A）&#8217;a就被foo赋值了。
