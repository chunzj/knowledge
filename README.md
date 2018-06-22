# 一、简述HTTPS获取加密密钥的过程
HTTPS就是加入了SSL的HTTP。而SSL协议位于TCP/IP协议与各种应用层协议之间，为数据通讯提供安全支持。在了解清楚HTTPS获取加密密钥的过程，就得先知道对称加密和非对称加密。按照我的理解，对称加密就是通信双方彼此熟悉、信任，事先约定了如何加解密。那如果彼此不认识，那就没法约定加密算法和密钥了，所以就有了非对称加密。而我们现在HTTPS获取加密密钥的过程就是非对称加密，具体过程如下：
- 客户端发出https请求服务器443端口
- 服务器响应公钥
- 客户端生成一个随机数(加密字符串)，然后用服务器返回的公钥进行加密
- 服务器拿到加密过的信息，用自己的私密进行解密，拿到了客户端的随机数。此时，服务器用此随机数加密数据，响应给客户端
- 客户端获取响应后，用本地保存的加密字符串对数据进行解密，就拿到了真正的数据


在服务器响应公钥的过程中，有可能被伪造，就需要进行标识，以确定是自己请求的服务器。而这个标识就是签名，目前最权威的就是CA。客户端在拿到签名后，要对签名进行校验，以验证是否有效包括过期。如果无效，就会给出警告，用户可以选择终止此次请求。

此过程，可以做一个很简单的类比，就是服务器会给客户端一把锁(公钥)，客户端可以锁任何东西，然后传递给服务器。而只有服务器有钥匙(私钥)可以解开这把锁，拿到锁住的东西。就算其他人拿到了这个被锁住的东西，也无力打开拿到。这个过程就是非对称加密，客户端事先不知道任何用于加解密的东西，都是服务器提供的。

# 二、事件循环
首先得明确一点，JS就是<strong>单线程</strong>的，就算加了多少新特性或者语法糖，也改变不了这个事实。而JS在执行的时候，是按照从上到下的顺序依次执行的。如果全是同步的话，那遇到比较耗时的操作，就会阻碍后续的代码执行。所以为了解决这个问题，JS加入了事件循环，即EVENT LOOP来解决此问题，具体过程如下：

JS把执行的业务分为同步任务和异步任务。
- 所有同步任务都在主线程中执行，而在内存中会有开辟一块区域，我们称作"执行栈"，来执行它们。
- 而所有异步任务执行的会注册一个回调，待到异步任务了有了结果，就会放置到一个任务队列中。
- 一旦"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队列"，检查是否有需要执行的事件。此时，异步任务结束等待状态，回到主线程中执行。
- 主线程会不断的重复前三步，来完成这样的过程。

而在JS内部对任务进行了更细致的划分，分为了宏任务(macro task)和微任务(micro task)。比如主代码块、setTimeout、setInterval以及nodejs中的setImediate等都属于宏任务。而promise.then、MutationObserver以及nodejs中process.nextTick等就属于微任务。JS执行栈不断的重复着，执行宏任务，检测并执行当前宏任务阶段的所有微任务,直至下一个宏任务。

至于宏任务和微任务的区别，按照我的理解，就是宏任务与宏任务之间会引发一次UI Render。而微任务则是辅助宏任务的执行。

而其中process.nextTick比较特殊。因为process是内核模块，运行时是全局上下文，所以微任务只有一个，无论你是在哪个阶段、哪个闭包内用nextTick注册的回调都会被push到nextTickQueue，并在事件循环当前阶段结束前执行。所以无论当前宏任务阶段，嵌套了多个process.nextTick都会加入到异步队列的开始，先于异步任务执行。

# 三、正则表达式

对1234567890.12用正则表达式添加千分符，答案为/(\d)(?=(\d{3})+(?!\d))/g
此题考察了对于正则表达式零宽断言的知识点以及分组的概念。所谓的零宽即用来判断位置，判断位置是否满足要求，而不会移动lastIndex。
当前可以用来进行零宽断言的有：
- 零宽正预测先行断言：(?=exp) 即满足后面跟随exp的目标位置
- 零宽正回顾后发断言：(?<=exp) 即满足前面跟随exp的目标位置
- 零宽负预测先行断言：(?!exp) 即满足后面不跟随exp的目标位置
- 零宽负预测后发断言：(?<!) 即满足前面不跟随exp的目标位置
- \b：零宽单词边界
- \B：零宽非单词边界
此外还有标识边界的^、$

# 四、原型链继承

JS的原型链继承，概括到底其实就一句：object.__proto__ = object.constructor.prototype
每个对象都有一个__proto__指向它的原型对象，而每个函数的prototype属性用来定义那些可以被它实例化的对象(即new)继承的属性。通过Object.getPrototypeOf()方法可以获取对象的原型。

所有对象最终都会继承至Object.prototype,除非Object.create(null)创建的对象

而所有构造函数的原型都为Function.prototype,包括function Object(), 而Object.getPrototypeOf(Function.prototype)的原型对象为Object.prototype

# 五、从输入网址到最后浏览器呈现页面内容，中间发生了什么？

当你在浏览器中输入网址，并且敲了回车以后，浏览器就开始工作了。
- 获得域名对应的ip地址，即发送一个UDP的包给DNS服务器，DNS服务器会返回coder.com的IP, 这时候浏览器通常会把IP地址给缓存起来，这样下次访问就会加快。
- 在TCP这个“虚拟的连接”上来发送和接收http请求。想要建立“虚拟的”TCP连接，TCP邮差需要知道4个东西：（本机IP, 本机端口，服务器IP, 服务器端口）

  本机端口很简单，操作系统可以给浏览器随机分配一个，服务器端口更简单，用的是一个“众所周知”的端口，HTTP服务就是80，我们直接告诉TCP邮差就行。
- 经过三次握手以后，客户端和服务器端的TCP连接就建立起来了，就可以发送http请求了。
  
  ![连接图](https://mmbiz.qpic.cn/mmbiz_png/KyXfCrME6UJtecodDmhbLARcnicmYUxLXfhzhnQ9QE6gj188MgVftYDnPoEU75myVCc9UNauypl9Ful7y7bh8cw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)
- 一个HTTP GET请求经过千山万水，历经多个路由器的转发，终于到达web服务器端。

  Web服务器需要着手处理了，它有三种方式来处理：
  - 可以用一个线程来处理所有请求，同一时刻只能处理一个，这种结构易于实现，但是这样会造成严重的性能问题。
  - 可以为每个请求分配一个进程/线程，但是当连接太多的时候，服务器端的进程/线程会耗费大量内存资源，进程/线程的切换也会让CPU不堪重负。
  - 复用I/O的方式，很多Web服务器都采用了复用结构，例如通过epoll的方式监视所有的连接，当连接的状态发生变化（如有数据可读），才用一个进程/线程对那个连     接进行处理，处理完以后继续监视，等待下次状态变化。用这种方式可以用少量的进程/线程应对成千上万的连接请求。
- 对于HTTP GET请求，Nginx利用epoll的方式给读取了出来， Nginx接下来要判断，这是个静态的请求还是个动态的请求啊？
  - 如果是静态的请求（HTML文件，JavaScript文件，CSS文件，图片等），也许自己就能搞定了（当然依赖于Nginx配置，可能转发到别的缓存服务器去），读取本机硬盘上的相关文件，直接返回。
  - 如果是动态的请求，需要后端服务器（如Tomcat)处理以后才能返回，那就需要向Tomcat转发，如果后端的Tomcat还不止一个，那就需要按照某种策略选取一个。
    
    当前Ngnix支持这么几种策略:
    - 轮询：按照次序挨个向后端服务器转发
    - 权重：给每个后端服务器指定一个权重，相当于向后端服务器转发的几率。
    - ip_hash： 根据ip做一个hash操作，然后找个服务器转发，这样的话同一个客户端ip总是会转发到同一个后端服务器。
    - fair：根据后端服务器的响应时间来分配请求，响应时间段的优先分配。
    
    ![nginx与应用服务器](https://mmbiz.qpic.cn/mmbiz_png/KyXfCrME6UJtecodDmhbLARcnicmYUxLXCPeNnuyz1ZBTHkPVlFAUIBy612Wn3GcyshlTRyjN2I8T3iaTPzuo7ibQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

- 不管用哪种算法，某个后端服务器最终被选中，然后Nginx需要把HTTP Request转发给后端的Tomcat，并且把Tomcat输出的HttpResponse再转发给浏览器。
  
  ![整个连接图谱](https://mmbiz.qpic.cn/mmbiz_png/KyXfCrME6UJtecodDmhbLARcnicmYUxLX85tOHEcpBVlOMPMBeVJ92Kt7SguL8eTjbUxNBHZggkI283lroDoM1g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)
  
- 浏览器收到了Http Response，从其中读取了HTML页面，开始准备显示这个页面。
  
  开始到不同的服务器去下载资源，包括js、css、图片等。如果下载的资源太多，则浏览器会创建多个TCP连接，并行的去下载。
  
  ![下载静态资源](https://mmbiz.qpic.cn/mmbiz_png/KyXfCrME6UJtecodDmhbLARcnicmYUxLXjdsj3kiaE65BnFF8TCCyI9UA1wVf1Nj7XC1htib8QpC12r1nUic1cqtCQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)
  
- 当服务器给浏览器发送JS,CSS这些文件时，会告诉浏览器这些文件什么时候过期（使用Cache-Control或者Expire），浏览器可以把文件缓存到本地，当第二次请求同样的文件时，如果不过期，直接从本地取就可以了。

- 如果过期了，浏览器就可以询问服务器端，文件有没有修改过？（依据是上一次服务器发送的Last-Modified和ETag），如果没有修改过（304 Not Modified），还可以使用缓存。否则的话服务器就会被最新的文件发回到浏览器。

- 浏览器会通过DOM Tree和CSS Rule Tree生成所谓“Render Tree”，计算每个元素的位置/大小，进行布局，然后调用操作系统的API进行绘制，这是一个非常复杂的过程。
  
最后，就能看到域名对应的内容了。

# 六、link、script、image新标签属性

- crossorigin
  
  该枚举属性指定在加载相关图片时是否必须使用CORS。可取的值包括以下两个:
  - anonymous：会发起一个跨域请求（即包含Origin: HTTP头）。但不会发送任何认证信息（即不发送cookie, X.509证书和HTTP基本认证信息）。如果服务器没有给出源站凭证（不设置Access-Control-Allow-Origin: HTTP头），这张图片就会被污染并限制使用。 
  - use-credentials：会发起一个带有认证信息 (发送 cookie, X.509 证书和 HTTP 基本认证信息) 的跨域请求 (即包含 Origin: HTTP 头). 如果服务器没有给出源站凭证 (不设置 Access-Control-Allow-Origin: HTTP 头), 这张图片就会被污染并限制使用. 
  - 当不设置该属性时, 资源将会不使用 CORS 加载 (即不发送 Origin: HTTP 头), 这将阻止其在元素中进行使用. 若设置了非法的值, 则视为使用 anonymous. 

- integrity
  
  子资源完整性 (SRI) 是一项安全功能，可让浏览器验证其抓取的文件 (例如，从一个 CDN) 是在没有意外操作的情况下传递的。它的工作原理是允许您提供一个获取的文件必须匹配的加密散列/哈希。 
