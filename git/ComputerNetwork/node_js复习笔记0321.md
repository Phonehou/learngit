# 跨域
## 域名、子域名、主域名
主机名/域名：主机名标识服务器。（例如163.com想建立一个www服务器，所以就有www.163.com，想建立一个邮箱服务器，mail.163.com就有了），另外，也许还会有子域名作为主机名的前缀。子域名可以是任何形式的，其中www最为常见。子域名通常是可选的。

域名/子域名/主机名

.com是顶级域名。
baidu.com是一级域名。
www.baidu.com、bbs.baidu.com、news.baidu.com是二级域名。

总结：（上面可以这么理清楚，www.baidu.com这个主机名/网站名中，.com是顶级域名，baidu.com是一级域名，www是主机名。主机名标识服务器，所以baidu.com想建立www服务器就是www.baidu.com.，想建立mail服务器，就是mail.baidu.com）。

ibm.com是域名，域名下可以有多个主机，域名下还可以有多个子域名，

## 同源策略（Same origin policy, SOP）
指“协议+域名+端口”三者相同，即是两个不同的域名指向同一个IP地址也非同源。如果缺少了同源策略，浏览器很容易受到XSS、CSFR等攻击。
1. 比如一个web应用，用户访问的页面，处理页面的请求的controller都是在同一个contextPath下的，无论在页面上请求AController还是BController，页面、A、B都是同源的，所处的空间位于同一个contextPath下。
2. 同源策略是为了安全，确保一个应用中的资源只能被本应用的资源访问。否则，岂不是谁都能访问。

2. 以下是相对于 http://www.a.com/test/index.html 的同源检测
1) http://www.a.com/dir/page.html 成功，默认port为80
2) http://www.child.a.com/test/index.html  失败，域名不同
3) https://www.a.com/test/index.html  失败，协议不同
4) http://www.a.com:8080/test/index.html  失败， port不同
5) http://192.168.4.12/b.js
http://www.domain.com/a.js   虽然域名指向的ip和ip相同，也不是同源，因为域名不同。
6) http://script.a.com/b.js  不允许,主域相同，子域不同
7) http://a.com/b.js  同一域名，不同二级域名（同上）	不允许(cookie这种情况下也不允许访问）
8) http://www.cnblogs.com/a.js  不同域名	不允许

特别注意两点：

第一，如果是协议和端口造成的跨域问题“前台”是无能为力的。

第二,在跨域问题上，域仅仅是通过“URL的首部”来识别而不会去尝试判断相同的ip地址对应着两个域或两个域是否在同一个ip上。
## 解决跨域问题  
### 前端解决跨域问题
1> document.domain + iframe      (只有在主域相同的时候才能使用该方法)
2> 动态创建script
3> location.hash + iframe

原理是利用location.hash来进行传值。

假设域名a.com下的文件cs1.html要和cnblogs.com域名下的cs2.html传递信息。
1) cs1.html首先创建自动创建一个隐藏的iframe，iframe的src指向cnblogs.com域名下的cs2.html页面
2) cs2.html响应请求后再将通过修改cs1.html的hash值来传递数据
3) 同时在cs1.html上加一个定时器，隔一段时间来判断location.hash的值有没有变化，一旦有变化则获取获取hash值
注：由于两个页面不在同一个域下IE、Chrome不允许修改parent.location.hash的值，所以要借助于a.com域名下的一个代理iframe
4> window.name + iframe

window.name 的美妙之处：name 值在不同的页面（甚至不同域名）加载后依旧存在，并且可以支持非常长的 name 值（2MB）。

5> postMessage（HTML5中的XMLHttpRequest Level 2中的API

6> CORS背后的思想，就是使用自定义的HTTP头部让浏览器与服务器进行沟通，从而决定请求或响应是应该成功，还是应该失败。

7> JSONP

JSONP包含两部分：回调函数和数据。

回调函数是当响应到来时要放在当前页面被调用的函数。

数据就是传入回调函数中的json数据，也就是回调函数的参数了。

8> web sockets

web sockets是一种浏览器的API，它的目标是在一个单独的持久连接上提供全双工、双向通信。(同源策略对web sockets不适用)

web sockets原理：在JS创建了web socket之后，会有一个HTTP请求发送到浏览器以发起连接。取得服务器响应后，建立的连接会使用HTTP升级从HTTP协议交换为web sockt协议。

只有在支持web socket协议的服务器上才能正常工作。

## JSONP
原理非常简单，就是HTML标签中，很多带src属性的标签都可以跨域请求内容，比如我们熟悉的img图片标签。同理，script标签也可以，可以利用script标签来执行跨域的javascript代码。通过这些代码，我们就能实现前端跨域请求数据。

当需要通讯时，本站脚本创建一个<script>元素，地址指向第三方的API网址，形如：     <script src="http://www.example.net/api?param1=1&param2=2"></script>     并提供一个回调函数来接收数据（函数名可约定，或通过地址参数传递）。     

第三方产生的响应为json数据的包装（故称之为jsonp，即json padding），形如：     callback({"name":"hax","gender":"Male"})     这样浏览器会调用callback函数，并传递解析后json对象作为参数。本站脚本可在callback函数里处理所传入的数据。


## CORS(跨域资源共享)
浏览器将CORS请求分成两类：简单请求（simple request）和非简单请求（预检请求）（not-so-simple request）

### CORS的简单请求

对于简单请求，浏览器直接发出CORS请求。具体来说，就是在头信息之中，增加一个Origin字段。

下面是一个例子，浏览器发现这次跨源AJAX请求是简单请求，就自动在头信息之中，添加一个Origin字段。
GET /cors HTTP/1.1 
Origin: http://m.xin.com
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...

上面的头信息中，Origin字段用来说明，本次请求来自哪个源（协议 + 域名 + 端口）。服务器根据这个值，决定是否同意这次请求。
如果Origin指定的源，不在许可范围内，服务器会返回一个正常的HTTP回应。浏览器发现，这个回应的头信息没有包含Access-Control-Allow-Origin字段（详见下文），就知道出错了，从而抛出一个错误，被XMLHttpRequest的onerror回调函数捕获。注意，这种错误无法通过状态码识别，因为HTTP回应的状态码有可能是200。

如果Origin指定的域名在许可范围内，服务器返回的响应，会多出几个头信息字段。
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: FooBar
Content-Type: text/html; charset=utf-8

# CORS非简单请求-预检请求

非简单请求是那种对服务器有特殊要求的请求，比如请求方法是PUT或DELETE，或者Content-Type字段的类型是application/json。

非简单请求的CORS请求，会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求（preflight）。

浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的XMLHttpRequest请求，否则就报错。

XSS(跨站脚本漏洞, Cross Site Scripting)

是指恶意攻击者往Web页面里插入恶意Script代码，当用户浏览该页之时，嵌入其中Web里面的Script代码会被执行，从而达到恶意攻击用户的目的。

通过php的输出函数将javascript代码输出到html页面中，通过用户本地浏览器执行的，所以xss漏洞关键就是寻找参数未过滤的输出函数。
常见的输出函数有： echo printf print print_r sprintf die var-dump var_export.

xss 分类：（三类）

* 反射型XSS：<非持久化> 攻击者事先制作好攻击链接, 需要欺骗用户自己去点击链接才能触发XSS代码（服务器中没有这样的页面和内容），一般容易出现在搜索页面。

在黑盒测试中，这种类型比较容易通过漏洞扫描器直接发现，我们只需要按照扫描结果进行相应的验证就可以了。

相对的在白盒审计中， 我们首先要寻找带参数的输出函数，接下来通过输出内容回溯到输入参数，观察是否过滤即可。


* 存储型XSS：<持久化> 代码是存储在服务器中的，如在个人信息或发表文章等地方，加入代码，如果没有过滤或过滤不严，那么这些代码将储存到服务器中，每当有用户访问该页面的时候都会触发代码执行，这种XSS非常危险，容易造成蠕虫，大量盗窃cookie（虽然还有种DOM型XSS，但是也还是包括在存储型XSS内）。

* DOM型XSS：基于文档对象模型Document Objeet Model，DOM)的一种漏洞。DOM是一个与平台、编程语言无关的接口，它允许程序或脚本动态地访问和更新文档内容、结构和样式，处理后的结果能够成为显示页面的一部分。DOM中有很多对象，其中一些是用户可以操纵的，如uRI ，location，refelTer等。客户端的脚本程序可以通过DOM动态地检查和修改页面内容，它不依赖于提交数据到服务器端，而从客户端获得DOM中的数据在本地执行，如果DOM中的数据没有经过严格确认，就会产生DOM XSS漏洞。

存储型XSS的白盒审计同样要寻找未过滤的输入点和未过滤的输出函数。

### 防御反射型xss

A.PHP直接输出html的，可以采用以下的方法进行过滤：

    1.htmlspecialchars函数
    2.htmlentities函数
    3.HTMLPurifier.auto.php插件
    4.RemoveXss函数

B.PHP输出到JS代码中，或者开发Json API的，则需要前端在JS中进行过滤：

    1.尽量使用innerText(IE)和textContent(Firefox),也就是jQuery的text()来输出文本内容
    2.必须要用innerHTML等等函数，则需要做类似php的htmlspecialchars的过滤

例如:htmlentities()函数对用户输入的<>做了转义处理,恶意代码当然也就没法执行了。

C.其它的通用的补充性防御手段

    1.在输出html时，加上Content Security Policy的Http Header
    （作用：可以防止页面被XSS攻击时，嵌入第三方的脚本文件等）
    （缺陷：IE或低版本的浏览器可能不支持）
    2.在设置Cookie时，加上HttpOnly参数
    （作用：可以防止页面被XSS攻击时，Cookie信息被盗取，可兼容至IE6）
    （缺陷：网站本身的JS代码也无法操作Cookie，而且作用有限，只能保证Cookie的安全）
    3.在开发API时，检验请求的Referer参数
    （作用：可以在一定程度上防止CSRF攻击）
    （缺陷：IE或低版本的浏览器中，Referer参数可以被伪造）

本地储存localStorage与cookie的区别
1）cookie在浏览器与服务器之间来回传递
sessionStorage和localStorage不会把数据发给服务器，仅在本地保存
2）数据有效期不同
cookie只在设置的cookie过期时间之前一直有效，即使窗口或浏览器关闭
sessionStorage：仅在当前浏览器窗口关闭前有效
localStorage 始终有效，长期保存
3）cookie数据还有路径的概念，可以限制cookie只属于某个路径下
存储大小也不同，cookie数据不能超过4k，sessionStorage和localStorage 虽然也有存储大小的限制，但比cookie大得多，可以达到5M或更大
4）作用域不用
sessionStorage不在不同的浏览器窗口中共享
localStorage在所有同源窗口中都是共享的
cookie也是在所有同源窗口中都是共享的
WebStorage 支持事件通知机制，可以将数据更新的通知发送给监听者。Web Storage 的 api 接口使用更方便

cookie和session的区别
1）cookie数据存放在客户的浏览器上，session数据放在服务器上
2）cookie不是很安全，别人可以分析存放在本地的cookie并进行cookie欺骗，考虑到安全应当使用session
3）session会在一定时间内保存在服务器上，当访问增多，会比较占用你服务器的性能，考虑到减轻服务器性能方面，应当使用cookie
4）单个cookie保存的数*据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie
5）建议将登录信息等重要信息存放为session，其他信息如果需要保留，可以放在cookie中
6）session保存在服务器，客户端不知道其中的信心；cookie保存在客户端，服务器能够知道其中的信息
7）session中保存的是对象，cookie中保存的是字符串
8）session不能区分路径，同一个用户在访问一个网站期间，所有的session在任何一个地方都可以访问到，而cookie中如果设置了路径参数，那么同一个网站中不同路径下的cookie互相是访问不到的

浏览器本地存储与服务器端存储的区别
1）数据既可以在浏览器本地存储，也可以在服务器端存储
2）浏览器可以保存一些数据，需要的时候直接从本地存取，sessionStorage、localStorage和cookie都是由浏览器存储在本地的数据
3）服务器端也可以保存所有用户的所有数据，但需要的时候浏览器要向服务器请求数据
4）服务器端可以保存用户的持久数据，如数据库和云存储将用户的大量数据保存在服务器端 ，服务器端也可以保存用户的临时会话数据，服务器端的session机制，如jsp的session对象，数据保存在服务器上
5）服务器和浏览器之间仅需传递session id即可，服务器根据session id找到对应用户的session对象，会话数据仅在一段时间内有效，这个时间就是server端设置的session有效期
6）服务器端保存所有的用户的数据，所以服务器端的开销较大，而浏览器端保存则把不同用户需要的数据分别保存在用户各自的浏览器中，浏览器端一般只用来存储小数据，而非服务可以存储大数据或小数据服务器存储数据安全一些，浏览器只适合存储一般数据
