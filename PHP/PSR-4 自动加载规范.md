# PSR-4 自动加载规范

# 自动加载

为了避免歧义，文档大量使用了「能愿动词」，对应的解释如下：

- `必须 (MUST)`：绝对，严格遵循，请照做，无条件遵守；
- `一定不可 (MUST NOT)`：禁令，严令禁止；
- `应该 (SHOULD)` ：强烈建议这样做，但是不强求；
- `不该 (SHOULD NOT)`：强烈不建议这样做，但是不强求；
- `可以 (MAY)` 和 `可选 (OPTIONAL)` ：选择性高一点，在这个文档内，此词语使用较少；

> 参见：[RFC 2119](http://www.ietf.org/rfc/rfc2119.txt)

## 1. 总览

PSR-4 描述了从文件路径中 [自动加载](http://php.net/autoload) 类的规范。 它拥有非常好的兼容性，并且可以在任何自动加载规范中使用，包括 [PSR-0](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md) 。 PSR-4 规范也描述了放置 autoload 文件（就是我们经常引入的 `vendor/autoload.php`）的位置。

## 2. 规范

1. 术语「class」指的是类（classes）、接口（interfaces）、特征（traits）和其他类似的结构。

2. 全限定类名具有以下形式：

   ```php
    \<NamespaceName>(\<SubNamespaceNames>)*\<ClassName>
   ```

    1. 全限定类名**必须**拥有一个顶级命名空间名称，也称为供应商命名空间（vendor namespace）。
    2. 全限定类名**可以**有一个或者多个子命名空间名称。
    3. 全限定类名**必须**有一个最终的类名（我想意思应该是你不能这样 `\<NamespaceName>(\<SubNamespaceNames>)*\` 来表示一个完整的类）。
    4. 下划线在全限定类名中没有任何特殊含义（在 [PSR-0](https://learnku.com/docs/psr/psr-0-automatic-loading-specification) 中下划是有含义的）。
    5. 全限定类名**可以**是任意大小写字母的组合。
    6. 所有类名的引用**必须**区分大小写。

3. 全限定类名的加载过程

    1. 在全限定的类名（一个「命名空间前缀」）中，一个或多个前导命名空间和子命名空间组成的连续命名空间，不包括前导命名空间的分隔符，至少对应一个「根目录」。
    2. 「命名空间前缀」后面的相邻子命名空间与根目录下的目录名称相对应（且**必须**区分大小写），其中命名空间的分隔符表示目录分隔符。
    3. 最终的类名与以`.php` 结尾的文件名保持一致，这个文件的名字**必须**和最终的类名相匹配（意思就是如果类名是 `FooController`，那么这个类所在的文件名必须是 `FooController.php`）。

4. 自动加载文件**禁止**抛出异常，**禁止**出现任何级别的错误，也**不建议**有返回值。

## 3. 范例

下表显示了与给定的全限定类名、命名空间前缀和根目录相对应的文件的路径。

| Fully Qualified Class Name   | Namespace Prefix | Base Directory         | Resulting File Path                       |
| ---------------------------- | ---------------- | ---------------------- | ----------------------------------------- |
| \Acme\Log\Writer\File_Writer | Acme\Log\Writer  | ./acme-log-writer/lib/ | ./acme-log-writer/lib/File_Writer.php     |
| \Aura\Web\Response\Status    | Aura\Web         | /path/to/aura-web/src/ | /path/to/aura-web/src/Response/Status.php |
| \Symfony\Core\Request        | Symfony\Core     | ./vendor/Symfony/Core/ | ./vendor/Symfony/Core/Request.php         |
| \Zend\Acl                    | Zend             | /usr/includes/Zend/    | /usr/includes/Zend/Acl.php                |

想要了解一个符合规范的自动加载器的实现可以查看[示例文件](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader-examples.md)。示例中的自动加载器**禁止**被视为规范的一部分，它随时都**可能**发生改变。