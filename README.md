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

