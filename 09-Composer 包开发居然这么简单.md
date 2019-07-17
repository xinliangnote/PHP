## composer 是用来干嘛的？

对于不了解composer的同学来说，肯定会有这个疑问。

这个单词经常在网上看到，特别是在 GitHub 中，在使用 `Laravel`、`Yii` 时也经常看到这个词，在安装的时候推荐使用 composer 安装？难道只是安装的时候使用吗？我也使用过其他框架，为什么 `Thinkphp`、`Codeigniter` 不用composer安装呢？

带着这些疑问，我们进行学习。

composer 官方网址：https://www.phpcomposer.com

composer 是 PHP（5.3+） 用来管理依赖关系的工具。

长久以来，PHP的开源方式都是项目级的，就是说一开源就是一个项目，比如一整套的CMS（Dede、WordPress、discuz）、一整套框架（Thinkphp、Codeigniter）。为啥呢？其中一个很重要的原因是你不好拆开，如果拆开的话，没有一个有效的管理工具组合起来，导致拆开的小模块大家无人问津。

然后 composer 出现了，它就是一个有效的管理工具，它负责管理大家拆开的小模块，然后进行有效的整合，使之成为一个完整的项目。

比如，记录日志使用 monolog/monolog ，HTTP client使用：guzzlehttp/guzzle 等等。

composer 包的平台：https://packagist.org，这里面包含了大量的优秀的安装包，我们就很轻松一个 composer 命令就可以将优秀的代码用到我们项目中来。

作为一名骄傲PHPer，我们总不能永远只使用别人的开发包，我们必须自己动手开发一个包给别人用，给自己用。

我们既然知道了它有这么多的好处，就让我们去学习他吧，先从 composer 的安装说起。

## composer 是如何安装的？

官方入门文档：https://docs.phpcomposer.com/00-intro.html

![](https://github.com/xinliangnote/PHP/blob/master/images/9_php_1.jpeg)

通过上述的方法，可以进行安装完成。

接下来我们以 GitHub 结合 Composer 工具来进行示例讲解如何开发一个 Composer 包。

## composer 包是如何开发的？

比如，开发一个处理数字的 composer 包。

#### 在 GitHub上 创建一个项目

1. 登录 GitHub（如果没有账号，请进行创建），点击右上角“+”，选择“New repository”。
2. 在创建界面中，Repository name 填写“numberFormat”，Description是选填的，暂时先不填，
接着在 Public（GitHub推荐的方式，免费，所有人都能访问）和 Private（收费，指定人才能访问，2019-01-09后对个人开发者免费了）中选择“Public”，接着在勾选“Initialize this repository with a README”，点击“Create Repository”按钮后创建成功。

至此，表示在GitHub上已经创建了一个名为“numberFormat”的空项目。

接下来，需要将远程的项目 clone 到本地（Git命令行、Git客户端）进行编码。

#### 学习创建 composer.json

composer.json 有哪些参数，如何编写，请参考文档：https://docs.phpcomposer.com/04-schema.html#composer.json

一个项目要调用开发包，通过composer.json就可以知道该样去加载文件。

composer.json 可以使用两个方式创建，一种是`composer init`，另一种是手工创建。

咱们一起先执行下`composer init` 看看效果。

在本地创建numberFormat目录，然后 git clone 刚才创建的项目。

```
//进入到本地numberFormat目录
composer init

Welcome to the Composer config generator

This command will guide you through creating your composer.json config.

Package name (<vendor>/<name>) [root/number-format]:number-format/number-format

Description []:一个处理数字的包

Author [XinLiang <109760455@qq.com>, n to skip]:  //回车

Minimum Stability []: //回车

Package Type (e.g. library, project, metapackage, composer-plugin) []: //回车

License []: //回车

Define your dependencies.

Would you like to define your dependencies (require) interactively [yes]?no

Would you like to define your dev dependencies (require-dev) interactively [yes]?no

{
    "name": "number-format/number-format",
    "description": "一个处理数字的包",
    "authors": [
        {
            "name": "XinLiang",
            "email": "109760455@qq.com"
        }
    ],
    "require": {}
}

Do you confirm generation [yes]?  //回车

```

至此，本地numberFormat目录就看到 composer.json 文件了，当然可以直接在目录下按照这个格式进行手工创建，后期直接编辑该文件即可。

#### 创建项目编码内容

开发包结构如下：

--src 源码目录（必须）

--tests 单元测试目录（非必须）

我们按照既定的目录结构去创建目录和文件，然后再到composer.json里面修改一下即可。

接下来，在src目录中创建一个类（NumberFormat.php）：

```
/**
 * 数字格式化类
 * @author XinLiang
 */

namespace numberFormat;

class NumberFormat
{
    /**
     * 格式化字节
     * @param int $num       数字
     * @param int $precision 精准度
     * @return string
     */
    public static function byte_format($num = 0, $precision = 1)
    {
        if ($num >= 1000000000000)
        {
            $num = round($num / 1099511627776, $precision);
            $unit = 'TB';
        }
        elseif ($num >= 1000000000)
        {
            $num = round($num / 1073741824, $precision);
            $unit = 'GB';
        }
        elseif ($num >= 1000000)
        {
            $num = round($num / 1048576, $precision);
            $unit = 'MB';
        }
        elseif ($num >= 1000)
        {
            $num = round($num / 1024, $precision);
            $unit = 'KB';
        }
        else
        {
            return number_format($num).' Bytes';
        }

        return number_format($num, $precision).' '.$unit;
    }
}
```

修改 composer.json

```
{
    "name": "number-format/number-format",
    "description": "一个处理数字的包",
    "authors": [
        {
            "name": "XinLiang",
            "email": "109760455@qq.com"
        }
    ],
    "minimum-stability": "dev",
    "require": {
        "php": ">=5.3.0"
    },
    "autoload": {
        "psr-4": {
            "numberFormat\\": "src/"
        }
    },
    "license": "MIT"
}
```

至此，我们的开发包已经完成，接下来我们来测试下这个包是否可用。

#### 测试开发包

在本地numberFormat目录下，通过`composer install` 安装

```
composer install

Loading composer repositories with package information
Updating dependencies (including require-dev)
Nothing to install or update
Writing lock file
Generating autoload files

//表示安装成功
```

惊奇的发现，在本地numberFormat目录多一个`vendor`目录。

在tests目录创建 NumberFormatTest.php

```
/**
 * 数字格式化测试类
 * @author XinLiang
 */

require '../vendor/autoload.php';

use \numberFormat;

$number = '102400010';
echo numberFormat\NumberFormat::byte_format($number);

//输出：97.7 MB
```

至此，测试成功，接下来就是要发布到packagist平台，给广大开发者见面了。

#### 发布到 packagist 平台 

packagist.org 为 composer 安装包的平台（可用GitHub账号登录）。

1. 现将本地代码提交到GitHub。
2. 发布到 packagist 平台，登录后在首页的右上角有一个"Submit"按钮，点击即可进入开发包提交的界面。在“Repository URL (Git/Svn/Hg)”输入框中，输入GitHub项目的地址，点击“Check”按钮，稍微等待几秒钟，会显示验证成功，并显示出“Submit”按钮，点击即完成了开发包的提交了。

恭喜你，这个开发包可以在任何支持 composer 的PHP框架中使用了。

那么问题来了，刚才我们的包写的有的简陋，后期我们维护代码，新增代码还需要按照原来的方式操作一遍吗？

不！因为我们可以在GitHub平台设置代码更新，同时能让 packagist.org 自动更新，是不是很酷！

在GitHub中找到代码仓库，然后选择"settings" -> “Webhooks” ，默认是绑定自动更新的。

如果未绑定，可以这样设置："settings" -> “Webhooks” -> "Add webhook" -> 

1. Payload URL填写：“https://packagist.org/api/github” 
2. Content type填写：“application/json”
3. Secret填写：“packagist提供的token”
4. 其他的默认即可
5. 点击“Add webhook” 完成。

至此，后期我们更新代码后会自动同步到 packagist.org 上。

```
//其他开发者可以这样获取包
composer require number-format/number-format:dev-master
```

为什么会有:dev-master，为什么引用其他的包不用这样设置？

因为我们引用的其他包都是稳定包，默认为：-stable。

是因为我们 composer.json 中设置了 minimum-stability 属性，这个可以了解下“版本约束”，在这就不多说了。

当我们在发布包后，如果获取不到报错怎么办，有可能是镜像的问题。

#### composer 设置镜像地址

```
//查看全局设置
composer config -gl

//第一种：设置国内镜像
composer config -g repo.packagist composer https://packagist.phpcomposer.com

//第二种：设置国内镜像
composer config -g repo.packagist composer https://packagist.laravel-china.org

//第三种：设置国内镜像
composer config -g repos.packagist composer https://php.cnpkg.org
```

## 小结

通过这篇文章，解决了上述提到的三个问题：

1. composer 是用来干嘛的？
2. composer 是如何安装的？
3. composer 包是如何开发的？

看完后，是不是觉得 Composer 包开发原来这么简单，作为骄傲的程序员，去开发属于自己的 Composer 包吧！

