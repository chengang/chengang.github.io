---
title: Perl语言中的协程 ——“Coro”简介
date: 2012-02-27T23:10:22+00:00

---
## 进程、线程和协程

进程是什么应该大家都知道了。我们用ps命令看到的一行就是一个进程。它就是一个独立运行的程序，拥有自己独立的内存空间和信号处理器等一切一个程序该拥有的东西。

线程有点像是一个轻量级的进程。不同的是，线程没有自己独立的内存区域和命名空间，它们所有的东西都是共用的。这意味着如果某线程修改了一个变量，那么其它线程立刻就能看到这个变量的改变。

协程。协程这个词意味着线程之间要互相协作，特别是在CPU资源占用的方面。简单地说，一个CPU同时只能有一个线程可以占用，此时如果另一个线程想要CPU，那么正在运行的线程就必须把CPU让出来。后者可以显式地表示自己希望占用CPU，也可以用等待一个资源（信号量或者IO完成）的方式隐式地来表达。

因为线程比进程轻量，线程模型在脚本语言中是十分流行的（比如python或者ruby，perl里不大流行），而协程模型通常要比线程模型高效很多很多，通常可以达到惊人的数个数量级的效率提升。 



## Perl中的线程

Perl语言的线程一直没有达到成熟的状态。

Perl本身在线程和进程的术语使用上是略显混乱的。目前Perl的线程实现代码其实是它用于Windows平台的Unix进程模拟器的代码，所以实际上它们更像是进程而非线程。最明显的的问题是：在Perl的线程中变量“不是”共享的。



## Coro给我们带来了“协程”

而Coro这个模块带给我们的就是比线程更加高效的协程模型。这样我们就再也不用忍受Perl那蹩脚的线程实现了。



## Coro用法一瞥

像其它Perl的模块一样，想要用Coro，我们需要先use它：

<pre class="brush: perl">use Coro;
</pre>

然后我们就可以用async方法来创建一个协程：

<pre class="brush: perl">async {
      print "hello\n";
   };  
</pre>

async方法的第一个参数是一个语句块，我们也可以在后面继续写其它参数，那些参数会被放在变量@_中传给语句块。

但是如果你就这样把上面的代码保存成文件并执行了，你会发现屏幕上并没有输出&#8217;hello&#8217;。

让我们来解释一下这事。

在我们use Coro之后，我们的主程序就变成了一个协程，和其它协程一样，只不过在程序开始运行的时候它首先占用着CPU。所以虽然我们创建了一个协程，但它只是被放到队列中了，如果主程序不释放对CPU的控制，那么它就没有办法拿到CPU资源去执行它自己。

我们可以使用cede方法来主动释放自己对CPU的占用（在其它协程的实现中此方法通常叫做yield）：

<pre class="brush: perl">use Coro;

   async {
      print "hello\n";
   };  

   cede;
</pre>

这样屏幕上就会输出hello了。



## 更有趣的例子

我们来看一个更有趣的例子：

<pre class="brush: perl">use Coro;

   async {
      print "async 1\n";
      cede;
      print "async 2\n";
   };

   print "main 1\n";
   cede;
   print "main 2\n";
   cede;
</pre>

这段代码会输出：

<pre class="brush: perl">main 1
   async 1
   main 2
   async 2
</pre>

这个例子很好说明了Coro是如何在不用的协程之间协商CPU资源的，其是它带给我们的也就是多路复用的能力。



## 说一点点细节

如果你还感兴趣的话，我们再说说更多细节。

我们传一段代码给async方法之后，它其实做了2个操作：首先创建一个协程，而后将其放入ready队列。

每当一个协程释放了CPU，Coro就会运行调度器（scheduler方法），这个方法会负责找到ready队列中下一个等待CPU资源协程，将它移除出队列并执行它。

cede方法也做了2件事：首先把当前协程推入ready队列，而后运行scheduler。这样同样也会起到释放CPU的效果。事实上，cede的实现完全可以是这样的：

<pre class="brush: perl">sub my_cede {
      $Coro::current->ready;
      schedule;
   }
</pre>

让我们简单说明一下，$Coro::current变量里总是存放着当前正在运行的协程。scheduler方法会调用Coro::schedule。

那么，如果在把自己放入ready队列之前就调用了scheduler会怎样呢？scheduler在ready队列中找到下一个协程弄出来运行。当前的这个协程呢，会sleep到有什么东西把它唤醒，如果有的话。



## 开始Coro之旅

如果你看到这里都还没有放弃的话，那么就赶紧进入[http://search.cpan.org/~mlehmann/Coro-6.07/Coro.pm](http://search.cpan.org/~mlehmann/Coro-6.07/Coro.pm "Coro")开始你的Coro之旅吧！

本文部分翻译自http://search.cpan.org/~mlehmann/Coro-6.07/Coro/Intro.pod，有删节和修改。
