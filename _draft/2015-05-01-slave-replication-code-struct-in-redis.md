---
title: redis的slave复制过程小窥备忘
date: 2015-05-01T21:30:48+00:00

---
## redis.c

1、让我们从redis.c开始，先是一个全局的struct redisServer server;

2、然后main()里面initServerConfig();

3、再然后int serverCron()里面run\_with\_period(1000) replicationCron()。这就转到了replication.c里面；

## replication.c

4、replicationCron()位于replication.c之中。首先执行slaveofCommand()里面做了一堆校验之类的，然后转到replicationSetMaster();

5、replicationSetMaster()里面继续一堆校验，这里第一次把server.repl\_state设成了REDIS\_REPL_CONNECT。

至此，初始化算是完成了。等待下一次replicationCron()的触发；

6、connectWithMaster()开始连接master，并在此函数内注册了回调的syncWithMaster()方法；

7、syncWithMaster()之中完成身份验证之类，转到slaveTryPartialResynchronization()开始尝试同步数据；

8、slaveTryPartialResynchronization()之中分辨要不要全量复制，如果不是则转到replicationResurrectCachedMaster()；

9、replicationResurrectCachedMaster()之中注册回调readQueryFromClient()用以读取同步数据，实际是命令；

## networking.c

10、readQueryFromClient()在networking.c之中。一堆校验后调用processInputBuffer()读取数据；

11、processInputBuffer()之中是一个while死循环，循环到buf被读空。每一次循环依靠processCommand()来执行命令，也就是写入数据；

## 回到redis.c

12、processCommand()在redis.c之中。如果是批量的命令，其依靠queueMultiCommand()来把命令丢到Queue里去慢慢消化。
  
如果是单条则通过call()来执行;

<pre>call(c,REDIS_CALL_FULL);
</pre>

13、call()的原型是void call(redisClient *c, int flags)，依然在redis.c之中。
  
其中核心的一条是c->cmd->proc(c);

## redis.h

14、看一下位于redis.h中的redisClient的定义。

<pre>/* With multiplexing we need to take per-client state.
 * Clients are taken in a linked list. */
typedef struct redisClient {
    uint64_t id;            /* Client incremental unique ID. */
    int fd;
    redisDb *db;
    int dictid;
    robj *name;             /* As set by CLIENT SETNAME */
    sds querybuf;
    size_t querybuf_peak;   /* Recent (100ms or more) peak of querybuf size */
    int argc;
    robj **argv;
    struct redisCommand *cmd, *lastcmd;
    int reqtype;
    int multibulklen;       /* number of multi bulk arguments left to read */
    long bulklen;           /* length of bulk argument in multi bulk request */
    list *reply;
    unsigned long reply_bytes; /* Tot bytes of objects in reply list */
    int sentlen;            /* Amount of bytes already sent in the current
                               buffer or object being sent. */
    time_t ctime;           /* Client creation time */
    time_t lastinteraction; /* time of the last interaction, used for timeout */
    time_t obuf_soft_limit_reached_time;
    int flags;              /* REDIS_SLAVE | REDIS_MONITOR | REDIS_MULTI ... */
    int authenticated;      /* when requirepass is non-NULL */
    int replstate;          /* replication state if this is a slave */
    int repl_put_online_on_ack; /* Install slave write handler on ACK. */
    int repldbfd;           /* replication DB file descriptor */
    off_t repldboff;        /* replication DB file offset */
    off_t repldbsize;       /* replication DB file size */
    sds replpreamble;       /* replication DB preamble. */
    long long reploff;      /* replication offset if this is our master */
    long long repl_ack_off; /* replication ack offset, if this is a slave */
    long long repl_ack_time;/* replication ack time, if this is a slave */
    char replrunid[REDIS_RUN_ID_SIZE+1]; /* master run id if this is a master */
    int slave_listening_port; /* As configured with: SLAVECONF listening-port */
    multiState mstate;      /* MULTI/EXEC state */
    blockingState bpop;   /* blocking state */
    list *watched_keys;     /* Keys WATCHED for MULTI/EXEC CAS */
    dict *pubsub_channels;  /* channels a client is interested in (SUBSCRIBE) */
    list *pubsub_patterns;  /* patterns a client is interested in (SUBSCRIBE) */
    sds peerid;             /* Cached peer ID. */

    /* Response buffer */
    int bufpos;
    char buf[REDIS_REPLY_CHUNK_BYTES];
} redisClient;
</pre>

得知c->cmd的定义是redisCommand，还看到了可爱的int argc和robj **argv变量。
  
感觉接近目标了。

15、redisCommand同样位于redis.h；

<pre>struct redisCommand {
    char *name;
    redisCommandProc *proc;
    int arity;
    char *sflags; /* Flags as string representation, one char per flag. */
    int flags;    /* The actual flags, obtained from the 'sflags' field. */
    /* Use a function to determine keys arguments in a command line. */
    redisGetKeysProc *getkeys_proc;
    /* What keys should be loaded in background when calling this command? */
    int firstkey; /* The first argument that's a key (0 = no keys) */
    int lastkey;  /* The last argument that's a key */
    int keystep;  /* The step between first and last key */
    long long microseconds, calls;
};
</pre>

## 搞定

16、所以，我们只需要在redis.c的1933行call()方法内处加上：

<pre>fprintf(stderr, "[%s]", c->cmd->name);
</pre>

就可以看到每一句redis执行的命令了。

17、如果想要解析出全部命令参数，可以照抄aof.c中的catAppendOnlyGenericCommand()。
  
这样——

<pre>// cg mod 20150501 start
fprintf(stderr, "cmdgot:");
for (int j = 0; j &lt; c->argc; j++) {
  robj *o = getDecodedObject(c->argv[j]);
  fprintf(stderr, " %s", (char *) o->ptr);
  decrRefCount(o);
}
fprintf(stderr, "\n");
// cg mod 20150501 end
</pre>

18、五一节第一天的任务完成。oy~
