---
title: 《Advandced Perl Programming》第七章、POE(2)
date: 2009-10-26T18:08:29+00:00
layout: post
---
## Wheels

Wheels就是POE的I/O系统的动力之源。wheel帮助我们和不断发生事件的外部世界连接起来。你暂时可以想象wheel是POE里的文件句柄，但是wheel还不仅仅是个文件句柄。

最容易理解的一个wheel是POE::Wheel::FollowTail，它用来跟踪一个增长中的文件。你把文件名给它，它就会在这个文件有新内容写入时触发一个事件。下面是一个简洁的例子:

<pre class="brush: perl">use POE qw(Wheel::FollowTail);
POE::Session-&gt;create
(
    inline_states =&gt;
    {
        _start =&gt; sub
        {
            my ($heap) = $_[HEAP];
            my $log_watcher = POE::Wheel::FollowTail-&gt;new
            (
                Filename =&gt; "my_log_file.txt",
                InputEvent =&gt; "got_record",
            );

            $heap-&gt;{watcher} = $log_watcher;
        },

        got_record =&gt; sub {my $record = $[ARG0]; print $record,"n";}
    },
);</pre>

先注意一下载入多个POE模块的紧凑方式；跟在use POE后面的参数都会被理解成相应的POE::模块名依次载入。

像以前一样，我们设置了两个事件。got\_record很好理解，它把它得到的参数原样打印出来。我们仔细看看 \_start里面的内容：

<pre class="brush: perl">my $log_watcher = POE::Wheel::FollowTail-&gt;new
(
    Filename =&gt; "my_log_file.txt",
    InputEvent =&gt; "got_record",
);</pre>

start事件设置了一个Wheel。我们让这个Wheel监视my\_log\_file.txt，文件每加一个新行它都会引发一个got_record事件。

我们用Wheel来干什么？我们想要它在整个运行时一直生效&#8211;不然它就没有用了&#8211;但它也是一个正常的Perl对象，如果我们不把它存储起来它就会在当前块结束时被垃圾回收机制收回。现在我们就能看到上一节所说的heap的用处了:

<pre class="brush: perl">my ($heap) = $_[HEAP];
...
$heap-&gt;{watcher} = $log_watcher;</pre>

这就是我们想要的，Wheel被保存下来一直监视文件并产生事件，然后事件处理函数把传递来的字符打印出来。现在我们再加入另一个Wheel。

让问题再稍稍复杂一点，如果我们的文件是二进制格式的，我们需要用hexdump打印文件内容，那我们该怎么做？

POE::Wheel::Run可以处理和外部程序进行交流的I/O问题。这里我们可以它来和hexdump沟通。

<pre class="brush: perl">use POE qw(Wheel::FollowTail Wheel::Run);
POE::Session-&gt;create
(
inline_states =&gt;
{
    _start=&gt;sub
    {
        my ($heap) = $_[HEAP];
        my $log_watcher = POE::Wheel::FollowTail-&gt;new
        (
            Filename =&gt; "my_log_file.txt",
            InputEvent =&gt; "redirect",
        );
        my $dumper = POE::Wheel::Run-&gt;new
        (
            Program =&gt; "/usr/bin/hexdump",
            StoutEvent =&gt; "print",
        );

        $heap-&gt;{watcher} = $log_watcher;
        $heap-&gt;{dumper} = $dumper;
    },

    redirect =&gt; sub
    {
        my ($heap,$data) = @_[HEAP, ARG0];
        $heap-&gt;{dumper}-&gt;put($data);
    },

    print =&gt; sub
    {
        my $record = $_[ARG0];
        print $record,"n";
    }
},
);</pre>

如果你没有hexdump程序的话，这里给出一小段用perl把二进制文件输出成16进制的代码:

<pre class="brush: perl">my $i = -16;
binmode(STDIN);
my $data;
$|++;
printf "%07x " . ("%02x%02x "x8) . "n", $i+=16, map ord , split//, $data
    while read STDIN, $data, 16;</pre>

图7-1描述了这段程序:

<img style="border: 0pt none; display: inline;" title="图7-1 Filtered log tailing" src="http://blog.yikuyiku.com/wp-content/uploads/2010/01/efc32a4fc780.jpg" border="0" alt="图7-1 Filtered log tailing" width="447" height="246" />

POE::Wheel::FollowTail获取数据并交予POE::Wheel::Run处理，POE::Wheel::Run的print事件打印出数据。看上去很完美。

但是它还不能正常工作。如果尝试和hexdump一起搭配运行它，会发现它什么都不输出。但是如果用上面提供的perl仿写的hexdump，又一切正常。你能猜到原因吗？

原因就在于我们程序里面的&#8221;$|++;&#8221;。系统的hexdump在它认为它在和管道通信时会缓冲自己的输出。主程序持续运行，hexdump却在等待我们结束，所以我们就什么输出都看不到了。我们需要欺骗hexdump让它认为我们是一个真正的终端。当然，POE提供了方法:

<pre class="brush: perl">my $dumper = POE::Wheel::Run-&gt;new
(
    Program       =&gt; "/usr/bin/hexdump",
    Conduit        =&gt; "pty",
    StdoutEvent =&gt; "print",
);</pre>

除上面介绍的两种以外，还有很多我们可以使用的Wheel:
  
POE::Wheel::Curses使用非阻塞的Curses库读取数据；POE::Wheel::ReadLine使用TERM::ReadKey实现可编辑的命令行控制台；POE::Wheel::ListenAccept是一个底层的socker监听器。下面一个例子里我们介绍两个更为重要的Wheel&#8211;POE::Wheel::ReadWrite和POE::Wheel::SocketFactory。
