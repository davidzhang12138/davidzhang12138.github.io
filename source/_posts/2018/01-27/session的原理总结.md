---
title: session的原理总结
date: 2018-01-27 13:50:59
tags: [PHP]
categories: [PHP]
---


之前在学校的时候，只知道session与cookie的区别在于：session是保存在服务器端，cookie保存在客户端。
session怎么样保存的？以文件的形式保存。自己去测试过。有的忘记了。对应session的id号模糊不清。
在开发中，非常有必要弄明白具体细节。不能停留在使用session_start()函数了，然后获取session值。不知道里面的机制，在开发中遇到了新的问题，解决起来比较费心。

<!-- more -->

### 一、session是怎么保存的？怎么去查看其内容？

session是以文件的形式保存的。php.ini中有个配置项--session.save_path= "";这个里面填写的路径，将会使session文件保存在该路径下。
session文件的命名格式是："sess_[PHPSESSID的值]"。每一个文件，里面保存了一个会话的数据。
其实只要使用代码$_SESSION['user_id'] = $value;就会促发php的session机制，结果往对应的session文件中写入一个值。

### 二、session.save_path路径下这么多的session文件，php是如何确定要调用哪个session文件的？

php是依据，一个名为PHPSESSID的cookie，根据它的值，确定要调用哪个session文件的。
去浏览器中，可以看到一个cookie名为PHPSESSID，假如它的值为"sess_adbjsf2q1ass26oootd163sf84"，
那么，当访问服务器的时候，就会调用session目录下名为"sess_adbjsf2q1ass26oootd163sf84"的文件。
其实，PHPSESSID就是一个会话id，以此来确定，哪个是你的会话数据。


> 以下是在浏览器查看cookie所看到的
> ![错误信息](/img/18-1-27/cookie_info.png)
> cookie的名字PHPSESSID是可以改的，在php.ini中 session.name = PHPSESSID就是设置该cookie的名字。
> 结合自己实际开发中遇到的问题，总结一下：要说session跟cookie有关联的地方，就是跟PHPSESSID这个cookie有绑定关系。
> 其他，不管你设置什么cookie，使用session的时候是不会用到这些值的。也无法获取到。
> 比如同步登陆，设置即使设置了cookie，而你的应用是依据session判断是否为登陆状态的(事实上也必须如此，因为session保存在服务器端，安全性更高，哪个依据cookie认为你已经登陆，那么很惨)。
> 所以，这样的情况就会出现，即使成功设置了cookie。也还是不能同步登陆。

### 三、经常遇到的现象：为什么删除一个session文件，之后生成一个session文件，新的文件名字还是与原来一样？

理解到session文件的命名规则是：“sess_PHPSESSID值“。那么，就很容易明白了。因为，客户端存在cookie：PHPSESSID。
客户端发送请求后，会将该cookie发送给服务器(php可以使用$_COOKIE['PHPSESSID']看到其内容)，
这样的话，还是会根据PHPSESSID生成一个session文件的。

### 四、如何查看session文件中的session值？

我在开发中发现，如果仅仅依靠session_start()和$_SESSION['user_id']这样的代码，去调试，还不够全面的了解问题所在。
比如，我想知道，session_start()到底在完成哪些操作？如果，想动态，实时知道session的值是如何被改写的，打开一个session文件，查看是很了然的。
原来，里面就是保存的是一些被序列化后的值。也明白一个知识点，"php圣经"中讲解session的时候，提到session值做被序列化了。下面看到的session内容就是被序列化了。
打开一个session文件，内容如下：
cityID|i:0;cityName|s:3:"all";fanwe_lang|s:5:"zh-cn";fanwe_currency|a:4:{s:2:"id";s:1:"1";s:6:"name_1";s:9:"人民币";s:4:"unit";s:3:"￥";s:5:"radio";s:6:"1.0000";}_fanwe_hash__|s:32:"77c18770c6cb5d89444c407aaa3e8477";

> 总结出读取规则：
> 1、每一个session的值是以分号";"分开的。比如“cityID|i:0;cityName|s:3:"all";”就是一个完整的session值结束
> 2、里面的读取规则：符号“|”前面表示session名称。符号后面是该session的具体信息。包括：数据类型，字符长度，内容。上面第一个就相当于使用如下php代码访问:$_SESSION['cityID']
> 后面的s表示数据值的长度，3表示字符长度。比如这一段：fanwe_lang|s:5:"zh-cn";  fanwe_lang是变量的名称，变量值是"zh-cn",长度刚好是5，就是”s:5“标明的。
> 最后"all"就表示session的具体值了。就是使用代码$_SESSION['cityID']后会得到的结果。
> 3.一个session可以保存一个数组。符号{}表示数组的内容。上面的花括号{}是$_SESSION['fanwe_currency']所保存的内容。要想查看id的值，就使用代码：$_SESSION['fanwe_currency']['id']

### 五、怎么样理解session_start等函数所做的实际操作是什么？

我是这样理解的：session_start，可以看成是创建一个session文件。假如有原来的session文件，或许没有创建。引入一个。
往session文件中写值，那是代码“$_SESSION['']=" ";  赋值所完成的操作。
session_start()生成一个新的session文件名时。会判断是否存在cookie名为PHPSESSID的值。
如果存在，那么就会按照它的值，组合成一个文件名"sess_[phpcookie值]"。
所以，在目录下，老是能够看到之前删除过的session文件名。
如果将浏览器中对应的cookie(PHPSESSID)删掉。那么就不会生成同样的名字了。
如果不存在名为PHPSESSID的cookie。php所做的估计为：先发送一个cookie，
然后按照cookie的值生成一个(我可以在浏览器中马上看到一个名为PHPSESSID的cookie)

其实，现在也更加深刻地理解了一个知识：在调用session_start()之前不能有任何输出。有输出就会报错。
session_start()已经封装了发送cookie的操作(发送一个名称为PHPSESSID的cookie到浏览器)。
涉及到http的一个原理:头部信息必须在内容之前发送才行。所以，使用echo '内容';
header('Content-type: text/xml; charset=gb2312');//头部信息，不算内容

可以这样认为：session_start()内部已经进行了一次发送头部动作。所以之前不能有任何输出内容。
手册中的英文大致是这样说的：创建一个session，或者恢复当前一个session(基于request请求传递的session id,这里应该值的就是http请求时传递的名为PHPSESSID的cookie)


> 实际开发应用总结：
> 只要是同一个用户的操作。导航程序访问记录和团购程序访问的记录都是保存在同一个session文件中
> 如果是不同的域呢？假如用户访问cs.test.com和daohang.test.com，两方程序都设置了session。那么session的结果保存在同一个session文件中吗？
> 因为：服务器是统一管理session文件的存放的。而php引擎是根据phpsessionid的值确定要操作哪个session文件。session
> 文件名的格式是："sess_[phpcookie值]"。依次寻找对应的session文件(于是在浏览器查看名为PHPSESSIONID的cookie，过期时间是在会话结束后)
> 所以，只要cs.test.com和daohang.test.com使用的是同一台服务器。
> 这样的话，假如是多台服务器的情况。那么就不得不将session保存在数据库中去。这样实现session共享。跟具体的服务器是无关的。
> （2013年更新：实际上共享session不局限于数据库中保存，关于session共享方案，根据自己的理解一年后写了一篇总结文章，http://www.cnblogs.com/wangtao_20/p/3395518.html）
> session文件是某个用户整个会话过程中数据。那么，假如cs.test.com和daohang.test.com下的两个程序运行在同一个服务器上。就意味着，访问cs.test.com与访问daohang.test.com是同一个会话。也就意味着，这两边访问后设置的session数据是保存在同一个session文件中的。
> 可以将名为PHPSESSIONID的cookie，其值看成是一个会话的id。会话结束后，该cookie过期或者被删。那么，服务器对应的session文件(名为"sess_[phpcookie值]")会被删掉吗？查看发现并不会被删掉。所以才会有session文件很多，出现读取性能的问题。session文件比较多的情况下，产生I/Q读写性能问题。了解到可以将session文件分多个目录保存(参考http://www.jb51.net/article/27941.htm)。
> php.ini中的配置项session.save_path,前面一个值M可以指定目录的深度级别。这个没测试过。需要用到的时候，再去测试一下。

## 待解决疑问

### 一、session的过期时间是怎么确定的？

查看session文件内容，发现里面有个值设置了session文件的过期时间：__HTTP_Session_Expire_TS|i:1297750868;
已掌握的信息：
PHPSESSID该cookie的过期时间在浏览器中显示：会话结束后过期
所有的session文件没有被自动删掉，只是有个过期时间，以此决定：是新生成一个session文件还是使用原来的。
原来：服务器定期session清理机制估计会用到这个东西 

### 二、如果没有设置php.ini中的参数。php默认会将session文件保存到什么位置？

#### [服务端]

```php
session.save_handler = files
默认为file，定义session在服务端的保存方式，file意为把sesion保存到一个临时文件里，
如果我们想自定义别的方式保存（比如用数据库），则需要把该项设置为user；

session.save_path = "D:/APMServ5.2.0/PHP/sessiondata/"
定义服务端存储session的临时文件的位置。

session.auto_start = 0
如置1，则不用在每个文件里写session_start(); session自动start ：）

session.gc_probability = 1
session.gc_divisor    = 100
session.gc_maxlifetime = 1440
这三个配置组合构建服务端session的垃圾回收机制
session.gc_probability与session.gc_divisor构成执行session清理的概率，
理论上的解释为服务端定期有一定的概率调用gc函数来对session进行清理，清理的概率为：gc_probability/gc_divisor 比如：1/100  表示每一个新会话初始化时，有
1%的概率会启动垃圾回收程序，清理的标准为session.gc_maxlifetime定义的时间。
```

#### [客户端]

```php
session.use_cookies = 1
sessionid在客户端采用的存储方式，置1代表使用cookie记录客户端的sessionid，
同时，$_COOKIE变量里才会有$_COOKIE[‘PHPSESSIONID’]这个元素存在；

session.use_only_cookies = 1
也是定义sessionid在客户端采用的存储方式，置1代表仅仅使用 cookie 来存放会话 ID。一般来说，现在客户端都会支持
cookie，所以建议设置成1，这样可以防止有关通过 URL 传递会话 ID 的攻击。

session.use_trans_sid = 0
相对应于上面那个设置，这里如果置1，则代表允许sessionid通过url参数传递，同理，建议设置成0；

session.referer_check =  
这个设置在session.use_trans_sid = 1的时候才会生效，目的是检查HTTP头中的"Referer"以判断包含于URL中的会话id是否
有效，HTTP_REFERER必须包含这个参数指定的字符串，否则URL中的会话id将被视为无效。所以一般默认为空，即不检查。  

session.name = PHPSESSID
定义sessionid的名称，即变量名，所以通过浏览器http工具看到的http头文件里的PHPSESSID=##############；

session.hash_function = 0
选择session_name的加密方式，0代表md5加密，1代表sha1加密，默认是0，但是据说用sha1方式加密，安全性更高；

session.hash_bits_per_character = 4
指定在session_name字符串中的每个字符内保存多少位二进制数，这些二进制数是hash函数的运算结果。
4   bits:   0-9,   a-f  
5   bits:   0-9,   a-v  
6   bits:   0-9,   a-z,   A-Z,   "-",   "," 

url_rewriter.tags = "a=href,area=href,frame=src,input=src,form=,fieldset="
指定重写哪些HTML标签来包含sid(session_id)（仅在"session.use_trans_sid"打开的情况下有效），URL重写器将添加一个
隐藏的"<input>"，它包含了本应当额外追加到URL上的信息。  

session.cookie_lifetime = 0
保存sessionid的cookie文件的生命周期，如置0，代表会话结束，则sessionid就自动消失，
常见的强行关闭浏览器，就会丢失上一次的sessionid；

session.cookie_path = /
保存sessionid的cookie文件在客户端的位置；

session.cookie_domain = /
保存sessionid的cookie的域名设置，这跟cookie允许的域名的访问权限设置有关，
一般来说想让自己网站所有的目录都能访问到客户端的cookie，就应该设置成“/”如需要详细了解，
可以看下setcookie()函数的domain参数相关设置和使用方法；

session.bug_compat_42 = 1
session.bug_compat_warn = 1
这两个可以说几乎是快要被废弃的设置，是为了老版本的php服务的，主要是针对 session_register函数，
因为php5的register_global默认是关闭状态，所以在php5里根本用不到 session_register这个函数；
并且php6就要废除这个设置，直接定义为关闭，所以没必要研究这两个了；
```


> 2015.6.14

今天无意中看php手册，注意到这句话：
在某些操作系统上，建议使用可以高效处理大量小尺寸文件的文件系统上的路径来保存会话数据。 
例如，在 Linux 平台上，对于会话数据保存的工作而言，reiserfs 文件系统会比 ext2fs 文件系统能够提供更好的性能。
这个是好的方向。session的数据一般很小。适合小文件的文件系统来使用，而我们的传统的文件系统不太适合。

> 注明：本文转自[CNBLOG][1]。

[1]: http://www.cnblogs.com/wangtao_20/archive/2011/02/16/1955659.html
