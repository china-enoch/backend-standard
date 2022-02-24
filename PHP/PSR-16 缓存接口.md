# PSR-16 缓存接口

# 缓存库公共接口

本文档描述了一个简单也易扩展的接口，针对缓存项目和缓存驱动。

本文档中的关键字「必须」、「不得」、「要求」、「应」、「不应」、「应该」、「不应该」、「推荐」、「可能」、「可选」，使用 [RFC 2119](http://tools.ietf.org/html/rfc2119) 中的描述进行解释。

最终的实现相比本提议可能会有更多功能，当然它们必须首先实现指明的接口、功能。

# 1. 规范

## 1.1 简介

使用缓存是提升项目性能的通用方法，这使得缓存功能成为许多框架和库最常见的功能之一。如果各个缓存库提供相同的使用接口，意味着库可以丢弃他们自己的缓存实现方式，然后方便的使用框架中的缓存功能，或者使用其他专门的缓存库。

PSR-6 已经解决了这个问题，但是在一些简单的用例中显得过于繁琐。这个标准为大部分情况构建更加简单的接口标准。它独立于 PSR-6，但尽可能的兼容 PSR-6。

## 1.2 定义

调用库，实现库，TTL，过期和 Key 都是从 PSR-6 复制而来，因为他们意义相同。

- 调用库 - 需要使用缓存服务的库或者代码。调用库会使用实现了该标准（PSR-16）接口的缓存服务，但不必知道这些缓存服务的具体实现方式。

- 实现库 - 实现库负责实现这个标准（PSR-16），以便为调用库提供缓存服务。实现库 **必须** 提供实现了 `Psr\SimpleCache\CacheInterface` 接口的类。实现库 **必须** 以整数秒（s）作为缓存有效时长（TTL）的最小粒度。

- 有效时长（TTL）- 有效时长 (TTL) 是指一个缓存项从存储到过期的时间长度。TTL 一般用以秒为单位的整数或者 `DateInterval` 实例对象表示。

- 过期（Expiration） - 过期是指一个缓存项过期的具体时间。它通过缓存项保存时指定的有效时长（TTL）计算得到。

  如果一个存储项在 1:30:00 存储，有效时长（TTL）为 300 秒，那么这个存储项会在 1:35:00 过期（Expiration）。

  实现库 **可以** 让一个缓存项提前过期，但缓存项一旦到了过期时间就 **必须** 作为已过期处理。如果调用库存储一个缓存项时没有设置过期时间或者有效时长，或者设置为了 null，实现库 **可以** 指定默认值。如果没有设置默认值，实现库 **必须** 把该缓存项设置为永不过期，或者过期时长设置为系统所支持的最大长度。

  如果有效时长（TTL）被设置为负数或者 0，该缓存项 **必须** 从缓存中删除使之失效。

- Key - 用于指定缓存项唯一性的字符串，至少一个字符长度。实现库 **必须** 支持由 `A-Z`，`a-z`，`0-9`，`_` 和 `.` 以任意顺序并使用 UTF-8 编码组成的字符串作为 Key，支持的长度需要达到 64 个字符长度。实现库 **可以** 支持额外的字符和字符编码，或者支持更长的字符长度，但上面所说的必须支持。实现库存储时允许根据需要对 Key 的字符进行转义处理，但 **必须** 能够返回未经处理过的原始 Key 字符串。以下字符作为保留字段，实现库 **必须不能** 使用它们：`{}()/\@:`

- 缓存（Cache）- 实现了 `Psr\SimpleCache\CacheInterface` 接口的对象。

- 缓存未命中（Cache Misses） - 缓存未命中时会返回 `null`，因此检查一个缓存项保存的值是否为 `null` 是不可能的。这是跟 PSR-6 主要的不同点。

## 1.3 缓存

如果对一个特定的缓存条目没有指定一个默认的 `TTL`，实现 **可以** 提供一个用户指定的机制。如果未提供用户指定的默认值，则实现 **必须** 默认为底层实现提供一个允许的最大合法值。如果底层实现不支持 `TTL`，则用户指定的 `TTL` **必须** 静默忽略。

## 1.4 数据

实现库 **必须** 支持所有序列化的 PHP 数据类型，包括：

- `Strings` - 任何 PHP 兼容编码中的任意大小的字符串。
- `Integers` - PHP 支持的任何大小的所有整数，高达 64 位的签名。
- `Floats` - 所有签名的浮点值。
- `Boolean` - True 和 False。
- `Null` - 空值（尽管它当从一个未命中的缓存中读取时不能区分）。
- `Arrays` - 索引，关联和任意深度的的多维数组。
- `Object` - 任何支持像这样 `$o == unserialize(serialize($o))` 无损序列化和反序列化的对象。对象 **可以** 利用 PHP 的可序列化接口，`__sleep()` 和 `__wakeup()` 魔术方法，或者相似语言的功能，如果合适的话。

传递到实现库中的所有数据 **必须** 完全按照传递的方式返回。这包括变量类型。也就是说，如果 `(int)5` 是要保存的值，返回 `(string)5` 的将是错误的。实现库 **可以** 使用 PHP 内置的 serialize ()/unserialize () 方法，但不需要这么做。与它们兼容被简单用作可接受对象值的基线。

如果由于任何原因无法返回确切要保存的值，实现库 **必须** 响应缓存未命名而不是损坏的数据。

# 2. 接口列表

## 2.1 CacheInterface 接口

缓存接口定义了基于缓存实体的最基本的操作，其包括读写和删除单个缓存项目。

另外该接口还定义了处理多个缓存项目的方法，如：一次性写入，读取，删除多个项目的操作。当你需要执行大量的读 / 写操作时很有用，仅仅一个单次访问缓存服务器便可执行操作多个项目，从而显著的减少延迟时间。

一个 CacheInterface 实例对应一个拥有单个键命令空间的缓存集合，其等价于在 PSR-6 中的 “Pool”，不同的 CacheInterface 实例可以被相同的 datastore 支持，但必须有在逻辑上是独立的。

```php
<?php

namespace Psr\SimpleCache;

interface CacheInterface
{
    /**
     * 从缓存中取出值
     *
     * @param string $key     该项在缓存中唯一的key值
     * @param mixed  $default key不存在时，返回的默认值
     *
     * @return mixed 从缓存中返回的值，或者是不存在时的默认值
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   如果给定的key不是一个合法的字符串时，抛出该异常
     */
    public function get($key, $default = null);

    /**
     * 存储值在cache中，唯一关键到一个key及一个可选的存在时间
     *
     * @param string                 $key   存储项目的key.
     * @param mixed                  $value 存储的值，必须可以被序列化的
     * @param null|int|\DateInterval $ttl   可选项.项目的存在时间，如果该值没有设置，且驱动支持生存时间时，将设置一个默认值，或者驱自行处理。
     *
     * @return bool true 存储成功  false 存储失败
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *  如果给定的key不是一个合法的字符串时，抛出该异常。
     */
    public function set($key, $value, $ttl = null);

    /**
     * 删除指定键值的缓存项
     *
     * @param string $key 指定的唯一缓存key对应的项目将会被删除
     *
     * @return bool 成功删除时返回ture，有其它错误时时返回false
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   如果给定的key不是一个合法的字符串时，抛出该异常。
     */
    public function delete($key);

    /**
     * 清除所有缓存中的key
     *
     * @return bool 成功返回True.失败返回False
     */
    public function clear();

    /**
     * 根据指定的缓存键值列表获取得多个缓存项目
     *
     * @param iterable $keys   在单次操作中可被获取的键值项
     * @param mixed    $default 如果key不存在时，返回的默认值
     *
     * @return iterable  返回键值对（key=>value形式）列表。如果key不存在，或者已经过期时，返回默认值。
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *  如果给定的keys既不是合法的数组，也不可以被转成数组，或者给得的任何一个key不是一个合法的值时，拖出该异常。
     */
    public function getMultiple($keys, $default = null);

    /**
     * 存储一个键值对形式的集合到缓存中。
     *
     * @param iterable               $values 一系列操作的键值对列表
     * @param null|int|\DateInterval $ttl     可选项.项目的存在时间，如果该值没有设置，且驱动支持生存时间时，将设置一个默认值，或者驱自行处理。
     *
     * @return bool 成功返回True.失败返回False.
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   如果给定的keys既不是合法的数组，也不可以被转成数组，或者给得的任何一个key不是一个合法的值时，拖出该异常.
     */
    public function setMultiple($values, $ttl = null);

    /**
     *  单次操作删除多个缓存项目.
     *
     * @param iterable $keys 一个基于字符串键列表会被删除
     *
     * @return bool True 所有项目都成功被删除时回true,有任何错误时返回false
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   如果给定的keys既不是合法的数组，也不可以被转成数组，或者给得的任何一个key不是一个合法的值时，拖出该异常.
     */
    public function deleteMultiple($keys);

    /**
     * 判断一个项目在缓存中是否存在
     *
     * 注意: has()方法仅仅在缓存预热的场景被推荐使用且不允许的活跃     * 的应用中场景中对get/set方法使用, 因为方法受竞态条件的限制，当     * 你调用has()方法时会立即返回true。另一个脚本可以删除它，使应     * 用状态过期。
     * @param string $key 缓存键值
     *
     * @return bool  
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *    如果给定的key不是一个合法的字符串时，抛出该异常.
     */
    public function has($key);
}
```

## 2.2 CacheException

```php
<?php

namespace Psr\SimpleCache;

/**
 * 库抛出异常的接口，用于所有类型异常。
 */
interface CacheException
{
}
```

## 2.3 InvalidArgumentException

```php
<?php

namespace Psr\SimpleCache;

/**
 * 无效缓存参数异常的接口。
 *
 * 当传递一个无效参数时，必须抛出一个实现了此接口的异常。
 */
interface InvalidArgumentException extends CacheException
{
}
```

