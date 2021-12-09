# 数据库键空间
typedef struct redisDb{
    dict *dict;  // 数据库键空间，保存着数据库中的所有键值对
}

## AOF、RDB和复制功能对过期键的处理
* 执行SAVE命令或者BGSAVE命令创建一个新的RDB文件时，程序会对数据库中的键进行检查，已过期的键不会被保存到新创建的RDB文件中。
* 如果服务器以主服务器模式运行，那么在载入RDB文件时，摁键中未过期的键会被载入到数据库中；如果以从服务器模式运行，文件中所有键都会被载入到数据库中

# RDB持久化
将服务器中的非空数据库以及他们的键值对同城为数据库状态

Redis是内存数据库，它将自己的数据库状态存储在内存里面，一旦服务器进程退出，服务器中断数据库状态也会消失，RDB持久化功能可以将Redis在内存中的数据库状态保存到磁盘里面。

## RDB文件的创建与载入
* SAVE命令会阻塞Redis服务器进程，直到RDB文件创建完毕
* BGSAVE会派生出一个子进程，然后由子进程负责创建RDB文件，服务器进程继续处理命令请求
```c
def SAVE():
    # 创建RDB文件
    rdbSave()
def BGSAVE():
    # 创建子进程
    pid = fork()
    if pid == 0:
        # 子进程负责创建RDB文件
        rdbSave()
        # 完成之后向父进程发送信号
        signal_parent()
    elif pid > 0:
        # 父进程继续处理命令请求，并通过轮询等待子进程的信号
        handle_request_and_wait_signal()
    else:
        # 处理出错情况
        handle_fork_error()
```
* 服务器在启动时检测到RDB文件存在，它就会自动载入RDB文件
* 如果服务器开启了AOF持久化功能，那么服务器会优先使用AOF还原数据库状态

## 设置保存条件
```c
struct redisServer{
    struct saveparam *saveparams;  // 记录了保存条件的数组
};

struct saveparam{
    time_t seconds; // 秒数
    int changes; // 修改数
    long long dirty; // 修改计数器,记录距离上一次成功执行SAVE命令或者BGSAVE命令之后，服务器对数据库状态进行了多少次修改
    time_t lastsave;  //上一次执行保存的时间，记录服务器上一次成功执行SAVE命令或BGSAVE命令的时间
};
```

# 事件
* 文件事件(file event):Redis服务器通过套接字与客户端（或者其他Redis服务器）进行连接，会产生相应的文件事件，而服务器则通过监听并处理这些事件来完成一系列网络通信操作。
* 时间事件(time event):Redis服务器中的一些操作需要在给定的事件执行

## 文件事件
Redis基于Reactor模式开发了自己的网络事件处理器：文件事件处理器(file event handler):
* 文件事件处理器使用I/O多路复用程序来同时监听多个套接字，并根据套接字目前执行的任务来为套接字关联不同的事件处理器
* 当被监听的套接字准备好执行连接应答等操作时，与操作相对应的文件事件就会产生。

### 文件事件处理器
* 套接字
* I/O多路复用程序
* 文件事件分派器(dispatcher)
* 事件处理器

1. I/O多路复用程序负责监听多个套接字,并向文件事件分派器传送那些产生了事件的套接字
2. I/O多路复用程序总是会将所欲产生事件的套接字都放到一个队列里面，然后通过这个队列，以有序、同步、每次一个套接字的方式向文件事件分派器传送套接字。

### I/O多路复用程序的实现
Redis为每个I.O多路复用函数库都实现了相同的API
所有功能都是通过包装常见的select、epoll、evport和kqueue这些I/O多路复用函数库来实现

### 事件的类型
当套接字变得可读时,或者有新的可应答(acceptable)套接字出现时，套接字产生AE_READABLE事件

当套接字变得可写时，套接字产生AE_WRITABLE事件

一次完整的客户端与服务器连接事件实例
1. 假设一个Redis服务器正在运作,那么这个服务器的监听套接字的AE_READABLE事件应该正处于监听状态之下,则该事件所对应的处理器为连接应答处理器
2. 如果这时有一个Redis客户端向服务器发起连接,那么监听套接字将产生AE_READABLE事件,触发连接应答处理器执行。处理器会对客户端的连接请求进行应答,然后创建客户端套接字,以及客户端状态,并将客户端套接字的AE_READABLE事件与命令请求处理器进行关联,使得客户端可以向主服务器发送命令请求。

## 时间事件
* 定时事件
* 周期性事件

id：服务器为时间事件创建的全局唯一ID(标识号)
when：毫秒进度的UNIX时间戳，记录了时间时间的到达时间
timeProc:时间事件处理器（一个函数）

## 实现
服务器将所有时间时间都放在一个无序链表中，当时间事件执行器运行时，它就遍历整个链表，查找所有已到达的时间事件，并调用相应的事件处理器。

事件的调度与执行
```c
def aeProcessEvents():
    # 获取到达时间离当前时间最接近的时间事件
    time_event = aeSearchNearestTimer()
    # 计算最接近的时间事件距离到达还有多少毫秒
    remained_ms = time_event.when - unix_ts_now()
    # 如果时间已到达，那么remaind_ms的值可能为负数，将它设定为0
    if remaind_ms < 0:
        remaind_ms = 0
    
    # 根据remaind_ms的值，创建timeval结构
    timeval = create_timeval_with_ms(remaind_ms)
    # 阻塞并等待文件事件产生，最大阻塞事件传入的timeval结构决定
    # 如果remaind_ms的值为0，那么aeApiPoll调用之后马上返回，不阻塞
    aeApiPoll(timeval)
    # 处理所有已产生的文件事件
    # 处理所有已到达的事件事件
```

# 客户端
```c
typedef struct redisClient{
    int fd;  //记录客户端正在使用的套接字描述符
    //fd为-1时是伪客户端(fake client).处理的命令请求来源于载入AOF文件并还原数据库状态或者Lua脚本，而不是网路
    int flags; 
    // REDIS_MASTER表示客户端代表的是一个主服务器,REDIS_SLAVE则是从服务器
    sds querybuf;  //输入缓冲区用于保存客户端发送的命令请求
    robj **argv;  
    int argc;
    //服务器将对命令请求的内容进行分析，保存得出的命令参数以及命令参数个数
    struct redisCommand *cmd;  //程序在命令表中找到argv[0]所对应的redisCommand结构时，它会将客户端状态的cmd指针指向这个结构
    // 之后服务器就可以使用cmd属性所指向的redisCommand结构，以及argv、argc属性中保存的命令参数信息，调用命令实现函数，执行客户端指定的命令
    // 执行命令所得的命令回复会被保存在客户端状态的输出缓冲区里面，每个科幻段都有两个输出缓冲区可用
    char buf[REDIS_REPLY_CHUNK_BYTES];//大小固定的缓冲区保存长度比较小的回复
    int bufpos;  //保存buf数组目前已使用的字节数量
    list *reply;  //可变大小缓冲区
    int authenticated;  //记录客户端是否通过了身份验证
    time_t ctime;  //记录创建客户端的时间
    time_t lastinteraction;  //记录客户端与服务器最后一次进行互动的时间，可以计算客户端的空转（idle）时间
    time_t obuf_soft_limit_reached_time; //记录输出缓冲区第一次到达软性限制(soft limit)的时间
}redisClient;
```
> 使用两种模式来限制客户端输出缓冲区的大小
* 硬性限制(hard limit)：如果输出缓冲区的大小超过了硬性限制所设置的大小，那么服务器立即关闭客户端
* 软性限制(soft limit):如果输出缓冲区的大小超过了软性限制所设置的大小但还没有超过硬性限制，那么服务器将使用客户端状态结构的obuf_soft_limit_reached_time记录下客户端到达软性限制的骑士时间，如果持续时间超过服务器设定的市场，那么服务器将关闭客户端。

# 服务端
命令执行器
1. 查找命令实现
redisCommand结构的主要属性
* name   char*  命令的名字，如“set”
* proc  redisCommandProc*  函数指针,例如setCommand
* arity  int   命令参数的个数,例如-3，标识命令接受三个火以上数量的参数
* sflags  char* 字符串形式的标识值，例如“wm”标识写入命令，并且在执行这个命令之前，服务器应该对占用内存状况进行检查，因为这个命令可能会占用戴安内存
* flags  int  对sflags标识进行分析得出的二进制标识，由程序自动生成
* calls long long  服务器总共执行了多少次这个命令
* milliseconds  long long 服务器执行这个命令所耗费的总时长

2. 执行预备操作

3. 调用命令的实现函数

4. 执行后续工作

## serverCron函数
Redis服务器中的serverCron函数默认每个100ms执行一次，这个函数负责管理服务器的资源，并保持服务器自身的良好运转

* 管理客户端资源 clientsCron
* 管理数据库资源 databasesCron

# 检查持久化操作的运行状态
```c
struct redisServer{
    // 记录执行BGSAVE命令的子进程的ID
    // 如果没有执行，值为-1
    pid_t rdb_child_pid;
    // 记录执行BGREWRITEAOF命令的子进程的ID
    // 如果没有执行，值为-1
    pid_t aof_child_pid;
}
```
每次serverCron函数执行时，程序都会检查这两个属性，只要由一个不为-1,程序就会执行一次wait3函数，检查子进程是否由信号发来服务器进程

* 如果由信号到达，那么表示新的RDB文件已经生成完毕或者AOF文件已经重写完毕，服务器需要进行相应命令的后续操作
* 如果没有信号到达，那么表示持久化操作未完成，程序不做动作。

如果两个值都为-1，那么表示服务器没有在进行持久化操作，那么程序会进行：
1. 查看是否由BGREWRITEAOF被延迟了，如果有的话，那么开始一次新的BGREWRITEAF操作

# 初始化服务器
1. 初始化服务器状态结构：执行initServerConfig函数初始化一般属性，创建一个struct redisServer类型的实例变量server作为服务器的状态，并为结构中的各个属性设置默认值
2. 载入配置选项：用户通过给定配置参数或者指定配置文件来修改服务器的默认配置
3. 初始化服务器数据结构

除了命令表之外，调用initServer函数，为服务器状态其他数据结构分配内存，比如：
* server.clients链表，记录所有与服务器相连的客户端的状态结构
* server.db数组，数组中包含了服务器的所有数据库
* 用于保存频道订阅信息的server.pubsub_channels字典，以及用于保存模式订阅信息的server.pubsub_patterns链表
* 用于执行Lua脚本的Lua环境server.lua
* 用于保存慢查询日志的server.slowlog属性

initServer还一些重要的设置操作，包括：
* 为服务器设置进程信号处理器
* 创建共享对象：包含Redis服务器经常用到的一些值，比如包含“OK”“ERR”回复的字符串对象，包含整数1到10000的字符串对象等。
* 打开服务器的监听端口
* 为serverCron函数创建时间事件
* 如果AOF持久化共嗯那个已经打开，那么打开现有的AOF文件，如果不存在那么创建并打开一个新的AOF文件
* 初始化服务器的后台I/O模块（bio）

4. 还原数据库状态：当服务器完成数据库状态还原工作之后，服务器将在日志中打印出载入文件并还原数据库状态所耗费的时长。
1863:M 13 Mar 2021 00:30:45.504 * DB loaded from disk: 0.008 seconds

5. 执行事件循环：服务器将打印出以下日志。
1863:M 13 Mar 2021 00:30:45.504 * Ready to accept connections

单机数据库(stane alone)

