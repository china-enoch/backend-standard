# PSR-11 容器接口

# 容器接口

本文描述了依赖注入容器的通用接口。

设定 `ContainerInterface` 的目的是为了标准化框架或类库如何使用容器来获取对象和参数。（本文其它部分称之为 **实体** ）。

为了避免歧义，文档大量使用了「能愿动词」，对应的解释如下：

- `必须 (MUST)`：绝对，严格遵循，请照做，无条件遵守；
- `一定不可 (MUST NOT)`：禁令，严令禁止；
- `应该 (SHOULD)` ：强烈建议这样做，但是不强求；
- `不该 (SHOULD NOT)`：强烈不建议这样做，但是不强求；
- `可以 (MAY)` 和 `可选 (OPTIONAL)` ：选择性高一点，在这个文档内，此词语使用较少；

> 参见：[RFC 2119](http://www.ietf.org/rfc/rfc2119.txt)

本文中关键字 `implementor` 被看作某些在依赖注入相关的框架或类库中实现了 `ContainerInterface` 接口。使用依赖注入容器（DIC）的用户被看作 `user` 。

## 1. 规范

### 1.1 基础知识

#### 1.1.1 实体标识符

实体标识符是一个任何合法的 PHP 字符串，它至少包含 1 个字符的，它用来唯一标识容器里的一个对象。实体标识符只是一个不透明的字符串，所以调用者不应该通过语义去猜测它具有的结构。

#### 1.1.2 容器的方法

- `Psr\Container\ContainerInterface` 接口提供了两个方法： `get` 和 `has`。
- `get` 方法有一个必传的参数：一个字符串格式的实体标识符。 `get` 方法可以返回任何类型的值，或者在容器没有标识符对应值的时候抛出一个 `NotFoundExceptionInterface` 接口实现类的异常。连续两次使用相同参数调用 `get` 方法得到的值应该是相同的，然而，这取决于 `implementor` 实现类的设计和 `user` 用户配置，可能也会返回不同的值。所以 `user` 用户不应该依赖在两次连续调用时可以获得相同的值。
- `has` 方法需要一个唯一参数：一个字符串格式的实体标识符。如果容器内有标识符对应的内容时 `has` 方法返回 `true` 值；否则 `has` 方法返回 `false` 。如果调用 `has($id)` 返回了 false ，那么相同 $id 调用 `get($id)` 方法一定是抛出 `NotFoundExceptionInterface` 接口的异常。

### 1.2 异常

容器抛出的异常都需要实现 [`Psr\Container\ContainerExceptionInterface`](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md#container-exception) 接口。

通过 `get` 方法获取一个容器中不存在实体标识符时必须抛出 [`Psr\Container\NotFoundExceptionInterface`](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md#not-found-exception) 接口的异常实现类。

### 1.3 推荐用法

用户 **不应该** 将容器作为参数传入对象然后在对象中通过容器获得对象的依赖。这样是把容器当作 [服务定位器](https://en.wikipedia.org/wiki/Service_locator_pattern) 使用，而服务定位器是一个不受欢迎的模式。

相关的详情信息，请查看文档的第 4 部分。

## 2. 包

[psr/container](https://packagist.org/packages/psr/container) 包中提供了上面提到的接口和相关异常类。

实现 PSR 容器接口的包应该申明为 `psr/container-implementation` `1.0.0` 包。

需要使用容器的项目只需要引入上面实现的包 `psr/container-implementation` `1.0.0` 即可。

## 3. 接口

### 3.1. `Psr\Container\ContainerInterface`

```text-html-php
<?php
namespace Psr\Container;

/**
 * 容器的接口类，提供了获取容器中对象的方法。
 */
interface ContainerInterface
{
    /**
     * 在容器中查找并返回实体标识符对应的对象。
     *
     * @param string $id 查找的实体标识符字符串。
     *
     * @throws NotFoundExceptionInterface  容器中没有实体标识符对应对象时抛出的异常。
     * @throws ContainerExceptionInterface 查找对象过程中发生了其他错误时抛出的异常。
     *
     * @return mixed 查找到的对象。
     */
    public function get($id);

    /**
     * 如果容器内有标识符对应的内容时，返回 true 。
     * 否则，返回 false。
     *
     * 调用 `has($id)` 方法返回 true，并不意味调用  `get($id)` 不会抛出异常。
     * 而只意味着 `get($id)` 方法不会抛出 `NotFoundExceptionInterface` 实现类的异常。
     *
     * @param string $id 查找的实体标识符字符串。
     *
     * @return bool
     */
    public function has($id);
}
```

### 3.2. `Psr\Container\ContainerExceptionInterface`

```text-html-php
<?php
namespace Psr\Container;

/**
 * 容器中的基础异常类。
 */
interface ContainerExceptionInterface
{
}
```

### 3.3. `Psr\Container\NotFoundExceptionInterface`

```text-html-php
<?php
namespace Psr\Container;

/**
 * 容器中没有查找到对应对象时的异常
 */
interface NotFoundExceptionInterface extends ContainerExceptionInterface
{
}
```

