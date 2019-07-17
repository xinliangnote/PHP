## 概览

工作中，我们时刻都会和接口打交道，有的是调取他人的接口，有的是为他人提供接口，在这过程中肯定都离不开签名验证。

在设计签名验证的时候，一定要满足以下几点：

- 可变性：每次的签名必须是不一样的。
- 时效性：每次请求的时效性，过期作废。
- 唯一性：每次的签名是唯一的。
- 完整性：能够对传入数据进行验证，防止篡改。

下面主要分享一些工作中常用的加解密的方法。

## 常用验证

举例：/api/login?username=xxx&password=xxx&sign=xxx

发送方和接收方约定一个加密的盐值，进行生成签名。

示例代码：

```
//创建签名
private function _createSign()
{
    $strSalt = '1scv6zfzSR1wLaWN';
    $strVal  = '';
    if ($this->params) {
        $params = $this->params;
        ksort($params);
        $strVal = http_build_query($params, '', '&', PHP_QUERY_RFC3986);
    }
    return md5(md5($strSalt).md5($strVal));
}

//验证签名
if ($_GET['sign'] != $this->_createSign()) {
    echo 'Invalid Sign.';
}
```

上面使用到了 MD5 方法，MD5 属于单向散列加密。

## 单向散列加密

#### 定义

把任意长的输入串变化成固定长的输出串，并且由输出串难以得到输入串，这种方法称为单项散列加密。

#### 常用算法

- MD5
- SHA
- MAC
- CRC

#### 优点

**以 MD5 为例。**

- 方便存储：加密后都是固定大小（32位）的字符串，能够分配固定大小的空间存储。
- 损耗低：加密/加密对于性能的损耗微乎其微。
- 文件加密：只需要32位字符串就能对一个巨大的文件验证其完整性。
- 不可逆：大多数的情况下不可逆，具有良好的安全性。

#### 缺点

- 存在暴力破解的可能性，最好通过加盐值的方式提高安全性。

#### 应用场景

- 用于敏感数据，比如用户密码，请求参数，文件加密等。

#### 推荐密码的存储方式

**password_hash()** 使用足够强度的单向散列算法创建密码的哈希（hash）。

示例代码：

```
//密码加密
$password = '123456';
$strPwdHash = password_hash($password, PASSWORD_DEFAULT);

//密码验证
if (password_verify($password, $strPwdHash)) {
  //Success
} else {
  //Fail
}
```

PHP 手册地址：

http://php.net/manual/zh/function.password-hash.php

## 对称加密

#### 定义

同一个密钥可以同时用作数据的加密和解密，这种方法称为对称加密。

#### 常用算法

- DES
- AES

AES 是 DES 的升级版，密钥长度更长，选择更多，也更灵活，安全性更高，速度更快。

#### 优点

算法公开、计算量小、加密速度快、加密效率高。

#### 缺点

发送方和接收方必须商定好密钥，然后使双方都能保存好密钥，密钥管理成为双方的负担。

#### 应用场景

相对大一点的数据量或关键数据的加密。

#### AES

AES 加密类库在网上很容易找得到，请注意类库中的 `mcrypt_encrypt` 和 `mcrypt_decrypt` 方法！

![](https://github.com/xinliangnote/PHP/blob/master/images/2_php_1.png)

![](https://github.com/xinliangnote/PHP/blob/master/images/2_php_2.png)

在 PHP7.2 版本中已经被弃用了，在新版本中使用 `openssl_encrypt` 和 `openssl_decrypt` 两个方法。

示例代码（类库）：

```
class Aes
{
    /**
     * var string $method 加解密方法
     */
    protected $method;

    /**
     * var string $secret_key 加解密的密钥
     */
    protected $secret_key;

    /**
     * var string $iv 加解密的向量
     */
    protected $iv;

    /**
     * var int $options
     */
    protected $options;

    /**
     * 构造函数
     * @param string $key     密钥
     * @param string $method  加密方式
     * @param string $iv      向量
     * @param int    $options
     */
    public function __construct($key = '', $method = 'AES-128-CBC', $iv = '', $options = OPENSSL_RAW_DATA)
    {
        $this->secret_key = isset($key) ? $key : 'CWq3g0hgl7Ao2OKI';
        $this->method = in_array($method, openssl_get_cipher_methods()) ? $method : 'AES-128-CBC';
        $this->iv = $iv;
        $this->options = in_array($options, [OPENSSL_RAW_DATA, OPENSSL_ZERO_PADDING]) ? $options : OPENSSL_RAW_DATA;
    }

    /**
     * 加密
     * @param string $data 加密的数据
     * @return string
     */
    public function encrypt($data = '')
    {
        return base64_encode(openssl_encrypt($data, $this->method, $this->secret_key, $this->options, $this->iv));
    }

    /**
     * 解密
     * @param string $data 解密的数据
     * @return string
     */
    public function decrypt($data = '')
    {
        return openssl_decrypt(base64_decode($data), $this->method, $this->secret_key, $this->options, $this->iv);
    }
}
```

示例代码：

```
$aes = new Aes('HFu8Z5SjAT7CudQc');
$encrypted = $aes->encrypt('锄禾日当午');
echo '加密前：锄禾日当午<br>加密后：', $encrypted, '<hr>';

$decrypted = $aes->decrypt($encrypted);
echo '加密后：', $encrypted, '<br>解密后：', $decrypted;
```

运行结果：

![](https://github.com/xinliangnote/PHP/blob/master/images/2_php_3.png)

## 非对称加密

#### 定义

需要两个密钥来进行加密和解密，这两个秘钥分别是公钥（public key）和私钥（private key），这种方法称为非对称加密。

#### 常用算法

- RSA

#### 优点

与对称加密相比，安全性更好，加解密需要不同的密钥，公钥和私钥都可进行相互的加解密。

#### 缺点

加密和解密花费时间长、速度慢，只适合对少量数据进行加密。

#### 应用场景

适合于对安全性要求很高的场景，适合加密少量数据，比如支付数据、登录数据等。

#### RSA 与 RSA2

算法名称 | 标准名称 | 备注
--- | --- | --- 
RSA2 | SHA256WithRSA | 强制要求RSA密钥的长度至少为2048
RSA  | SHA1WithRSA   | 对RSA密钥的长度不限制，推荐使用2048位以上

RSA2 比 RSA 有更强的安全能力。

蚂蚁金服，新浪微博 都在使用 RSA2 算法。

创建公钥和私钥：

```
openssl genrsa -out private_key.pem 2048
openssl rsa -in private_key.pem -pubout -out public_key.pem
```

执行上面命令，会生成 `private_key.pem` 和 `public_key.pem` 两个文件。

示例代码（类库）：

```
class Rsa2 
{
    private static $PRIVATE_KEY = 'private_key.pem 内容';
    private static $PUBLIC_KEY  = 'public_key.pem 内容';

    /**
     * 获取私钥
     * @return bool|resource
     */
    private static function getPrivateKey()
    {
        $privateKey = self::$PRIVATE_KEY;
        return openssl_pkey_get_private($privateKey);
    }

    /**
     * 获取公钥
     * @return bool|resource
     */
    private static function getPublicKey()
    {
        $publicKey = self::$PUBLIC_KEY;
        return openssl_pkey_get_public($publicKey);
    }

    /**
     * 私钥加密
     * @param string $data
     * @return null|string
     */
    public static function privateEncrypt($data = '')
    {
        if (!is_string($data)) {
            return null;
        }
        return openssl_private_encrypt($data,$encrypted,self::getPrivateKey()) ? base64_encode($encrypted) : null;
    }

    /**
     * 公钥加密
     * @param string $data
     * @return null|string
     */
    public static function publicEncrypt($data = '')
    {
        if (!is_string($data)) {
            return null;
        }
        return openssl_public_encrypt($data,$encrypted,self::getPublicKey()) ? base64_encode($encrypted) : null;
    }

    /**
     * 私钥解密
     * @param string $encrypted
     * @return null
     */
    public static function privateDecrypt($encrypted = '')
    {
        if (!is_string($encrypted)) {
            return null;
        }
        return (openssl_private_decrypt(base64_decode($encrypted), $decrypted, self::getPrivateKey())) ? $decrypted : null;
    }

    /**
     * 公钥解密
     * @param string $encrypted
     * @return null
     */
    public static function publicDecrypt($encrypted = '')
    {
        if (!is_string($encrypted)) {
            return null;
        }
        return (openssl_public_decrypt(base64_decode($encrypted), $decrypted, self::getPublicKey())) ? $decrypted : null;
    }

    /**
     * 创建签名
     * @param string $data 数据
     * @return null|string
     */
    public function createSign($data = '')
    {
        if (!is_string($data)) {
            return null;
        }
        return openssl_sign($data, $sign, self::getPrivateKey(), OPENSSL_ALGO_SHA256) ? base64_encode($sign) : null;
    }

    /**
     * 验证签名
     * @param string $data 数据
     * @param string $sign 签名
     * @return bool
     */
    public function verifySign($data = '', $sign = '')
    {
        if (!is_string($sign) || !is_string($sign)) {
            return false;
        }
        return (bool)openssl_verify($data, base64_decode($sign), self::getPublicKey(), OPENSSL_ALGO_SHA256);
    }
}
```

示例代码：

```
$rsa2 = new Rsa2();
        
$privateEncrypt = $rsa2->privateEncrypt('锄禾日当午');
echo '私钥加密后:'.$privateEncrypt.'<br>';

$publicDecrypt = $rsa2->publicDecrypt($privateEncrypt);
echo '公钥解密后:'.$publicDecrypt.'<br>';

$publicEncrypt = $rsa2->publicEncrypt('锄禾日当午');
echo '公钥加密后:'.$publicEncrypt.'<br>';

$privateDecrypt = $rsa2->privateDecrypt($publicEncrypt);
echo '私钥解密后:'.$privateDecrypt.'<br>';

$sign = $rsa2->createSign('锄禾日当午');
echo '生成签名:'.$privateEncrypt.'<br>';

$status = $rsa2->verifySign('锄禾日当午', $sign);
echo '验证签名:'.($status ? '成功' : '失败') ;
```

运行结果：

部分数据截图如下：

![](https://github.com/xinliangnote/PHP/blob/master/images/2_php_4.png)

#### JS-RSA

**JSEncrypt** ：用于执行OpenSSL RSA加密、解密和密钥生成的Javascript库。

Git源：https://github.com/travist/jsencrypt

**应用场景：**

我们在做 WEB 的登录功能时一般是通过 Form 提交或 Ajax 方式提交到服务器进行验证的。

为了防止抓包，登录密码肯定要先进行一次加密（RSA），再提交到服务器进行验证。

一些大公司都在使用，比如淘宝、京东、新浪 等。

示例代码就不提供了，Git上提供的代码是非常完善的。

## 密钥安全管理

这些加密技术，能够达到安全加密效果的前提是 **密钥的保密性**。

实际工作中，不同环境的密钥都应该不同（开发环境、预发布环境、正式环境）。

那么，应该如何安全保存密钥呢？

#### 环境变量

将密钥设置到环境变量中，每次从环境变量中加载。

#### 配置中心

将密钥存放到配置中心，统一进行管理。

#### 密钥过期策略

设置密钥有效期，比如一个月进行重置一次。

**在这里希望大佬提供新的思路 ~**

## 接口调试工具

#### Postman

一款功能强大的网页调试与发送网页 HTTP 请求的 Chrome插件。

这个不用多介绍，大家肯定都使用过。

#### SocketLog

Git源：https://github.com/luofei614/SocketLog

**解决的痛点：**

- 正在运行的API有Bug，不能在文件中使用var_dump进行调试，因为会影响到client的调用。将日志写到文件中，查看也不是很方便。

- 我们在二次开发一个新系统的时候，想查看执行了哪些Sql语句及程序的warning，notice等错误信息。

SocketLog，可以解决以上问题，它通过WebSocket将调试日志输出到浏览器的console中。

**使用方法**

1. 安装、配置Chrome插件
2. SocketLog服务端安装
3. PHP中用SocketLog调试
4. 配置日志类型和相关参数

## 在线接口文档

接口开发完毕，需要给请求方提供接口文档，文档的编写现在大部分都使用Markdown格式。

也有一些开源的系统，可以下载并安装到自己的服务器上。

也有一些在线的系统，可以在线使用同时也支持离线导出。

根据自己的情况，选择适合自己的文档平台吧。

常用的接口文档平台：

- eolinker
- Apizza
- Yapi
- RAP2
- DOClever

## 扩展

一、在 HTTP 和 RPC 的选择上，可能会有一些疑问，RPC框架配置比较复杂，明明用HTTP能实现为什么要选择RPC？

下面简单的介绍下 HTTP 与 RPC 的区别。

**传输协议：**

- HTTP 基于 HTTP 协议。
- RPC 即可以 HTTP 协议，也可以 TCP 协议。

HTTP 也是 RPC 实现的一种方式。

**性能消耗：**

- HTTP 大部分基于 JSON 实现的，序列化需要时间和性能。
- RPC 可以基于二进制进行传输，消耗性能少一点。

推荐一个像 JSON ，但比 JSON 传输更快占用更少的新型序列化类库 **MessagePack**。

官网地址：https://msgpack.org/

还有一些服务治理、负载均衡配置的区别。

**使用场景：**

比如浏览器接口、APP接口、第三方接口，推荐使用 HTTP。

比如集团内部的服务调用，推荐使用 RPC。

RPC 比 HTTP 性能消耗低，传输效率高，服务治理也方便。

推荐使用的 RPC 框架：**Thrift**。

二、动态令牌

简单介绍下几种动态令牌，感兴趣的可以深入了解下。

OTP：One-Time Password 一次性密码。

HOTP：HMAC-based One-Time Password 基于HMAC算法加密的一次性密码。

TOTP：Time-based One-Time Password 基于时间戳算法的一次性密码。

使用场景：

- 公司VPN登录双因素验证
- 服务器登录动态密码验证
- 网银、网络游戏的实体动态口令牌
- 银行转账动态密码
- ...

## 小结

本文讲了设计签名验证需要满足的一些条件：可变性、时效性、唯一性、完整性。

还讲了一些加密方法：单向散列加密、对称加密、非对称加密，同时分析了各种加密方法的优缺点，**大家可以根据自己的业务特点进行自由选择**。

提供了 Aes、Rsa 相关代码示例。

分享了可以编写接口文档的在线系统。

分享了开发过程中使用的接口调试工具。

扩展中分析了 HTTP 和 RPC 的区别，动态令牌的介绍等。

还提出了一个问题，**关于如何安全的进行密钥管理？** ， 欢迎各位 前辈/大佬，提供新的思路 ~ 
