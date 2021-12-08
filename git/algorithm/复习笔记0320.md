# 特性
单线程、异步IO、事件驱动式
# 包管理器(node package manager, npm)

举例：
```c++
res = db.query('SELECT * from some_table');
res.output();
```
按照传统方式实现时，在执行第一行时，线程会阻塞，等待数据库返回查询结果，然后再继续处理。由于数据库查询可能设计磁盘读写和网路通信，其延时可能相当大。容易遭受低速连接攻击

Node.js处理方式
```c++
db.query('SELECT * from some_table', function(res){
   res.output(); 
});
```
db.query第二个参数是一个函数，我们称为**回调函数**，进程在执行到db.query的时候，不会等待结果返回，而是直接继续执行后面的语句，知道进入事件循环。但数据库查询结果返回时，会将事件发送到事件队列，等到线程进入事件循环以后，才会调用之前的回调函数继续执行后面的逻辑。

> Node.js的异步机制是基于事件的，所有的磁盘I/O、网络通信、数据库查询都已非阻塞的方式请求，返回的结果由事件循环来处理。

Node.js架构
V8              ----JavaScript引擎
libuv           ----在libev和libeio的基础上抽象来利用epoll或kqueues
libev  libeio   ----支持事件驱动和异步式I/O

## ECMAScript标准
同一JavaScript语言标准

## CommonJS规范与实现
定义一套普通应用程序使用的API,从而填补JavaScript标准库过于简单的不足，目标是制定一个像C++标准库一样的规范，使得基于CommonJS API的应用程序可以在不同的环境下运行。

### 使用node的REPL模式
REPL(Read-eval-print loop)即输入——求值——输出循环


> 相应地，异步式 I/O （Asynchronous I/O）或非阻塞式 I/O（Non-blocking I/O）则针对所有 I/O 操作不采用阻塞的策略。
当线程遇到 I/O 操作时，不会以阻塞的方式等待 I/O 操作的完成或数据的返回，而只是将 I/O 请求发送给操作系统，继续执行下一条语句。
当操作系统完成 I/O 操作时，以事件的形式通知执行 I/O 操作的线程，线程会在特定时候处理这个事件。
为了处理异步 I/O，线程必须有事件循环，不断地检查有没有未处理的事件，依次予以处理。

## 单线程事件驱动的异步式I/O相比传统的多线程阻塞式I/O的优点
简而言之，异步式 I/O 就是少了多线程的开销。对操作系统来说，创建一个线程的代价是十分昂贵的，需要给它分配内存、列入调度，同时在线程切换的时候还要执行内存换页，CPU 的缓存被清空，切换回来的时候还要重新从内存中读取信息，破坏了数据的局部性。

## 回调函数
```JavaScript
var fs = require('fs');
fs.readFile('file.txt', 'utf-8', function(err, data){
    if(err){
        console.error(err);
    }else{
        console.log(data);
    }
});
console.log('end.');
```
## 事件
异步I/O操作在完成时都会发送一个事件到事件队列。事件由EventEmitter来实现
```JavaScript
var EventEmitter = require('events').EventEmitter;
var event = new EventEmitter();
event.on('some_event', function() {
 console.log('some_event occured.');
});
setTimeout(function() {
 event.emit('some_event');
}, 1000);
//运行这段代码，1秒后控制台输出了 some_event occured.。其原理是 event 对象
//注册了事件 some_event 的一个监听器，然后我们通过 setTimeout 在1000毫
```

## Node.js 的事件循环机制
Node.js 程序由事件循环开始，到事件循环结束，所有的逻辑都是事件的回调函数，所以 Node.js 始终在事件循环中，程序入口就是事件循环第一个事件的回调函数。事件的回调函数在执行的过程中，可能会发出 I/O 请求或直接发射（emit）事件，执行完毕后再返回事件循环，事件循环会检查事件队列中有没有未处理的事件，直到程序结束。

## npm本地模式和全局模式
npm 与 Ruby 的 gem 或者 Python 的 pip，不同，gem 或 pip 总是以全局模式安装，使包可以供所有的程序使用，而 npm 默认会把包安装到当前目录下。这反映了 npm 不同的设计哲学。
如果把包安装到全局，可以提高程序的重复利用程度，避免同样的内容的多
份副本，但坏处是难以处理不同的版本依赖。
如果把包安装到当前目录，
或者说本地，则不会有不同程序依赖不同版本的包的冲突问题，同时还减
轻了包作者的 API 兼容性压力，但缺陷则是同一个包可能会被安装许多次。

npm 本地模式仅仅是把包安装到 node_modules 子目录下，其中
的 bin 目录没有包含在 PATH 环境变量中，不能直接在命令行中调用。而当我们使用全局模式安装时，npm 会将包安装到系统目录，譬如 /usr/local/lib/node_modules/，同时 package.json 文件中 bin 字段包含的文件会被链接到 /usr/local/bin/。

## 全局对象
Node.js global,所有全局变量都是global对象的属性

process argv  命令行参数数组(第一个元素 node, 第二个元素是脚本文件名, 从第三个元素开始每个元素是一个运行参数)

process.stdout是标准输出流

process.stdin是标准输入流

process.nextTick(callback)为事件循环设置一项任务，Node.js会在下次事件循环响应时调用callback

Node.js适合I/O密集型的应用

## 事件驱动events
### 事件发射器
events 模块只提供了一个对象： events.EventEmitter。EventEmitter 的核心就是事件发射与事件监听器功能的封装。
EventEmitter 的每个事件由一个事件名和若干个参数组成，事件名是一个字符串，通常表达一定的语义。
对于每个事件，EventEmitter 支持若干个事件监听器。
当事件发射时，注册到这个事件的事件监听器被依次调用，事件参数作为回调函数参数传递。
```c++
var events = require('events');
var emitter = new events.EventEmitter();   // 生成一个EventEmitter实例
emitter.on('someEvent', function(arg1, arg2) {
 console.log('listener1', arg1, arg2);
});   //为function(arg1, arg2)注册一个监听器,接受一个字符串 event 和一个回调函数 listener
emitter.on('someEvent', function(arg1, arg2) {
 console.log('listener2', arg1, arg2);
});  //为function(arg1, arg2)注册一个监听器,接受一个字符串 event 和一个回调函数 listener
emitter.emit('someEvent', 'byvoid', 1991); //发射 event 事件，传递若干可选参数到事件监听器的参数表。
```

## 文件系统fs
### HTTP服务器
1. http.Server的事件,继承自EventEmitter提供事件：
* request:当客户端请求到来时,该事件被触发,提供两个参数req、res,分别表示请求和响应信息的实例
* connection:TCP连接建立时触发
* close():服务器关闭时触发

> GET请求把所有的内容编码到访问路径中,POST请求的内容全部都在请求体中

### HTTP客户端

## Express框架
MVC 架构:浏览器发起请求，由路由控制器接受，根据不同的路径定
向到不同的控制器。控制器处理用户的具体请求，可能会访问数据库中的对象，即模型部分。控制器还要访问模板引擎，生成视图的 HTML，最后再由控制器返回给浏览器，完成一次请求。

### REST风格的路由规则
表征状态转移(Representational State Transfer)
* GET: 请求获取指定资源
* HEAD: 请求指定资源的响应头
* POST: 向指定资源提交数据
* PUT: 请求服务器存储一个资源
* DELETE: 请求服务器删除指定资源
* TRACE: 回显服务器收到的请求，主要用于测试或诊断
* CONNECT: HTTP/1.1 协议中预留给能狗将连接改为管道方式的代理服务器
* OPTIONS: 返回服务器支持的HTTP请求方法

## 模板引擎（Template Engine）
从页面模板根据一定的规则生成HTML的工具

推荐使用：ejs(Embedded JavaScript)

### 片段视图(partials)

## 会话支持
会话是一种持久的网络协议，用于完成服务器和客户端之间的一些交互行为，是一个比连接粒度更大的概念，一次会话可能包含多次连接

为了在无状态的HTTP协议之上实现会话，Cookie诞生了。Cookie是一些存储在客户端的信息，每次连接的时候由浏览器向服务器递交，服务器也向浏览器发起存储Cookie的请求，依靠这样的手段服务器可以识别客户端。
具体来说，浏览器首次向服务器发起请求时，服务器生成一个唯一标识符并发送给客户端浏览器，浏览器将这个唯一标识符存储在 Cookie 中，以后每次再发起请求，客户端浏览器都会向服务器传送这个唯一标识符，服务器通过这个唯一标识符来识别用户。

# 闭包(closure)
由函数(环境)及其封闭的自由变量组成的集合体,当一个函数返回它内部定义的一个函数时，就产生了一个闭包，闭包不但包括被返回的函数，还包括这个函数的定义环境。
```JavaScript
var generateClosure = function(){
    var count = 0;
    var get = function(){
        count ++;
        return count;
    };
    return get;
};
var counter = generateClosure();
console.log(counter()); // 输出 1
console.log(counter()); // 输出 2
console.log(counter()); // 输出 3 
```

请实现一个函数，将一个字符串中的每个空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。
```c++
class Solution {
public:
    /**
     * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
     *
     * 
     * @param s string字符串 
     * @return string字符串
     */
    string replaceSpace(string s) {
        if(s.size() == 0)
            return s;
        // write code here
        //1. 遍历一次s,统计出各个非空格子串的起始位置及终止位置
        vector<vector<int>> pos_w;
        int start = 0;
        int end = 0;
        int index = 0;  //非空格及空格字符的下标（一个字符、一个空格各占一位）
//         for(int i = 0; i < s.size(); i++)
//         {
        int i = 0;
        while(i < s.size())
        {
            if(s[i] != ' ')
            {
                index += 1;
                start = i;
                
                while(s[i] != ' '&&i < s.size())
                {
                    i++;
                    end = i;
                }
//                 if(i < s.size()-1 && s[i+1] == ' ' || i == s.size()-1)
//                 {
//                     end = i;
                vector<int> pos{index, start, end};
                pos_w.push_back(pos);
                    
            }
            else{
                index += 1;
                vector<int> pos{index, i, -1};
                pos_w.push_back(pos);
                i++;
            }
            
        }

        string new_s;
        // 将原来非空格字符填充到新的字符串中
        // 新的起始位置=start+index*3, 新的终止位置=end+index*3
        for(int i = 0; i < pos_w.size(); i++)
        {
            if(pos_w[i][2] >= 0)  // 非空白格字符
            {
                new_s += s.substr(pos_w[i][1], pos_w[i][2]-pos_w[i][1]);
            }
            else{
                new_s += "%20";
            }
        }
        return new_s;
    }
};

```
运行时间：3ms 占用内存：504KB 

输入一个链表，按链表从尾到头的顺序返回一个ArrayList。
```c++
/**
*  struct ListNode {
*        int val;
*        struct ListNode *next;
*        ListNode(int x) :
*              val(x), next(NULL) {
*        }
*  };
*/
class Solution {
public:
    vector<ListNode*> dns(ListNode* head, ListNode* tail, int length){
        ListNode* h = head;
        ListNode* t = tail;
        vector<ListNode*> res;
        if(length == 3)  
        {
            t->next = h->next;
            h->next->next =h;
            h->next = NULL;
            res.push_back(tail);  //头结点
            res.push_back(head);  //尾节点

        }
        else if(length == 2)  
        {
            t->next = h;
            h->next = NULL;
            
            res.push_back(tail);  //头结点
            res.push_back(head);  //尾节点
            
        }
        else{
            int mid = length / 2;
            ListNode* m = head;
            int n = 0;
            while(n < mid-1)
            {
                m = m->next;
                n++;
            }  // 找到中点节点
            ListNode* m_n = m->next;
            vector<ListNode*> node_left = dns(head, m, mid);
            vector<ListNode*> node_right = dns(m_n, tail, length-mid);
            node_right[1]->next = node_left[0];
            node_left[1]->next = NULL;
            res.push_back(node_right[0]);  //头结点
            res.push_back(node_left[1]);  //尾节点
        }
        return res;
    }
    vector<int> printListFromTailToHead(ListNode* head) {
        vector<int> res;
        if(!head)
            return res;
        if(!head->next)
        {
            res.push_back(head->val);
            return res;
        }
           
        //采用双指针首位两头兑换
        ListNode* p = head;
        ListNode* t = head;  // 结尾指针
        int length = 1;
        while(t->next){
            t = t->next;
            length += 1;
        }
        vector<ListNode*> node_ = dns(p, t, length);

        ListNode* h = node_[0];
        while(h->next){
            res.push_back(h->val);
            h = h->next;
        }
        res.push_back(h->val);
        return res;
    }
};
```
运行时间：3ms
超过31.11%用C++提交的代码
占用内存：476KB
超过68.17%用C++提交的代码

输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。
```c++
/**
 * Definition for binary tree
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    TreeNode* dns(vector<int> &pre, vector<int> &vin, int k, int start, int end)
    {
        // k表示val在pre的下标
        // start指vin子串开始的位置
        TreeNode* root = new TreeNode(pre[k]);
        if(start == end)
        {
            return root;
        }
        int pos = start;
        // 查找pre第k个元素在vin中的位置
        while(vin[pos]!=pre[k])
        {
            pos++;
        }

        if(pos == start) // 没有左子树
        {
            root->right = dns(pre, vin, k+1, start+1, end);
            root->left = NULL;
        }
           
        else if(pos == end) //没有右子树
        {
            root->left = dns(pre, vin, k+1, start, end-1);
            root->right = NULL;
        }
            
        else{
            root->left = dns(pre, vin, k+1, start, pos-1);
            root->right = dns(pre, vin, k+pos-start+1, pos+1, end);
        }

        return root;
    }
    TreeNode* reConstructBinaryTree(vector<int> pre,vector<int> vin) {
        if(pre.size() == 0 || vin.size() == 0)
            return NULL;
        TreeNode* res = dns(pre, vin, 0, 0, vin.size()-1);
        return res;
    }
};
```
运行时间：3ms
超过73.27%用C++提交的代码
占用内存：500KB
超过73.89%用C++提交的代码

同理：
输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）
```c++
class Solution {
public:
    bool dns(vector<int> &pre, vector<int> &vin, int k, int start, int end)
    {
       // k表示val在pre的下标
        // start指vin子串开始的位置
        if(start == end)
        {
            if(pre[k] != vin[start])
                return false;
            else
                return true;
        }
        int pos = start;
        // 查找pre第k个元素在vin中的位置
        while(vin[pos]!=pre[k])
        {
            pos++;
        }

        if(pos == start) // 没有左子树
        {
            return dns(pre, vin, k+1, start+1, end);
        }
           
        else if(pos == end) //没有右子树
        {
            return dns(pre, vin, k+1, start, end-1);
        }
            
        else{
            return dns(pre, vin, k+1, start, pos-1) && dns(pre, vin, k+pos-start+1, pos+1, end);
        }

    
    }
    bool IsPopOrder(vector<int> pushV,vector<int> popV) {
        //将入栈顺序看作先序遍历,将出栈顺序看作中序遍历,如果无法重建二叉树，则出栈顺序是错误的
        if(pushV.size() == 0)
            return false;
        bool res = dns(pushV, popV, 0, 0, popV.size()-1);
        return res;
    }
};
```
运行时间：3ms
超过31.23%用C++提交的代码
占用内存：444KB
超过75.83%用C++提交的代码

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。
输入一个非递减排序的数组的一个旋转，输出旋转数组的最小元素。
NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。
```c++
class Solution {
public:
    int minNumberInRotateArray(vector<int> rotateArray) {
        if(rotateArray.size()==0)
            return 0;
        //采用二分查找,如果头元素大于尾元素，则说明最小元素在后半部分，否则说明在前半部分
        int low = 0;
        int high = rotateArray.size()-1;
        int mid = (low+high)/2;
        while(low!=mid)
        {
            if(rotateArray[low] > rotateArray[mid] && rotateArray[high] > rotateArray[mid])
            {
                high = mid;
                mid = (low+high)/2;
            }
//             else if(rotateArray[low] < rotateArray[mid] && rotateArray[high] < rotateArray[mid])
            else{
                low = mid;
                mid = (low+high)/2;
            }
        }
        return rotateArray[high]; // high是最小的
    }
};
```
运行时间：21ms
超过79.42%用C++提交的代码
占用内存：888KB
超过34.50%用C++提交的代码