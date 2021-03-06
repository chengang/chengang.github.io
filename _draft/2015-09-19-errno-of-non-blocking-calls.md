---
title: 非阻塞read()时的errno
date: 2015-09-19T02:59:08+00:00

---
各个系统非阻塞read()的errno一般不会设置为EINTR。查看socket方法bind()、connect()、send()、receive()的man手册，或是POSIX标准，会发现一件有趣的事情：只有bind()不会把errno设置为EINTR，而bind()正好是其中唯一一个默认非阻塞的方法。

所以，看起来好像只有阻塞的方法才会设置EINTR，包括read()、write()。如果用O_NONBLOCK将它们设置为非阻塞的话，那它们也将不会设置EINTR了。

我们可以从API设计的角度来思考一下这个问题：当阻塞读被信号打断时，系统如何处理这个情况呢？

说read()失败了？其实它只是中断了一下，而且没读到数据而已。

说它成功了但是返回0字节？这也不合适，因为成功且返回0字节这件事已经用在“对方关闭连接”或者“读到文件尾”的状况了。也就是说继续读就读不到内容了。但我们这种情况，如果咱们继续读，是可以读出数据的。

想来想去，还是标记一个错误更合适一些。但是确实又没有发生什么值得去处理的关键性错误，所以就把错误标记成EINTR吧。它大概是在说：“数据流其实没发生啥错误，只是这次调用出了点问题，可能是被信号打断了。后面还有数据呢，请继续调用吧。”

但是如果我们使用了O_NONBLOCK的话，以上的情况就不会发生。因为反正对非阻塞读而言，返回成功又没读到任何东西是正常现象，它一般就会设置EAGAIN了。它大概是在说：“现在还读不到数据呢。我被设置为不许等着数据来，必须退出，所以你自己过会儿再尝试一次吧。”

当然，非阻塞读也会遇到被信号打断的情况。信号会无情地打断所有的方法，不管是系统的还是我们自己写的。所以理论上所有的系统API都可能设置EINTR。但由于非阻塞读总是会空返回，它就不额外提示这种状况了。

MacOS的代码中也确实是这样处理的。如果没读到东西，内核检查句柄的O_NONBLOCK标志位，如果有则返回EAGAIN，如果没有则调用msleep()等待。如果此时信号来了，msleep()将会被打断并设置EINTR，而read()则会把EINTR直接传递上来，于是我们就看到read()返回EINTR了。

所以如果是非阻塞读，我们则完全不会调用到msleep()，也就不会遇见EINTR了。

当然这只是MacOS，但大多数系统在这些基础的API设计上努力在保持一致。我们可以认为，一般而已非阻塞读不会设置EINTR，只会设置EAGAIN。

译自：http://stackoverflow.com/questions/14134440/eintr-and-non-blocking-calls
