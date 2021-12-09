# 多机数据库的实现
在Redis中，用户可以通过执行SLAVEOF命令或者slaveof选项，让一个服务器去复制(replicate)另一个服务器，我们称呼被复制的服务器为主服务器(master)，而对主服务器进行复制的服务器则被称为从服务器(slave)
> 进行府志中的主从服务器双方的数据库将保存相同的数据，这种现象称作"数据库状态一致"

## 旧版复制功能的实现
分为同步(sync)和命令传播(command propagate)两个操作。
* 同步将从服务器的数据库状态更新至主服务器当前所在的数据库状态
* 命令传播用于在主服务器的数据库状态被修改，导致主从服务器的数据库状态出现不一致时，让主从服务器的数据库重新回到一致状态。

## 新版复制功能的实现
为了解决旧版复制功能在处理断线重复制情况时的低效问题，Redis从2.8版本开始,使用PSYNC命令代替SYNC命令来执行复制时 同步操作。

PSYNC命令具有两种模式
1. 完整重同步(full resynchronization),处理初次复制情况
2. 部分重同步(partially resynchronization)，处理断线后重复制：当从服务器在断开重新连接主服务器时，如果条件允许，主服务器可以将主从服务器连接断开期间执行的写命令发送给服务器，从服务器只要接收并执行这些写命令，就可以将数据库更新至主服务器当前所处的状态。（从服务器发PSYNC命令，主服务器返回＋CONTINUE回复）

三个部分构成：
1. 主服务器的复制偏移量(replication offset)和从服务器的复制偏移量
2. 从服务器的复制积压缓冲区（replication backlog)

由主服务器维护的一个**固定长度**先进先出的队列(FIFO),默认大小为IMB。当主服务器进行命令传播时，它不仅会将写命令发送给所有从服务器，还会将写命令入队到复制积压缓冲区里面。主服务器会保存着一部分最近传播的写命令，并且复制积压缓冲区会为队列中断每个字节记录相应的复制偏移量。

当从服务器重新连上主服务器时，从服务器会通过PSYNC命令将自己的复制偏移量offset发送给主服务器，主服务器会根据这个复制偏移量来决定对从服务器执行何种同步操作。

如果offset偏移量之后的数据仍然存在于复制积压缓冲区里面，那么主服务器将对从服务器执行部分重同步操作；
如果offset偏移量之后的数据已经不存在于复制积压缓冲区，那么主服务器器舰队从服务器执行完整重同步操作。

> 复制积压缓冲区的最小大小可以根据公式second * write_size_per_second来估算：其中second为从服务器断线后重新连接上主服务器所需的平均时间,后者是主服务器平均每秒产生的写命令数据量。 

* 服务器到达运行ID（run ID)
不论主服务器还是从服务器，都会有自己的运行ID。

运行ID在服务器启动时自动生成，由40个随机的十六进制字符组成.

当从服务器对主服务器进行初次复制时，主服务器会将自己的运行ID传送给从服务器，而从服务器则会将这个运行ID保存起来。

当从服务器断线重连时，从服务器将向当前连接的主服务器发送之前保存的运行ID：如果从服务器保存的运行ID和当前连接的主服务器的运行ID相同，说明从服务器断线之前服务之的就是当前连接的这个主服务器，主服务器可以继续尝试执行部分重同步操作。象返，就要执行完整重同步。

## 复制的实现
1. 设置主服务器的地址和端口
2. 建立套接字连接：如果从服务器创建的套接字能成功连接到主服务器，那么从服务器将为这个套接字关联一个专门用于处理复制工作的文件事件处理器，这个处理器将负责执行后续的复制工作，比如接收RDB文件，以及接收主服务器传播来的写命令。
3. 从服务器向主服务器发送一个PING命令：一可以检查读写状态是否正常，二可以检查主服务器是否正常处理命令请求
4. 身份验证：从服务器在收到主服务器返回的“PONG”回复之后，如果从服务器设置了masterauth选项，那么进行身份验证，从服务器将向主服务器发送一条AUTH命令。
5. 发送端口消息：从服务器将执行命令REPLCONF listening-port <port-number>，向主服务器发送从服务器的监听端口号。
6. 同步：从服务器将向主服务器发送PSYNC命令，执行同步操作，并将自己的数据库更新至主服务器数据库当前所处的状态。注意：在同步操作执行之前，只有从服务器是主服务器的客户端，但是在执行同步操作之后，**主服务器也会成为从服务器的客户端**。
7. 命令传播

> 心跳检测：在命令传播阶段，从服务器默认会以每秒一次的频率，向主服务器发送命令：REPLCONF ACK <replication_offset>
其中replication_offset是从服务器当前的复制偏移量。

发送REPLCONF ACK命令对于主从服务器有三个作用：
* 检测主从服务器的网络连接状态：如果超过一秒钟没有收到从服务器发来的REPLCONF ACK命令，那么主服务器就知道主从服务器之间的连接出现问题了
* 辅助实现min-slaves选项：录入，如果向主服务器提供以下设置：min-slaves-to-write 3  min-slaves-max-lag 10,那么在从服务器的数量少于3个，或者三个从服务器的延迟(lag值都大于或等于10秒时，主服务器将拒绝执行写命令。
* 检测命令丢失

# Sentiel（哨岗、哨兵）
Redis的高可用性(high availability)解决方案
由一个或多个Sentiel实例(instance)组成的Sentiel系统可以监视任意多个主服务器以及主服务器属下的所有从服务器，并在被监视的主服务器进入下线状态时，自己将下线主服务器属下的某个从服务器升级为新的主服务器，然后由新的主服务器代替已下线的主服务器继续处理命令请求。

## 启动Sentiel：
1. 初始化服务器：Sentiel本质上是一个特殊的Redis普乌漆
2. 将普通Redis服务器使用的代码替换成Sential专用代码
3. 初始化Sentinel状态
```c
// sentinel.c/sentielState
struct sentinelState{
    uint64_t current_epoch;  // 当前纪元，用于实现故障转移
    dict *masters;  //保存所有被这个坚实的主服务器
    // 字典的键是主服务器的名字
    // 字典的值则是一个指向sentinelRedisInstance结构的指针
    int tilt;  // 是否进入了TILT模式？
    int running _scripts; // 目前正在执行的脚本的数量
    mstime_t tilt_start_time;  // 进入TILT模式的时间
    mstime_t previous_time; //最后一次执行时间处理器的时间
    list *scripts_queue;  //一个包含所有需要执行的用户脚本的FIFO队列
}sentinel;

// sentinel.c/sentielRedisInstance
typedef struct sentinelRedisInstance{
    int flags; // 标识值，记录了实例的类型，以及该实例的当前状态
    char *name; //名字
    char *runid;  //运行ID
    uint64_t config_epoch;  // 配置纪元，用于实现故障转移
    sentineAddr *addr;  //实例地址
    mstime_t down_after_period; // 实例无响应多少毫秒之后才会被判断为主观下线（subjective down）
    int quorum;  //判断这个实例为客观下线所需的支持投票数(objectively down)
    int parallel_syncs; // 在执行故障转移操作时，可以同时对新的主服务器进行同步的从服务器数量
    mstime_t failover_timeout; // 刷新故障迁移状态的最大时限
}sentinelRedisInstance;

// sentinel.c/sentinelAddr
typedef struct sentinelAddr{
    char *ip;
    int port;
}sentinelAddr;
```
4. 根据给定的配置文件，初始化Sentinel的监视主服务器列表
5. 创建连向主服务器的网络连接:对于每个被Sentinel监视的主服务器，Sentienl会创建两个连向主服务器的异步网络连接：命令连接，专门用于向主服务器发送命令并接收命令回复；订阅连接，专门用于订阅主服务器的__sentinel__:hello频道。

## 获取主服务器信息
## 获取从服务器信息

## 向主服务器和从服务器发送消息
默认Sentinel会以每两秒一次的频率，通过命令连接向所有被监视的主服务器和从服务器发送以下格式的命令：
PUBLISH __sentinel__:hello "<s_ip>,<s_port>,<s_runid>,<s_epoch>,<m_name>,<m_ip>,<m_port>,<m_epoch>"
其中以s_开头的参数记录的是Sentinel本身的信息，m_开头记录的是主服务器的信息，如果Sentinel正在监视的是从服务器，那么这些参数记录的就是从服务器正在复制的主服务器的信息

## 接收来自主服务器和从服务器的频道信息

## 选举领头Sentinel
当一个主服务器被判断为客观下线时，监视这个下线主服务器的各个Sentinel会进行协商，选举出一个零头，并由领头对下线主服务器执行故障转移操作。

# 故障转移
1. 在已下线主服务器属下的所有从服务器里面，挑选出一个从服务器，并将其转换为主服务器。
2. 让已下线主服务器属下的所有从服务器改为复制新的服务器
3. 将已下线主服务器设置为新的主服务器的从服务器，当这个旧的主服务器重新上线时，他就会成为新的主服务器的从服务器。

# 集群
集群通过分片(sharding)来进行数据共享，并提供复制和故障转移功能。

一个Redis集群通常由多个节点(node)(运行在集群模式下的Redis服务器)组成

连接各个节点
CLUSTER MEET <ip> <port>, 让node节点与ip和port所指定的节点进行握手，当握手成功时，node节点就会将ip和port所指定的节点添加到node节点当前所在的集群中。

集群数据结构
clusterNode结构保存了一个节点的当前状态
```c
struct clusterNode{
    mstime_t ctime;  // 创建节点的时间
    char name[REDIS_CLUSTER_NAMELEN];  //节点的名字
    int flags; //节点标识，记录角色（主节点或从节点）以及节点目前所处的状态（比如在线或者下线）
    uint64_t configEpoch; //节点当前的配置纪元，用于实现故障转移
    char ip[REDIS_IP_STR_LEN];  //节点的IP地址
    int port; //节点的端口号
    clusterLink *link;  // 保存连接节点所需的有关消息
};

typedef struct clusterLink{
    mstime_t ctime;  //连接的创建时间
    int fd;  // TCP套接字描述符
    sds sndbuf;  //输出缓冲区
    sds rcvbuf;  //输入缓冲区
    struct clusterNode *node; //与这个连接相关联的节点
}clusterLink;

typedef struct clusterState{
    clusterNode *myself;  //指向当前节点的指针
    uint64_t currentEpoch;  //集群当前的配置纪元，用于实现故障转移
    // 某个节点开始一次故障转移操作时值会增一
    int state;  //集群当前的状态： 是在线还是下线
    int size;  //集群中至少处理着一个槽的节点的数量
    dict *nodes; //集群节点名单
    clusterNode *slots[16384]; //指向一个clusterNode结构所代表的节点
    zskiplist *slots_to_keys; //保存槽和键之间的关系
    clusterNode *importing_slots_from[16384];   // 若指向一个clusterNode结构，表示当前节点正在从clusterNode所代表的节点导入槽i
    clusterNode *importing_slots_to[16384];   // 若指向一个clusterNode结构，表示当前节点正在将槽i迁移至clusterNode所代表的节点
}clusterState; 
```

## 在集群中执行命令
当客户端向节点发送与数据库键有关的命令时，接收命令的节点会计算出命令要处理的数据库键属于哪个槽，并检查这个槽是否指派给了自己，如果键所在的槽并没有指派给当前节点，那么节点会向客户端返回一个MOVED错误，指引客户端转向(redirect)至正确的节点，并再次发送之前想要执行的命令。

### 计算键属于哪个槽
CRC16(key) & 16383   //计算key的CRC-16校验和


## 槽指派
通过分片的方式来保存数据库中的键值对：集群的整个数据库被分为16384个槽(slot)，数据库中的每个键都属于这些槽的其中一个，集群中的每个节点可以处理0个或最多16384个槽。

当数据库中的16384个槽都有节点在处理时，集群处于上线状态(ok);如果数据库中有任何一个槽没有得到处理，那么集群处于下线状态(fail)

通过向节点发送CLUSTER ADDSLOTS命令，我们可以将一个或多个槽指派(assign)给节点负责

### 记录节点的槽指派信息
```c
struct clusterNode{
    unsigned char slots[16384/8];  // 二进制位数组，共包含16384个二进制位，如果在索引i上的二进制位的值为1，那么表示节点负责处理槽i
    int numslots;
    list * fail_reports; //记录所有其他节点对该节点的下线报告
};
```
### 传播节点的槽指派信息

### 记录集群所有槽的指派信息

### 节点数据库的实现
节点和单机服务器在数据库方面的一个区别是，节点只能使用0号数据库，而单机Redis服务器则没有这一限制

## 重新分片

## ASK错误
在进行重新分片期间，源节点向目标节点迁移一个槽的过程中，可能会出现这样的情况：属于被迁移槽的一部分键值对保存在源节点里面，而另一部分键值对则保存在目标节点里面

1. 如果节点收到一个关于键key的命令请求，节点没有在自己的数据库里找到键key，那么节点会检查migrating_slots_to[i]属性，看键key所属的槽i是否正在进行迁移
2. 如果槽i的确在进行迁移的话，那么节点会向客户端发送一个ASK错误，引导客户端到正在导入槽i的节点去查找键key,如果没有，向客户端返回MOVED错误
3. 接到ASK错误的客户端会根据错误提供的IP地址和端口号，转向至正在导入槽的目标节点，然后首先向目标节点发送一个ASKING命令，之后再重新发送原本想要执行的命令。
4. 这时节点判断客户端带有ASKING标识，那么节点执行客户端发送的命令

## 复制与故障转移
主节点 从节点
### 故障检测
发送PING消息的节点会将没有在规定时间接收PING消息的节点标记为疑似下线(probable fail,PFAIL).