# PSR-6 说明文档

## 1. 概述

使用缓存是一种常用的提高性能的方法，并适用于任何项目，这使得缓存库成为许多框架和库最常见的特性之一。最后导致了许多库都有自己的缓存库，并且具有不同级别的功能。这些差异导致开发人员不得不学习多个缓存系统，而他们所需要的功能可能在有的系统里并没有提供。此外，缓存库本身的开发人员只能选择要么支持少量框架，要么创建大量的适配器类。

## 2. 为什么有必要？

通用的缓存接口会解决这些问题。库和框架开发者可以期望缓存系统能正常工作，与此同时，缓存系统的开发者只用实现一部分接口而不是做一大堆适配工作。

而且，这里的实现也是为了方便未来扩展，它提供了许多本质上不同但是却又兼容 API 的实现，而且也为后面 PSRs 规范或者特定的实现提供了清晰的路径规划。

正方:

- 一个标准的缓存接口能提供独立的库来让我们轻松缓存中间数据；他们可以简单的依赖和使用这些标准接口而不用关心具体的实现细节。
- 由多个项目共享的常见开发的缓存系统，即使他们扩展了这个接口，也比单独开发的实现要健壮

反方:

- 任何接口标准化会被认为扼杀了未来创新，被认为不应该这样实现。但是我们相信缓存是一个足够商业化的问题场景，缓存接口在这里提供的扩展能力降低了任何潜在的停滞风险。

## 3. 范围

### 3.1 目标

- 一种通用的底层和中间级缓存需求接口。
- 一种清晰的机制，用于扩展规范以支持高级功能，包括将来的 PSRs 或单个实现。 此机制必须允许多个独立扩展而不会发生冲突。

### 3.2 非目标

- 与所有现有缓存的实现体系结构兼容。
- 像命名空间或标记这样由少数用户使用的高级缓存特性。

## 4. 方法

### 4.1 选择的方法

该规范采用『存储模型』或『数据映射』模型进行缓存，而不是传统可『可过期键 - 值』模型。主要原因是灵活性。简单的键 / 值模型更加难以扩展。

这里的模型要求用 CacheItem 对象和 Pool 对象，CacheItem 对象表示缓存条目，Pool 对象是缓存数据给定缓存。从池中检索项目，交互并返回到项目。有时候有些冗长，但是它提供了一个良好、稳健、灵活的缓存方法，尤其是在缓存比简单的保存在字符串更复杂的情况下。

大多数方法名称是根据成员项目和其他流行的非成员系统调查中的通用实践和方法名来选择的。

优点:

- 灵活并且可扩展
- 允许在不违反接口的情况下实现大量的变化
- 不会将对象构造函数的隐式暴露为伪接口

缺点:

- 比简单的方法更冗长

示例:

下面是一些常用的使用模式。这些是非规范的，但是可以说明一些设计决策的应用。

```php
/**
 * 获取可用控件列表。
 *
 * 在这种情况下，我们假设小部件列表很少改动 
 * 列表一直缓存到显式清除为止。
 */
function get_widget_list()
{
    $pool = get_cache_pool('widgets');
    $item = $pool->getItem('widget_list');
    if (!$item->isHit()) {
        $value = compute_expensive_widget_list();
        $item->set($value);
        $pool->save($item);
    }
    return $item->get();
}
/**
 * 可用控件缓存列表。.
 *
 * 在这种情况下，我们假设已经计算了一个小部件列表，
 * 缓存它，无论缓存的是什么。
 */
function save_widget_list($list)
{
    $pool = get_cache_pool('widgets');
    $item = $pool->getItem('widget_list');
    $item->set($list);
    $pool->save($item);
}
/**
 * 清除缓存小部件列表。
 *
 * 在这种情况下，我们只想从缓存中删除小部件。
 * 我们不在意他是否已被设置；POST的条件是『不再设置』 。
 */
function clear_widget_list()
{
    $pool = get_cache_pool('widgets');
    $pool->deleteItems(['widget_list']);
}
/**
 * 清除所有的小部件。
 *
 * 在这种情况下，我们只想清空池中所有的小部件。 
 * 应用中其他的池可能不会受到影响。
 */
function clear_widget_cache()
{
    $pool = get_cache_pool('widgets');
    $pool->clear();
}
/**
 * 加载小部件.
 *
 * 我们想要获取一个小部件的列表，其中一些是缓存 一些
 * 不是.这里假设从缓存中加载比在非缓存加载机制中更快 
 *  比在非缓存加载机制中更快。
 *
 * 在这种情况下， 假设窗口小部件需要经常更改因此我们仅
 * 设置缓存的时间为一小时 (3600 秒). 我们也将新缓存的
 * 对象
 * 返回到池中。
 *
 * 还需要注意在实际实现中还需要对小部件窗口进行多次
 * 加载操作，但是这与本次演示无关。
 */
function load_widgets(array $ids)
{
    $pool = get_cache_pool('widgets');
    $keys = array_map(function($id) { return 'widget.' . $id; }, $ids);
    $items = $pool->getItems($keys);

    $widgets = array();
    foreach ($items as $key => $item) {
        if ($item->isHit()) {
            $value = $item->get();
        } else {
            $value = expensive_widget_load($id);
            $item->set($value);
            $item->expiresAfter(3600);
            $pool->saveDeferred($item, true);
        }
        $widget[$value->id()] = $value;
    }
    $pool->commit(); // 如果没有延期的项目这里无操作。

    return $widgets;
}
/**
 * 这个示例反应了此规范未包含的
 * 功能，但是显示为如何通过扩展来实现
 * 添加此类功能的示例。
 */

interface TaggablePoolInterface extends Psr\Cache\CachePoolInterface
{
    /**
     * 只清除池中指定标记的项目。
     */
    clearByTag($tag);
}

interface TaggableItemInterface extends Psr\Cache\CacheItemInterface
{
    public function setTags(array $tags);
}

/**
 * 标记缓存小部件。
 */
function set_widget(TaggablePoolInterface $pool, Widget $widget)
{
    $key = 'widget.' . $widget->id();
    $item = $pool->getItem($key);

    $item->setTags($widget->tags());
    $item->set($widget);
    $pool->save($item);
}
```

### 4.2 替代方案: "弱项" 方法

许多早起的草案采用了一种更简单的「带过期时间的键值对」的方法，同时也被称之为 「弱项」 方法。在这种模型中，「缓存项」 对象只是一个不能使用的数组方法对象。用户可以直接实例化它，然后将它们扔进缓存池。虽然这种方法更为常见，但它有效的防止了缓存项的任何有意义的扩展。它有效的使缓存项的构造函数成为隐式接口的一部分，从而严重的降低了缓存项在实际灵活应用中的可扩展能力.

在 2013 年的一次调研中，大多数参与者都表现出明显的偏好，如果不大传统的 「强项」存储库方法更为健壮，那么它将被采用作为未来发展的方向.

正方:

- 更加传统的方法.

反方:

- 较差的扩展及灵活性.

### 4.3 选择: "Naked value" 方法

一些早期的缓存规范讨论建议跳过 “缓存项” 概念，而只是读取 / 写入要缓存的原始值。 尽管更简单，但需要指出的是，这使得无法分辨出缓存未命中与已缓存的原始值之间的区别。 也就是说，如果缓存查找返回 NULL，则无法判断是否没有缓存的值或 NULL 是否为已缓存的值。 (在很多情况下，NULL 是已缓存的值。)

我们审查过的最健壮的缓存实现 - 尤其是 Stash 缓存库和 Drupal 使用的本地缓存系统 - 至少在 `get` 上使用某种结构化对象，以避免混淆未命中值和标记值。 Based on that prior experience FIG decided that a naked value on `get` was impossible. 根据先前的经验，FIG 认为在 `get` 上 Naked value 是不可能的。

### 4.4 选择: ArrayAccess Pool

有人建议让 Pool 实现 ArrayAccess，这将允许缓存获取 / 设置操作使用数组语法。 由于应用有限而被拒绝，该方法的灵活性有限（使用默认控制信息进行简单的获取和设置就可以实现），如果需要，将特定实现包含为附加组件很简单。

## 5. 参与者

### 5.1 文档

- Larry Garfield

### 5.2 赞助商

- Paul Dragoonis, PPI Framework (Coordinator)
- Robert Hafner, Stash

## 6. 投票详情

[Acceptance vote on the mailing list](https://groups.google.com/forum/#!msg/php-fig/dSw5IhpKJ1g/O9wpqizWAwAJ)

## 7. 链接

*Note: Order descending chronologically.*

- [Survey of existing cache implementations](https://docs.google.com/spreadsheet/ccc?key=0Ak2JdGialLildEM2UjlOdnA4ekg3R1Bfeng5eGlZc1E#gid=0), by @dragoonis
- [Strong vs. Weak informal poll](https://docs.google.com/spreadsheet/ccc?key=0AsMrMKNHL1uGdDdVd2llN1kxczZQejZaa3JHcXA3b0E#gid=0), by @Crell
- [Implementation details informal poll](https://docs.google.com/spreadsheet/ccc?key=0AsMrMKNHL1uGdEE3SU8zclNtdTNobWxpZnFyR0llSXc#gid=1), by @Crell

## 8. 其他

### 8.1 在 expiresAt () 中处理不正确的 DateTime 值

在接口中  `CacheItemInterface::expiresAt()` 方法的 `$expiration` 参数中我们未做类型提示，但在文档块中将其指定为 `\DateTimeInterface`。 目的是允许使用  `\DateTime` 或 `\DateTimeImmutable` 对象。 但是，在 PHP 5.5 中添加了 `\DateTimeInterface` 和  `\DateTimeImmutable`，并且作者选择不在规范上强加 PHP 5.5 的严格语法要求。

尽管如此，实现者必须只接受  `\DateTimeInterface` 或兼容的类型（例如 `\DateTime` 和 `\DateTimeImmutable`），就好像该方法已做类型提示一样。 （请注意，在不同的语言版本之间，类型化参数的差异规则可能会有所不同。）

模拟失败的类型检查在 PHP 不同版本之间会有所不同，因此不建议这样做。 相反，实现者应该抛出  `\Psr\Cache\InvalidArgumentException` 的实例。

建议使用以下示例代码，以便对 expiresAt（）方法执行类型检查：

```php
class ExpiresAtInvalidParameterException implements Psr\Cache\InvalidArgumentException {}

// ...

if (! (
        null === $expiration
        || $expiration instanceof \DateTime
        || $expiration instanceof \DateTimeInterface
)) {
    throw new ExpiresAtInvalidParameterException(sprintf(
        'Argument 1 passed to %s::expiresAt() must be an instance of DateTime or DateTimeImmutable; %s given',
        get_class($this),
        is_object($expiration) ? get_class($expiration) : gettype($expiration)
    ));
}
```

