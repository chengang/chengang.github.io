---
title: 《Advandced Perl Programming》第七章、POE(1)
date: 2009-10-09T17:52:28+00:00
layout: post
---
这一章，我们一起来看看被Mark-Jason Dominus称为“Perl5里最有意思的进化”的Perl Object Environment。POE有很多用途，列举一些如下：

  * 提供了一个堪比线程和IPC的多任务调度机制；
  * 简化了以协议为基础的网络服务器和客户端的开发；
  * 提供了一个状态机框架；
  * 把复杂程序中等待I/O的麻烦抽象出来。

就像我们看到的，POE很难概括。但POE最重要的一点是，它尝试着把在事件驱动环境中编程的恼人细节给隐藏起来。

## 在事件驱动环境中编程

如果你使用过TK或者GTK编写过图形应用，你应该知道那比编写平常的程序要麻烦一些。平常编程时，你只需要输入一个顺序执行的指令序列，然后交给计算机去执行。GUI程序略有不同，它先是设置一个环境（比如说一个窗口），然后对其上发生的事件作出响应（比如点击标题栏或选择一个菜单项）。这种工作方式就叫做事件驱动环境。

不只是GUI是事件驱动的。比如说，一个服务器程序，显然它也不只是序列化执行一系列指令。它等待来自客户端的连接（一个事件），然后根据输入做出回应。当客户端完成工作断开连接，它就回到等待状态等待下一次连接。同样，也可以用一个程序来监视一个目录里每一个文件的变化，并做出处理。

事件驱动的核心是事件循环，也可以叫做主循环（main loop）。TK有，Event模块有，POE作为一个处理事件驱动环境编程的框架，当然也有。POE是用内置的POE kernel来处理事件循环的。

POE可以看作是一个操作系统，它在运行时就像操作系统一样工作。当一个操作系统启动完毕，它就静待事件的到来，这事件可能是从用户空间发起的系统调用，也可能是硬件中断。操作系统还负责在不同的工作组件之间传递消息—一般来说是在进程间传递消息(IPC)。POE kernel也做同样的事—响应事件和在不同的POE工作组件（在POE中称为Session）之间传递消息。

## Hello,POE

讲了很多空话，让我们来看一个小例子吧:

<pre class="brush: perl">use POE;
POE::Session-&gt;create
(
inline_states=&gt;
    {
        _start=&gt;&start,
        hello=&gt;&hello,
    },
);

print "Running Kerneln";
$poe_kernel-&gt;run();
print "Exitingn";
exit(0);

sub start
{
    my ($kernel) = $_[KERNEL];
    print "setting up a Sessionn";
    $kernel-&gt;yield("hello");
}

sub hello
{
    print "Hello,worldn";
}</pre>

这就是用POE实现的Hello,world。继续用类似操作系统的比喻来看（目前为止还可以这么看），我们启动了POE内核，然后创建了一个进程让它打印&#8221;Hello,world&#8221;，然后退出。下面让我们来看看这个例子的各个部分。

<pre class="brush: perl">use POE;
print "Running Kerneln";
$poe_kernel-&gt;run();
print "Exitingn";
exit(0);</pre>

这是每个POE程序的核心，变量$poe_kernel是POE模块提供来表示它自己的。在许多情况下，run方法的调用根本就不返回，比如说，一个网络服务器应该一直等待新的连接，除非发生了严重错误。但我们这里只是创建了一个很快结束的简短session。较新的代码可能会这样写POE::Kernel->run，效果是等同的。

<pre class="brush: perl">POE::Session-&gt;create
(
    inline_states=&gt;
    {
        _start=&gt;&start,
        hello=&gt;&hello,
    },
);</pre>

这段代码创建了一个session，一个session可以看作是一个有多种状态的状态机，或者一个多事件的句柄&#8211;这两者是等效的。用状态机的看法，这段代码在inline\_states里定义了两种状态。下划线开头的状态是POE预定义的，其它的是自定义的。session在成功创建后会马上进入\_start状态。如果你更喜欢事件驱动的视角，那可以说这个session响应\_start事件和hello事件，POE会在session创建后立即向它发出一个\_start事件。

还有其它的预定义事件，大多数都是处理父/子关系和信号。比如说_stop事件，他在一个session结束前发出。下面我们看看我们怎么处理自定义的事件。

<pre class="brush: perl">sub start
{
    my ($kernel) = $_[KERNEL];
    print "Setting up a sessionn";
    $kernel-&gt;yield("hello");
}
sub hello
{
    print "Hello,worldn";
}</pre>

POE会给事件处理方法传递包括POE kernel自身的引用在内的一系列参数。在start方法里我们用KERNEL关键字把POE kernel拿出来。出于效率的考虑，POE用类似这样的数组而不是哈希。你常常会看到POE事件处理方法这样开头:

<pre class="brush: perl">my ($kernel,$heap,$session) = @_[KERNEL, HEAP, SESSION];</pre>

代码邮编只是一个用预定义常数做索引的普通数组，返回了POE kernel，heap和当前的session对象。heap是一个session可以存放私有数据的地方，per-session stuff。我们稍后会讲到什么类型的数据适合存放在heap里。

现在我们有了POE kernel，那么我们用它干啥呢？我们告诉它我们想转入另一个状态，&#8221;hello&#8221;状态。

<pre class="brush: perl">$kernel-&gt;yield("hello");</pre>

向当前session发出事件使用yield。如果要向别的session发出事件，我们要使用post方法。我们一会儿就会看到一个post的例子。

现在我们的start状态处理方法已经告诉POE kernel我们想要马上转入hello状态。但这要到POE开始了事件循环以后才会发生。一旦我们执行了$poe_kernel->run，POE kernel查看当前的任务列表，于是发现第一件要做的事就是把我们的session转入hello状态，然后它就执行了hello状态的处理方法。然后&#8221;hello，world&#8221;就被打印出来了。

## Hello,Again,POE!

如果我们需要每5秒钟都&#8221;hello，world!&#8221;一次，当然，我们可以这样做:

<pre class="brush: perl">sub hello
{
    my ($kernel) = @_[KERNEL];
    print "hello,world!n";
    sleep 5;
    $kernel-&gt;yield("hello");
}</pre>

但这可不是在多任务处理环境中的好办法。我们不能简单地把整个kernel停住5秒，因为其它的session可能会有事情要做:也许有需要马上处理的网络请求，或者诸如此的。所以，我们要做的是让kernel把hello列到5秒钟之后的计划里去。我们用delay_set方法来做到这一点。

<pre class="brush: perl">sub hello
{
    my ($kernel) = @_[KERNEL];
    print "hello,world!n";
    $kernel-&gt;delay_set("hello",5);
}</pre>

现在这样就对其它session礼貌多了。现在让我们来看看在有两个session运行的时候我们能做什么。下面是一些稍做改动后的Matt Sergeant的POE教程里的代码(http://www.axkit.org/docs/presentations/tpc2002/poe/):

<pre class="brush: perl">use POE;
for my $session_no (1..2)
{
    POE::Session-&gt;create
    (
        inline_states =&gt;
        {
            hello=&gt; &hello,
            _start=&gt; sub{ $_[KERNEL]-&gt;alias_set("session_" . $session_no)},
        }
    );
}
$poe_kernel-&gt;post("session_1","hello","session_2");
$poe_kernel-&gt;run();
exit(0);

sub hello
{
    my ($kernel, $session, $next) = @_[KERNEL, SESION, ARG0];
    print "Event in ",$kernel-&gt;alias_list($session), "n";
    $kernel-&gt;post($next, "hello", $session-&gt;ID);
}</pre>

我们之前已经很多次创建一个包括初始化事件和一个hello事件的session，这次我们创建了两个。注意，这两个session使用了相同的事件处理方法，但传给每个session的数据是不一样的。

这次，_start方法做了些和之前不同的事，它告诉kernel给当前session注册了一个别名。每个session都有一个内部ID(我们稍后就会用到)，但只有session创建的时候POE才能得到这个ID。注册一个易记的别名有助于我们之后更方便地找到这个session出来干活。

还有一个小技巧是，我们还可以向kernel询问一个session的别名，这样可以让输出更易读。

<pre class="brush: perl">print "Event in ",$kernel-&gt;alias_list($session), "n";</pre>

现在有了两个准备好的session，接下来我们需要告诉kernel哪个先开始运行，我们给session1传递一个hello事件让它开始运行，session_1是我们给它的别名:

<pre class="brush: perl">$poe_kernel-&gt;post("session_1", "hello", "session_2");</pre>

我们可以在给别的session或者给当前session传递事件时附带任意个参数。可以在@_里取到传进来的参数，从ARG0开始一直往后。如果我们有很多参数，我们可以这样把全部的参数都拿下来:

<pre class="brush: perl">my ($kernel, $session, @args) = @_[KERNEL, SESSION, ARG0..$#_];</pre>

这里我们只要第一个参数，里面放着下个要干活的session。session1把控制权交给session2，然后session2又交还给session1，如此循环。现在程序已经开始运转了，我们不再需要分辨出是哪个session在运行了，所以我们可以通过传递之前说过的session自身内部ID来确定下一个工作的session。

<pre class="brush: perl">$kernel-&gt;post($next, "hello", $session-&gt;ID);</pre>

这就是说:“我现在把控制权交给你，你一会儿干完活儿就交回来。”

这两个session交替运行着，这就是一个简单的多任务协同工作的环境，输出如下:

Event in session_1
  
Event in session_2
  
Event in session_1
  
Event in session_2
  
&#8230;

但如果我们想用这个多任务协同环境来做点真正有用的事，那我们就要看看按照POE的思想是如何去处理复杂的I/O问题的。进入下一小节。
