# 数据结构与对象
键值对(key-value pair)都是由对象构成：
* 键总是一个字符串对象(string object)
* 值可以是字符串对象、列表对象、哈希对象、集合对象、有序集合对象中的一种

# 简单动态字符串(simple dynamic string, SDS)
数据结构
```c
struct sdshdr{
    int len; // 记录buf数组中已使用字节的数量，等于SDS所保存字符串的长度,将获取字符串长度所需的复杂度为O(1)
    int free;  //记录buf数组中未使用字节的数量
    char buf[];  //字节数组，用于保存字符串
}
```
SDS与C字符串的区别
* C语言使用长度为N+1的字符数组来表示长度为N的字符串，并且字符数组的最后一个元素总是空字符'\0'
* SDS的空间分配策略完全杜绝了发生缓冲区溢出的可能性：如果空间不够，SDS会自动分配空间
* 减少修改字符串带来的内存重分配次数。通过未使用空间，SDS实现了空间预分配和惰性空间释放两种优化策略。
* 惰性空间释放：当SDS的API需要缩短SDS保存的字符串时，程序并不立即使用内存重分配来回收缩短后多出来的字节，而是使用free属性将这些字节的数量记录起来，并等待将来使用。
* 二进制安全（程序不会因为特殊字符队数据做任何限制、过滤），使其可以保存任意格式的二进制数据

# 链表
用于实现列表键、发布与订阅、慢查询、监视器等。
```c
typedef struct listNode{
    struct listNode *prev; // 前置节点
    struct listNode *prev; // 后置节点
    void *value;  //节点的值
}listNode;
// (void*)类型指针，不指定指向任何类型
typedef struct list{
    listNode *head;  //表头节点
    listNode *tail;  //表尾节点
    unsigned long len;  //链表所包含的节点数量
    void *(*dup)(void *ptr);  //节点值复制函数，函数指针(*dup)变量，指向一个返回类型为void指针的函数
    void *(*free)(void *ptr);  //节点值释放函数
    int (*match)(void *ptr, void *key); //节点值对比函数
} list;
```

# 字典
使用哈希表作为第层实现
```c
// 哈希表
typedef struct dictht{
    dictEntry **table;  //哈希表数组
    unsigned long size;  // 哈希表大小
    unsigned long sizemask;  // 哈希表大小掩码，用于计算索引值，总是等于size-1
    unsigned long used;  // 该哈希表已有节点的数量
}dictht;

//哈希表节点
typedef struct dictEntry{
    void *key;  //键
    union{ 
        void *val;
        uint64_tu64;
        int64_ts64;
    } v;  // 值可以是一个指针，或者是一个uint64_t整数，或者是是一个int64_t整数。
    struct dictEntry *next;  // 指向下个哈希表节点，形成链表，可以将多个哈希值相同的键值对连接在一起，以此来解决键冲突(collision)
}dictEntry;

// 字典
typedef struct dict{
    dictType *type;  //类型特定函数，每个dictType结构保存了一簇用于操作特定类型键值对的函数
    void *privdata;  //私有数据，保存了需要传给那些类型特定函数的可选参数
    dictht ht[2];  //哈希表，ht[1]哈希表只会在对ht[0]哈希表进行rehash时使用
    in trehashidx;  //rehash索引，当rehash不在进行时，值为-1
}dict;

typedef struct dictType{
    unsigned int (*hashFunction)(const void *key);  // 计算哈希值的函数
    void *(*keyDup)(void *privdata, const void* key); //复制键的函数
    void *(*valDup)(void *privdata, const void* obj); //复制值的函数
    int (*keyCompare)(void *privdata, const void *key1, const void *key2)  // 对比键的函数
    void (*keyDestructor)(void *privdata, void *key); //销毁键的函数
    void (*valDestructor)(void *privdata, void *obj); //销毁值的函数
}dictType;
// 计算哈希值和索引值的方法如下：
//使用字典设置的哈希函数，计算键key的哈希值
hash = dict->type->hashFunction(key);
// 使用哈希表的sizemask属性和哈希值，计算出索引值，根据情况不同，ht[x]可以是ht[0]或者是ht[1]
index = hash & dict->ht[x].sizemask; // & 位的与操作，全部位为1时才为1
// Redis使用MurmurHash2算法来计算键的哈希值
```
* Redis使用链地址法解决冲突
## rehash
* 为了让哈希表的负载因子维持在一个合理的范围之内,需要对哈希表的大小进行相应的扩展或者收缩

rehash步骤：
1) 为字典的ht[1]哈希表分配空间，如果执行的是扩展操作，那么ht[1]的大小为第一个大于等于ht[0].used*2的2的n次方幂;如果是收缩操作，那么ht[1]d的大小为第一个大于等于ht[0].used的2的n次方幂.
2) 将保存在ht[0]中的所有键值对rehash到ht[1]上面：rehash指的是重新计算键的哈希值和索引值，然后将键值对放置到ht[l]哈希表的指定位置上。
3） ht[0]变为空表后，将ht[1]设置为ht[0]
，并在ht[1]新创建一个空白哈希表，为下一次hash做准备。

> 负载因子=海表已保存节点数量/哈希表大小

### 程序自动扩展哈希表的条件：
1. 服务器目前没有在执行BGSAVE命令或者BGREWRITEAOF命令，并且哈希表的负载因子大于等于1
2. 服务器目前正在执行BGSAVE命令或者BGREWRITEAOF命令，并且哈希表的负载因子大于等于5

在执行BGSAVEm命令或BGREWRITERAOF命令的过程中，Redis需要创建当前服务器进程的子进程，而大多数操作系统都采用写时复制技术来优化子进程的使用效率，所以在子进程存在期间，服务器会提高执行扩展操作所需的负载因子，从而尽可能地避免在子进程存在期间进行哈希表扩展操作，这可以避免不必要的内存写入操作，最大限度地节约内存。

* 渐进式rehash
1) 为ht[1]分配空间，让字典同时持有ht[0]和ht[1]两个哈希表。
2）在字典中为耻一个索引计数器变量rehashidx,并将它的值设置为0，表示rehash工作正式开始。
3) 在rehash进行期间，每次对字典执行添加、删除、查找或者更新操作时，会顺带将ht[0]哈希表在rehashidx索引上的所有键值对rehash到ht[1]，当rehash工作完成之后，程序将rehashidx属性的值增一。
4) 随着字典操作的不断执行，最终在某个时间点上，ht[0]的所有键值对都会被rehash至ht[1]，这是rehashidx属性的值设为-1，表示rehash操作已完成。

# 跳跃表（skiplist）
一种有序数据结构，它通过在每个节点中维持多个指向其他节点的指针，从而达到快速访问节点的目的。

Redis使用跳跃表作为有序集合键的底层实现之一;另一个个应用是在集群节点中用作内部数据结构

```c
// redis.h/zskiplist
typedef struct zskiplist{
    struct zskiplistNode *header,*tail;//指向跳跃表的表头节点,指向跳跃表的表尾节点
    int level; //记录目前跳跃表内，层数最大的那个节点的层数，每个节点的层高是1至32之间的随机数
    unsigned long length; //记录跳跃表的长度（包含节点个数）,表头节点的层高并不计算在内
}zskiplist;

// redis.h/zskiplistNode
typedef struct zskiplistNode{
    //level 每个层都带有两个属性：前进指针和跨度
    struct zskiplistLevel{
        struct zskiplistNode *forward; // 前进指针
        unsigned int span;  //跨度,用来计算排位，在查找某个结点的过程中，键沿途访问过的所有曾的跨度累积起来，得到的结果就是目标节点在跳跃表中的排位
    } level[];
    struct zskiplistNode *backward; //后退指针，节点中用BW字样标记节点的后退指针，它指向位于当前节点的前一个结点
    double score; //分支，各节点按各自所保存的分支从小到大排列
    robj *obj;  //成员对象，指向一个字符串对象，而字符串对象则保存着一个SDS值
}zskiplistNode;
```
## 升级
新元素的类型比整数集合现有所有元素的类型都要长时，整数集合需要项进行升级(upgrade)，然后才能将新元素添加到整数集合里面。

# 整数集合(intset)
集合键的第层实现之一。
typedef struct intset{
    uint32_t encoding;  //编码方式
    uint32_t length; //集合包含的元素数量
    int8_t contents[];  //保存元素的数组，content数组的真正类型取决于encoding属性的值
}intset;

# 压缩列表（ziplist）一个压缩列表则以包含任意多个节点(entry)，每个节点可以保存一个字节数组或者一个整数值
列表键和哈希键的底层实现

由一系列特殊编码的连续内存块组成的顺序型(sequential)数据结构

|属性|类型|长度|用途|
|:---:|:---:|:---:|:----|
|zlbytes|uint32_t|4字节|记录整个压缩列表占用的内存字节数|
|zltail|uint32_t|4字节|记录压缩列表表尾节点距离压缩列表的起始地址有多少字节|
|zllen|uint16_t|2字节|记录了压缩列表包含的节点数量|
|entryX|列表节点|不定|压缩列表包含的各个节点，节点的长度由节点保存的内容决定|
|zlend|uint8_t|1字节|特殊值0xFF,用于标记压缩列表的末端|

每个压缩列表节点组成部分：
1. previous_entry_length 记录压缩列表中前一个节点的长度
2. encoding 记录节点的content属性所保存数据的类型以及长度
3. content 保存节点的值

### 连锁更新
添加/删除新节点到压缩列表，当这种操作出现的机率并不高（压缩列表里要恰好由多个连续的、长度介于250字节至253字节之间的节点）

# 对象
## 对象的类型与编码
每次当我们在Redis的数据库中新创建一个键值对时，我们至少会创建两个对象，一个对象用作键值对的键（键对象），另一个对象用作键值对的值（值对象）。

每个对象都有一个redisObject结构表示：
```c
typedef struct redisObject{
    unsigned type:4;  //类型
    unsigned encoding:4;  //编码,说明对象使用了什么数据结构作为对象的第层实现
    void *ptr;  //指向底层实现数据结构的指针
}robj;
// type属性包括REDIS_STRING(字符串对象)\REDIS_LIST(列表对象)\REDIS_HASH(哈希对象)\REDIS_SET(集合对象)\REDIS_ZSET(有序集合对象)
// 称呼一个数据库键为“字符串键”时，指的是“这个数据库键对应的值为字符串对象”
```

## 字符串对象
编码可以是以下三种：
* REDIS_ENCODING_INT  int(long类型表示的整数值) 
* REDIS_ENCODING_RAW  raw(长度大于32字节的字符串值)  SDS保存
* REDIS_ENCODING_EMBSTR embstr(长度小于等于32字节的字符串值)   embstr编码保存(只调用一次内存分配函数来分配一块连续的空间，空间依次包含redisObject结构和sdshdr结构) 
> long double类型表示的浮点数也是作为字符串值来保存的

## 列表对象
编码
* ziplist(所有字符串元素的长度都小于64字节/元素数量小于512个)
* linkedlist(使用双端链表作为底层实现)

## 哈希对象
* ziplist
* hashtable(使用字典作为底层实现)

## 集合对象
* intset
* hashtable

## 有序集合对象
* ziplist
* skiplist(使用zset结构作为底层实现)

一个zset结构同时包含一个字典和一个跳跃表：
```c
typedef struct zset{
    zskiplist *zsl;  //按分值从小到大保存了所有集合元素
    dict *dict;  //c创建了一个从成员到分值的映射，字典的键保存了元素的成员，字典的值则保存了元素的分值
}zset;
```

## 多态命令的实现
Redis除了会根据值对象的类型来判断键是否能够执行指定命令之外，还会根据值对象的编码方式，选择正确的命令实现代码来执行命令。

## 内存回收
Redis在对象系统中构建了一个引用技术（reference counting）技术实现内存回收机制

## 对象共享
