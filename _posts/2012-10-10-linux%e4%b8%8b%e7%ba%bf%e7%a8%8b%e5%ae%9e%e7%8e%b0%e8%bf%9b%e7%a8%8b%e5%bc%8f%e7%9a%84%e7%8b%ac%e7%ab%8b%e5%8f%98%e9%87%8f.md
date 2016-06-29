---
id: 3201
title: Linux下线程实现进程式的独立变量
date: 2012-10-10T09:51:11+00:00
author: chen
layout: post
guid: http://blog.yikuyiku.com/?p=3201
permalink: /3201.html
categories:
  - 实践
tags:
  - c
  - 线程
---
<pre class="brush: cpp">#include &lt;pthread.h>
#include &lt;stdio.h>

pthread_mutex_t t_lock;
struct thread_struct
{
        int a;
};

void * thread_start(void *arg)
{
        struct thread_struct *tmp = (struct thread_struct *) arg;
        int b = tmp->a;
        pthread_mutex_unlock(&t_lock);
        sleep(3);
        printf("%u [%d]\n", (unsigned int) pthread_self(), b);
        pthread_exit((void *)0);
}

int main()
{
        printf("parent start.\n");
        int i = 0;
        pthread_mutex_init(&t_lock, NULL);
        while(i&lt;100)
        {
                i++;
                printf("=%d=\n", i);

                pthread_mutex_lock(&#038;t_lock);
                struct thread_struct aaa;
                aaa.a = i;

                pthread_t thread_id;
                pthread_attr_t thread_attr;
                pthread_attr_init(&#038;thread_attr);
                pthread_attr_setdetachstate(&#038;thread_attr, PTHREAD_CREATE_DETACHED);
                pthread_create(&#038;thread_id, &#038;thread_attr, thread_start, (void *) &#038;aaa);
        }
        sleep(10);
        return 0;
}
</pre>