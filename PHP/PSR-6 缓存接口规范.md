# PSR-6 缓存接口规范

# 缓存接口

缓存是提升应用性能的常用手段，为框架中最通用的功能，每个框架也都推出专属的、功能多样的缓存库。这些差别使得开发人员不得不学习多种系统，而很多可能是他们并不需要的功能。此外，缓存库的开发者同样面临着一个窘境，是只支持有限数量的几个框架还是创建一堆庞大的适配器类。

一个通用的缓存系统接口可以解决掉这些问题。库和框架的开发人员能够知道缓存系统会按照他们所预期的方式工作，缓存系统的开发人员只需要实现单一的接口，而不用去开发各种各样的适配器。

本文件中的 `必须`，`不得`，`需要`，`应`，`不应`，`应该`，`不应该`，`推荐`，`可能` 和 `可选` 等能愿动词按照 [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt) 中的描述进行解释。

## 目标

本 PSR 的目标是：创建一套通用的接口规范，能够让开发人员整合到现有框架和系统，而不需要去开发框架专属的适配器类。

## 定义

- **调用类库 (Calling Library)** - 调用者，使用缓存服务的类库，这个类库调用缓存服务，调用的是此缓存接口规范的具体「实现类库」，调用者不需要知道任何「缓存服务」的具体实现。
- **实现类库 (Implementing Library)** - 此类库是对「缓存接口规范」的具体实现，封装起来的缓存服务，供「调用类库」使用。实现类库 **必须** 提供 PHP 类来实现 `Cache\CacheItemPoolInterface` 和 `Cache\CacheItemInterface` 接口。实现类库 **必须** 支持最小的如下描述的 TTL 功能，秒级别的精准度。
- **生存时间值 (TTL - Time To Live)** - 定义了缓存可以存活的时间，以秒为单位的整数值。
- **过期时间 (Expiration)** - 定义准确的过期时间点，一般为缓存存储发生的时间点加上 TTL 时间值，也可以指定一个 DateTime 对象。假如一个缓存项的 TTL 设置为 300 秒，实现类库 **可以** 让缓存项提前过期，但是 **必须** 在到达过期时间时立即把缓存项标示为过期。如果调用类库在保存一个缓存项的时候未设置「过期时间」、或者设置了 `null` 作为过期时间（或者 TTL 设置为 `null`），实现类库 **可以** 使用默认自行配置的一个时间。如果没有默认时间，实现类库 **必须**把存储时间当做 `永久性` 存储，或者按照底层驱动能支持的最长时间作为保持时间。

- **键 (KEY)** - 长度大于 1 的字串，用作缓存项在缓存系统里的唯一标识符。实现类库 **必须** 支持「键」规则 `A-Z`, `a-z`, `0-9`, `_`, 和 `.` 任何顺序的 UTF-8 编码，长度小于 64 位。实现类库 **可以** 支持更多的编码或者更长的长度，不过 **必须** 支持至少以上指定
  的编码和长度。实现类库可自行实现对「键」的转义，但是 **必须** 保证能够无损的返回「键」字串。以下的字串作为系统保留: `{}()/\@:`，**一定不可** 作为「键」的命名支持。
- **命中 (Hit)** - 一个缓存的命中，指的是当调用类库使用「键」在请求一个缓存项的时候，在缓存池里能找到对应的缓存项，并且此缓存项还未过期，并且此数据不会因为任何原因出现错误。调用类库 **应该** 确保先验证下 `isHit()` 有命中后才调用 `get()` 获取数据。
- **未命中 (Miss)** - 一个缓存未命中，是完全的上面描述的「命中」的相反。指的是当调用类库使用「键」在请求一个缓存项的时候，在缓存池里未能找到对应的缓存项，或者此缓存项已经过期，或者此数据因为任何原因出现错误。一个过期的缓存项，**必须** 被当做 `未命中` 来对待。
- **延迟 (Deferred)** - 一个延迟的缓存，指的是这个缓存项可能不会立刻被存储到物理缓存池里。一个缓存池对象 **可以** 对一个指定延迟的缓存项进行延迟存储，这样做的好处是可以利用一些缓存服务器提供
  的批量插入功能。缓存池 **必须** 能对所有延迟缓存最终能持久化，并且不会丢失。**可以** 在调用类库还未发起保存请求之前就做持久化。当调用类库调用 `commit()` 方法时，所有的延迟缓存都 **必须** 做持久化。实现类库 **可以** 自行决定使用什么逻辑来触发数据持久化，如对象的 `析构方法 (destructor)` 内、调用 `save()` 时持久化、倒计时保存或者触及最大数量时保存等。当请求一个延迟缓存项时，**必须** 返回一个延迟，未持久化的缓存项对象。

## 数据

实现类库必须支持序列化 `PHP` 的所有数据类型，包含以下类型：

- Strings - `PHP` 兼容编码中任意长度的字符串。
- Integers - `PHP` 支持的所有大小的整数，最大为 64 位有符号整数。
- Floats - 有符号的浮点值。
- Boolean - True 和 False 。
- Null - 实际的空值。
- Arrays - 可索引，关联，任意深度的数组。
- Object - 所有对象支持无损的序列化和反序列化，比如 `$o == unserialize(serialize($o))`。对象可以使用 PHP 的原生序列化接口， `__sleep()` 、 `__wakeup()` 等魔术方法，或者其他可用的类似语法。

所有数据传入实现类库 **必须** 准确返回传入的值，并且返回变量的类型一致。非法示例：传入 (int) 5 返回 (string) 5。实现类库 **可以** 使用 `PHP` 中 serialize ()/unserialize () 方法进行序列化和反序列化，当然也不强制要求这样做。兼容这些的一个基本准线就是接受对象类型。

如果由于某些技术原因不能准确保存，实现类库 **必须** 返回缓存未命中，而不是损坏的数据。

## 主要概念

### 缓存池 Pool

缓存池包含缓存系统里所有缓存数据的集合。缓存池逻辑上是所有缓存项存储的仓库，所有存储进去的数据，都能从缓存池里取出来，所有的对缓存的操作，都发生在缓存池子里。

### 缓存项 Items

一条缓存项在缓存池里代表了一对「键 / 值」对应的数据，「键」被视为每一个缓存项主键，是缓存项的唯一标识符，**必须** 是不可变更的，当然，「值」**可以** 任意变更。

## 错误处理

缓存对应用性能起着至关重要的作用，但是，无论在任何情况下，缓存 **一定不可** 作为应用程序不可或缺的核心功能。缓存系统里的错误 **一定不可** 导致应用程序故障，所以，实现类库 **一定不可** 抛出任何除了此接口规范定义的以外的异常，并且 **必须** 捕捉包括底层存储驱动抛出的异常，不让其冒泡至超出缓存系统内。

实现类库 **应该** 对此类错误进行记录，或者以任何形式通知管理员。

调用类库发起删除缓存项的请求，或者清空整个缓冲池子的请求，「键」不存在的话 **必须** 不能当成是有错误发生。后置条件是一样的，如果取数据时，「键」不存在的话 **必须** 不能当成是有错误发生。

## 接口

### CacheItemInterface

`CacheItemInterface` 定义了缓存系统里的一个缓存项。每一个缓存项 **必须** 有一个「键」与之相关联，此「键」通常是通过 Cache\CacheItemPoolInterface 来设置。

Cache\CacheItemInterface 对象把缓存项的存储进行了封装，每一个 Cache\CacheItemInterface 由一个 Cache\CacheItemPoolInterface 对象生成，CacheItemPoolInterface 负责一些必须的设置，并且给对象设置具有 `唯一性` 的「键」。

Cache\CacheItemInterface 对象 **必须** 能够存储和取出任何类型的，在「数据」章节定义的 PHP 数值。

调用类库 **一定不可** 擅自初始化「CacheItemInterface」对象，「缓存项」只能使用「CacheItemPoolInterface」对象的 `getItem()` 方法来获取。调用类库 **一定不可** 假设由一个实现类库创建的「缓存项」能被另一个实现类库完全兼容。

```php
<?php

namespace Psr\Cache;

/**
 * CacheItemInterface 定了缓存系统里对缓存项操作的接口
 */
interface CacheItemInterface
{
    /**
     * 返回当前缓存项的「键」
     *
     * 「键」由实现类库来加载，并且高层的调用者（如：CacheItemPoolInterface）
     * **应该** 能使用此方法来获取到「键」的信息。
     *
     * @return string
     *  当前缓存项的「键」
     */
    public function getKey();

    /**
     * 凭借此缓存项的「键」从缓存系统里面取出缓存项。
     *
     * 取出的数据 **必须** 跟使用 `set()` 存进去的数据是一模一样的。
     *
     * 如果 `isHit()` 返回 false 的话，此方法必须返回 `null`，需要注意的是 `null` 
     * 本来就是一个合法的缓存数据，所以你 **应该** 使用 `isHit()` 方法来辨别到底是
     * "返回 null 数据" 还是 "缓存里没有此数据"
     *
     * @return mixed
     *   此缓存项的「键」对应的「值」，如果找不到的话，返回 `null`
     */
    public function get();

    /**
     * 确认缓存项的检查是否命中。
     *
     * 注意: 调用此方法和调用 `get()` 时 **一定不可** 有先后顺序之分。
     *
     * @return bool
     * 如果缓冲池里有命中的话，返回 `true`，反之返回 `false`
     */
    public function isHit();

    /**
     * 为此缓存项设置「值」。
     *
     * 参数 $value 可以是所有能被 PHP 序列化的数据，序列化的逻辑
     * 需要在实现类库里书写。
     *
     * @param mixed $value
     * 将被存储的可序列化的数据。
     *
     * @return static
     * 返回当前对象。
     */
    public function set($value);

    /**
     * 设置缓存项的准确过期时间点。
     *
     * @param \DateTimeInterface|null $expiration
     * 过期的准确时间点，过了这个时间点后，缓存项就 **必须** 被认为是过期了的。
     * 如果明确的传参 `null` 的话，**可以** 使用一个默认的时间。
     * 如果没有设置的话，缓存 **应该** 存储到底层实现的最大允许时间。
     *
     * @return static
     * 返回当前对象
     */
    public function expiresAt($expiration);

    /**
     * 设置缓存项的过期时间
     *
     * @param int|\DateInterval|null $time
     * 以秒为单位的过期时长，过了这段时间后，缓存项就 **必须** 被认为是过期了的。
     * 如果明确的传参 `null` 的话，**可以** 使用一个默认的时间。
     * 如果没有设置的话，缓存 **应该** 存储到底层实现的最大允许时间。
     *
     * @return static
     * 返回当前对象
     */
    public function expiresAfter($time);

}
```

### CacheItemPoolInterface

Cache\CacheItemPoolInterface 的主要目的是从调用类库接收「键」，然后返回对应的 Cache\CacheItemInterface 对象。

此接口也是作为主要的，与整个缓存集合交互的方式。所有的配置和初始化由实现类库自行实现。

```php
<?php

namespace Psr\Cache;

/**
 * CacheItemPoolInterface 生成 CacheItemInterface 对象
 */
interface CacheItemPoolInterface
{
    /**
     * 返回「键」对应的一个缓存项。
     *
     * 此方法 **必须** 返回一个 CacheItemInterface 对象，即使是找不到对应的缓存项
     * 也 **一定不可** 返回 `null`。
     *
     * @param string $key
     * 用来搜索缓存项的「键」。
     *
     * @throws InvalidArgumentException
     *   如果 $key 不是合法的值，\Psr\Cache\InvalidArgumentException 异常会被抛出。
     *
     * @return CacheItemInterface
     *  对应的缓存项。
     */
    public function getItem($key);

    /**
     * 返回一个可供遍历的缓存项集合。
     *
     * @param string[] $keys
     * 由一个或者多个「键」组成的数组。
     *
     * @throws InvalidArgumentException
     *   如果 $keys 里面有哪个「键」不是合法，\Psr\Cache\InvalidArgumentException 异常
     *   会被抛出。
     *
     * @return array|\Traversable
     *   返回一个可供遍历的缓存项集合，集合里每个元素的标识符由「键」组成，即使即使是找不到对的缓存项，也要返回一个「CacheItemInterface」对象到对应的「键」中。
     *   如果传参的数组为空，也需要返回一个空的可遍历的集合。
     */
    public function getItems(array $keys = array());

    /**
     * 检查缓存系统中是否有「键」对应的缓存项。
     *
     * 注意: 此方法应该调用 `CacheItemInterface::isHit()` 来做检查操作，而不是 `CacheItemInterface::get()`
     *
     * @param string $key
     * 用来搜索缓存项的「键」。
     *
     * @throws InvalidArgumentException
     *   如果 $key 不是合法的值，\Psr\Cache\InvalidArgumentException 异常会被抛出。
     *
     * @return bool
     *   如果存在「键」对应的缓存项即返回 true，否则 false
     */
    public function hasItem($key);

    /**
     * 清空缓冲池
     *
     * @return bool
     *   成功返回 true，有错误发生返回 false
     */
    public function clear();

    /**
     * 从缓冲池里移除某个缓存项
     *
     * @param string $key
     *   用来搜索缓存项的「键」。
     *
     * @throws InvalidArgumentException
     * 如果 $key 不是合法的值，\Psr\Cache\InvalidArgumentException 异常会被抛出。
     *
     * @return bool
     *   成功返回 true，有错误发生返回 false
     */
    public function deleteItem($key);

    /**
     * 从缓冲池里移除多个缓存项
     *
     * @param string[] $keys
     * 由一个或者多个「键」组成的数组。
     * 
     * @throws InvalidArgumentException
     * 如果 $keys 里面有哪个「键」不是合法，\Psr\Cache\InvalidArgumentException 异常会被抛出。
     *
     * @return bool
     * 成功返回 true，有错误发生返回 false
     */
    public function deleteItems(array $keys);

    /**
     * 立刻为「CacheItemInterface」对象做数据持久化。
     *
     * @param CacheItemInterface $item
     * 将要被存储的缓存项
     *
     * @return bool
     * 成功返回 true，有错误发生返回 false
     */
    public function save(CacheItemInterface $item);

    /**
     * 稍后为「CacheItemInterface」对象做数据持久化。
     *
     * @param CacheItemInterface $item
     * 将要被存储的缓存项
     *
     * @return bool
     * 成功返回 true，有错误发生返回 false
     */
    public function saveDeferred(CacheItemInterface $item);

    /**
     * 提交所有的正在队列里等待的请求到数据持久层，配合 `saveDeferred()` 使用
     *
     * @return bool
     * 成功返回 true，有错误发生返回 false
     */
    public function commit();
}
```

### CacheException

异常接口用于严重错误发生的时候，包括但不限于缓存设置错误，例如：无法连接到缓存服务器、提供了无效的凭证。

库抛出的任何异常都必须实现此接口。

```php
<?php

namespace Psr\Cache;

/**
 * 异常接口，针对库抛出的所有异常。
 */
interface CacheException
{
}
```

### InvalidArgumentException

```php
<?php

namespace Psr\Cache;

/**
 * 无效缓存参数的异常接口。
 *
 * 任何时候，一个无效参数传递到方法时，必须抛出一个实现了
 * Psr\Cache\InvalidArgumentException 的异常类。
 */
interface InvalidArgumentException extends CacheException
{
}
```

