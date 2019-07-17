## 概述

![](https://github.com/xinliangnote/PHP/blob/master/images/5_php_1.png)

缓存已经成了项目中是必不可少的一部分，它是提高性能最好的方式，例如减少网络I/O、减少磁盘I/O 等，使项目加载速度变的更快。

缓存可以是CPU缓存、内存缓存、硬盘缓存，不同的缓存查询速度也不一样（CPU缓存 > 内存缓存 > 硬盘缓存）。

接下来，给大家逐一进行介绍。

## 浏览器缓存

浏览器将请求过的页面存储在客户端缓存中，当访问者再次访问这个页面时，浏览器就可以直接从客户端缓存中读取数据，减少了对服务器的访问，加快了网页的加载速度。

#### 强缓存

用户发送的请求，直接从客户端缓存中获取，不请求服务器。

根据 Expires 和 Cache-Control 判断是否命中强缓存。

代码如下：

```
header('Expires: '. gmdate('D, d M Y H:i:s', time() + 3600). ' GMT');
header("Cache-Control: max-age=3600"); //有效期3600秒
```

Cache-Control 还可以设置以下参数：

- public：可以被所有的用户缓存（终端用户的浏览器/CDN服务器）
- private：只能被终端用户的浏览器缓存
- no-cache：不使用本地缓存
- no-store：禁止缓存数据

#### 协商缓存

用户发送的请求，发送给服务器，由服务器判定是否使用客户端缓存。

代码如下：

```
$last_modify = strtotime($_SERVER['HTTP_IF_MODIFIED_SINCE']);
if (time() - $last_modify < 3600) {
    header('Last-Modified: '. gmdate('D, d M Y H:i:s', $last_modify).' GMT');
    header('HTTP/1.1 304'); //Not Modified
    exit;
}
header('Last-Modified: '. gmdate('D, d M Y H:i:s').' GMT');
```

#### 用户操作行为对缓存的影响

操作行为 | Expires | Last-Modified
--- | --- | --- 
地址栏回车 | 有效 | 有效
页面跳转 | 有效 | 有效
新开窗口 | 有效 | 有效
前进/后退 | 有效 | 有效
F5刷新 | 无效 | 有效
Ctrl+F5刷新 | 无效 | 无效

## 文件缓存

#### 数据文件缓存

将更新频率低，读取频率高的数据，缓存成文件。

比如，项目中多个地方用到城市数据做三级联动，我们就可以将城市数据缓存成一个文件（city_data.json），JS 可以直接读取这个文件，无需请求后端服务器。

#### 全站静态化

CMS（内容管理系统），也许大家都比较熟悉，比如早期的 DEDE、PHPCMS，后台都可以设置静态化HTML，用户在访问网站的时候读取的都是静态HTML，不用请求后端的数据库，也不用Ajax请求数据接口，加快了网站的加载速度。

静态化HTML有以下优点：

- 有利于搜索引擎的收录（SEO）
- 页面打开速度快
- 减少服务器负担

#### CDN缓存

CDN（Content Delivery Network）内容分发网络。

用户访问网站时，自动选择就近的CDN节点内容，不需要请求源服务器，加快了网站的打开速度。

缓存主要包括 HTML、图片、CSS、JS、XML 等静态资源。


## NoSQL缓存

#### Memcached 缓存

Memcached 是高性能的分布式内存缓存服务器。

一般的使用目的是，通过缓存数据库查询结果，减少数据库访问次数，以提高动态Web应用的速度、提高可扩展性。

它也能够用来存储各种格式的数据，包括图像、视频、文件等。

Memcached 仅支持K/V类型的数据，不支持持久化存储。

**Memcache 与 Memcached 的区别**

- Memcached 从0.2.0开始，要求PHP版本>=5.2.0，Memcache 要求PHP版本>=4.3。
- Memcached 最后发布时间为2018-12-24，Memcache 最后发布时间2013-04-07。
- Memcached 基于libmemcached，Memcache 基于PECL扩展。

可以将 Memcached 看作是 Memcache 的升级版。

PHP Memcached 使用手册：

http://www.php.net/manual/zh/book.memcached.php

Memcached 经常拿来与 Redis 做对比，接下来介绍下 Redis 缓存。

#### Redis缓存

Redis 是一个高性能的 K/V 数据库。

Redis 很大程度补偿了 Memcached K/V存储的不足，比如 List（链表）、Set（集合）、Zset（有序集合）、Hash（散列），既可以将数据存储在内存中，也可以将数据持久化到磁盘上，支持主从同步。

总的来说，可以将 Redis 看作是 Memcached 的扩展版，更加重量级，功能更强大。

Redis 在日常工作中使用的居多。

Redis 学习网址：http://www.redis.cn/

#### MongoDB缓存

MongoDB 是一个基于分布式文件存储的数据库。由 C++ 语言编写。

旨在为 WEB 应用提供可扩展的高性能数据存储解决方案。

MongoDB 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。

MongoDB 学习网址：http://www.mongodb.org.cn

## WEB服务器缓存

#### Apache缓存

利用 `mod_expires` ，指定缓存的过期时间，可以缓存HTML、图片、JS、CSS 等。

打开 `http.conf`，开启模块：

```
LoadModule expires_module modules/mod_expires.so
```

指定缓存的过期时间：

```
<IfModule expires_module>
     #打开缓存
     ExpiresActive on 

     #css缓存(8640000秒=10天)
     ExpiresByType text/css A8640000

     #js缓存
     ExpiresByType application/x-javascript A8640000
     ExpiresByType application/javascript A8640000

     #html缓存
     ExpiresByType text/html A8640000

     #图片缓存
     ExpiresByType image/jpeg A8640000
     ExpiresByType image/gif A8640000
     ExpiresByType image/png A8640000
     ExpiresByType image/x-icon A8640000

 </IfModule>
```

#### Nginx缓存

利用 `expire` 参数，指定缓存的过期时间，可以缓存HTML、图片、JS、CSS 等。

打开 `nginx.conf` ：

```
//以图片为例：
location ~\.(gif|jpg|jepg|png|bmp|ico)$ { #加入新的location
    root html;
    expires 1d; #指定缓存时间
}
```

大家也可以了解下：proxy_cache_path 和 proxy_cache，进行缓存的设置。

## Opcode缓存

Opcode（Operate Code）操作码。

PHP程序运行完后，马上释放所有内存，所有程序中的变量都销毁，每次请求都要重新翻译、执行，导致速度可能会偏慢。

当解释器完成对脚本代码的分析后，便将它们生成可以直接运行的中间代码，也称为操作码。

操作码 的目地是避免重复编译，减少CPU和内存开销。

#### APC缓存

APC（Alternative PHP Cache）可选 PHP 缓存。

APC 的目标是提供一个自由、 开放，和健全的框架，用于缓存、优化 PHP 中间代码。

APC 可以去掉 php 动态解析以及编译的时间，使php脚本可以执行的更快。

APC 扩展最后的发布时间为 2012-09-03。

感兴趣可以了解下，官方介绍：http://php.net/manual/zh/book.apc.php

#### eAccelerator

eAccelerator：A PHP opcode cache。

感兴趣可以了解下，官方介绍：http://eaccelerator.net/

#### XCache

XCache 是一个又快又稳定的 PHP opcode 缓存器。

感兴趣可以了解下，官方介绍：http://xcache.lighttpd.net/

## 小结

文章主要简单的介绍了 浏览器缓存、文件缓存、NoSQL缓存、WEB服务器缓存、Opcode缓存。

每一种缓存都可以深入研究，从介绍 -> 安装 -> 使用 -> 总结应用场景。

大家可以思考下，通过上面的介绍，工作中我们使用了哪些缓存？

还可以再使用哪些缓存，可以对我们的项目有帮助？

## 关于缓存的常见问题

用过缓存，大家肯定遇到过比较头痛的问题，比如数据一致性，雪崩，热点数据缓存，缓存监控等等。

给大家列出几个问题，纯属抛转引玉。

#### 当项目中使用到缓存，我们是选择 Redis 还是 Memcached ，为什么？

举一些场景：

一、比如实现一个简单的日志收集功能或发送大量短信、邮件的功能，实现方式是先将数据收集到队列中，然后有一个定时任务去消耗队列，处理该做的事情。

直接使用 Redis 的 lpush，rpop 或 rpush，lpop。

```
//进队列
$redis->lpush(key, value);

//出队列
$redis->rpop(key);
```

Memcached 没有这种数据结构。

二、比如我们要存储用户信息，ID、姓名、电话、年龄、身高 ，怎么存储？

方案一：key => value

key = user_data_用户ID

value = json_encode(用户数据)

查询时，先取出key，然后进行json_decode解析。

方案二：hash

key = user_data_用户ID

hashKey = 姓名，value = xx

hashKey = 电话，value = xx

hashKey = 年龄，value = xx

hashKey = 身高，value = xx

查询时，取出key即可。

```
//新增
$redis->hSet(key, hashKey, value);
$redis->hSet(key, hashKey, value);
$redis->hSet(key, hashKey, value);

//编辑
$redis->hSet(key, hashKey, value);

//查询
$redis->hGetAll(key); //查询所有属性
$redis->hGet(key, hashKey); //查询某个属性
```

方案二 优于 方案一。

三、比如社交项目类似于新浪微博，个人中心的关注列表和粉丝列表，双向关注列表，还有热门微博，还有消息订阅 等等。

以上都用 Redis 提供的相关数据结构即可。

四、Memcached 只存储在内存中，而 Redis 既可以存储在内存中，也可以持久化到磁盘上。

如果需求中的数据需要持久化，请选择 Redis 。

个人在工作中没有用到 Memcached ，通过查询资料得到 Memcached 内存分配时优于 Redis。

Memcached 默认使用 Slab Allocation 机制管理内存，按照预先规定的大小，将分配的内存分割成特定长度的块以存储相应长度的key-value数据记录，以完全解决内存碎片问题。

#### 如何保证，缓存与数据库的数据一致性？

新增数据：先新增到数据库，再新增到缓存。

编辑数据：先删除缓存数据，再修改数据库中数据，再新增到缓存。

删除数据：先删除缓存数据，再删除数据库中数据。

查询数据：先查询缓存数据，没有，再查询数据库，再新增到缓存。

强一致性是很难保证的，比如事务一致性，时间点一致性，最终一致性等。

具体问题具体分析吧。

#### 缓存穿透怎么办？

用户请求缓存中不存在的数据，导致请求直接落在数据库上。

一、设置有规则的Key值，先验证Key是否符合规范。

二、接口限流、降级、熔断，请研究 istio：https://istio.io/

三、布隆过滤器。

四、为不存在的key值，设置空缓存和过期时间，如果存储层创建了数据，及时更新缓存。

#### 雪崩怎么办？

一、互斥锁，只允许一个请求去重建索引，其他请求等待缓存重建执行完，重新从缓存获取数据。

![](https://github.com/xinliangnote/PHP/blob/master/images/5_php_2.png)

二、双缓存策略，原始缓存和拷贝缓存，当原始缓存失效请求拷贝缓存，原始缓存失效时间设置为短期，拷贝缓存设置为长期。

已上，纯属抛转引玉，结合自己的情况，具体问题，具体分析吧。
