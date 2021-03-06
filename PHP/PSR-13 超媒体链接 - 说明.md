# PSR-13 超媒体链接 - 说明文档

# 链接定义元文档

## 1. 总结

在 `HTML` 和各种 `API` 格式的上下文中，超媒体链接已经变成 `Web` 越来越重要的一部分。然而遗憾的是，没有一种通用单一的超媒体格式，也没有一种通用的方式来表示链接间的格式。

该规范旨在为 `PHP` 开发人员提供一种简单的、通用的方式来表示一个独立于所使用的序列化格式的超媒体链接。 这反过来又允许系统将超媒体链接的响应序列化为一种或多种有线格式，而不依赖于决定这些链接应该是什么的过程。

## 2. 范围

### 2.1 目标

- 本规范旨在提取和标准化不同格式之间的超媒体链接表示。

### 2.2 非目标

- 本规范不寻求去标准化或偏爱任何特定超媒体序列化的格式

## 3. 设计决策

### 为什么没有增加方法

本规范的主要目标之一是 `PSR-7` 响应对象。设计的响应对象必须是不可变的。其他 `value-object` 的实现可能也需要一个不可变的接口。

此外，一些链接提供者对象可能不是值对象，而是一个给定域中的其它对象，这个对象能够动态生成链接，可能是在数据库结果或其他底层表示之外。在这些情况下，与可写的提供者定义是不兼容的。

因此，本规范将访问器方法和可演进方法拆分为单独的接口，允许对象仅根据适合它们的用例去实现只读或可演化的版本。

### 为什么一个链接对象上的 rel 是多值？

不同的超媒体标准处理具有相同关系的多个链接。有些有一个单一的链接，此链接有多个 **rel** 的定义。其它有单一的 **rel** 条目，然后包含多个链接。

每个链接唯一地定义，但允许它有多个 **rel** 提供一个最具兼容共性的定义。一个单独的 `LinkInterface` 对象可以被序列化为一个或多个链接条目，用一个给定的合适的超媒体格式。然而，指定多个链接对象，每个链接对象有单个 **rel** 但具有相同的 `URI` 也是合法的，并且超媒体格式也可以根据需要对其进行序列化。

### 为什么需要一个 `LinkProviderInterface` 接口？

在许多上下文中，一组链接将被系到一些其它的对象上。这些对象可以在所有相关是它们的链接或者它们链接的子集的情形下使用它们。例如，可以定义代表不同 `REST` 格式的各种不同的值对象，比如：`HAL`，`JSON-LD` 或者 `Atom`。从这样的一个对象中均匀地提取这些链接以进行进一步的处理可能是有用的。例如，从一个对象中提取下一个 / 前一个链接，并将其作为一个链接头添加到 `PSR-7` **响应** 对象上。另外，许多链接用一个『预负荷』的链接关系表示是有意义的，这将指示一个 `HTTP 2` 兼容的 Web 服务器，在预期的后续请求下，应该将链接的资源流传输到客户端。

所有的这些情况都独立于有效负荷和编码对象。通过提供一个公共的接口去访问这样的链接，我们启用链接自身的通用处理，而不管生成它们的值对象或域对象。

## 4. 相关人

### 4.1 编辑者

- Larry Garfield

### 4.2 赞助者

- Matthew Weier O'Phinney (coordinator)
- Marc Alexander

### 4.3 贡献者

- Evert Pot

## 5. 投票[#](https://learnku.com/docs/psr/psr-13-links-meta/1625#d16238)

## 6. 相关链接

- [What's in a link?](http://evertpot.com/whats-in-a-link/) by Evert Pot
- [FIG Link Working Group List](https://groups.google.com/forum/#!forum/php-fig-link)

