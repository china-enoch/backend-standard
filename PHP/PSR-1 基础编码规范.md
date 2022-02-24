# PSR-1 基础编码规范

# 基本代码规范

本篇规范制定了代码基本元素的相关标准，以确保共享的 PHP 代码间具有较高程度的技术互通性。

本文件中的 `必须`，`不得`，`需要`，`应`，`不应`，`应该`，`不应该`，`推荐`，`可能` 和 `可选` 等能愿动词按照 [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt) 中的描述进行解释。

## 1. 概览

- PHP 代码文件 **必须** 以 `<?php` 或 `<?=` 标签开始；
- PHP 代码文件 **必须** 以 `不带 BOM 的 UTF-8` 编码；
- PHP 代码中 **应该** 声明任一标志（类、函数、常量等），或引起`副作用`（如果一个函数修改了自己范围之外的资源，那就叫做有副作用，如：生成输出以及修改 .ini 配置文件等），但是**不应该**二者都有；
- 命名空间以及类 **必须** 符合 PSR 的自动加载规范： [[PSR-0](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md)（已废弃）或 [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md)] 中的一个。
- 类的命名 **必须** 遵循 `StudlyCaps` 大写开头的驼峰命名规范；
- 类中的常量所有字母都 **必须** 大写，单词间用下划线分隔；
- 方法名称 **必须** 符合 `camelCase` 式的小写开头驼峰命名规范。

## 2. 文件

### 2.1. PHP 标签

PHP 代码 **必须** 使用 `<?php ?>` 长标签 或 `<?= ?>` 短输出标签；
**一定不可** 使用其它自定义标签。

### 2.2. 字符集编码

PHP 代码 **必须** 且只可使用 `不带 BOM 的 UTF-8` 编码。

### 2.3. 副作用

一份 PHP 文件中 **应该** 要不就只定义新的声明，如类、函数或常量等不产生 `副作用` 的操作，要不就只执行会产生 `副作用` 的逻辑操作，但 **不该** 同时具有两者。

「副作用」(side effects) 一词的意思是，仅仅通过包含文件，不直接声明类、函数和常量等，而执行的逻辑操作。

「副作用」包含却不仅限于：生成输出，明确使用 require 或 include，连接到外部服务，修改 ini 设置，发出错误或异常，修改全局或静态变量，读取或写入一个文件，等等。

以下是一个 `反例`，一份包含「函数声明」以及产生「副作用」的代码：

```php
<?php
// 「副作用」：修改 ini 配置
ini_set('error_reporting', E_ALL);

// 「副作用」：引入文件
include "file.php";

// 「副作用」：生成输出
echo "<html>\n";

// 声明函数
function foo()
{
    // function body
}
```

下面是一个范例，一份只包含声明不产生「副作用」的代码：

```php
<?php
// 声明函数
function foo()
{
    // 函数主体部分
}

// 条件声明 **不** 属于「副作用」
if (! function_exists('bar')) {
    function bar()
    {
        // 函数主体部分
    }
}
```

## 3. 命名空间和类名

命名空间和类名 **必须** 遵循『自动加载』规范： [[PSR-0](https://learnku.com/docs/psr/psr-0-automatic-loading-specification)， [PSR-4](https://learnku.com/docs/psr/psr-4-autoloader)]。

这意味着每个类都独立为一个文件，并且至少在一个层次的命名空间内，那就是：顶级组织名（vendor name）。

类名 **必须** 以类似 `StudlyCaps` 形式的大写开头的驼峰命名方式声明。

PHP 5.3 及更高版本的代码 **必须** 使用正式的命名空间。

举个例子：

```text-html-php
<?php
// PHP 5.3 及更高版本：
namespace Vendor\Model;

class Foo
{
}
```

PHP 5.2 及更低版本 **应该** 使用伪命名空间，约定俗成，以顶级组织名称 `Vendor_` 为类名前缀：

```text-html-php
<?php
// PHP 5.2.x 及更低版本：
class Vendor_Model_Foo
{
}
```

## 4. 类的常量、属性和方法

此处的「类」指代所有的类、接口以及可复用代码块（traits）。

### 4.1. 常量

类的常量中所有字母都 **必须** 大写，词间以下划线分隔。例如：

```php
<?php
namespace Vendor\Model;

class Foo
{
    const VERSION = '1.0';
    const DATE_APPROVED = '2012-06-01';
}
```

### 4.2. 属性

类的属性命名 **可以** 遵循：

- 大写开头的驼峰式 (`$StudlyCaps`)
- 小写开头的驼峰式 (`$camelCase`)
- 下划线分隔式 (`$under_score`)

本规范不做强制要求，但无论遵循哪种命名方式，都 **应该** 在一定的范围内保持一致。这个范围可以是整个团队、整个包、整个类或整个方法。

### 4.3. 方法

方法名称 **必须** 符合 `camelCase()` 式的小写开头驼峰命名规范。