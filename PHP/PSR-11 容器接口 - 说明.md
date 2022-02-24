# PSR-11 容器接口 - 说明

# 容器元文档

## 1. 介绍

文档介绍了容器 PSR 出现的过程和争论。目的是解释每个决定背后的原因。

## 2. 为什么要 PSR

下面列举了 10 多个依赖注入容器，它们使用各种各样的方法来保存对象。

- 有一些基于回调函数（ Pimple，Laravel，...）
- 另一些基于不同格式（ PHP 数组，YAML 文件，XML 文件）的配置（ Symfony，ZF，...）
- 有一些可以利用工厂模式...
- 有一些使用 PHP API 来创建对象 （ PHP-DI，ZF，Symfony，Mouf... ）
- 有一些可以进行自动装载对象（ Laravel，PHP-DI，... ）
- 另一些可以基于注解来装载对象（ PHP-DI，JMS Bundle... ）
- 有一些提供图形用户界面（ Mouf... ）
- 有一些可以编译配置文件到 PHP 类中（ Symfony，ZF... ）
- 有一些可以使用对象别名...
- 有一些可以使用代理来提供依赖的延迟加载...

所以当你了解了整体情况后，你会发现有很多不同的方法来解决 DI 问题，因此也有很多不同的容器实现。然而，所有的 DI 容器都是为了解决一个相同的问题：给应用提供一种方法来查找、获取配置的对象（通常是应用需要的服务）。

通过标准化从容器中获取对象的方法，可以让使用 PSR 容器规范的框架和库可以选择使用任何与之兼容的容器类。这样就能让终端用户根据自己的喜好来选择他们自己的容器。

## 3. 容器规范的范围

### 3.1. 目的

容器 PSR 规范的目的是通过标准化框架和库通过容器获取对象的方法和参数。

区分容器下面的两个用法是很重要的：

- 配置对象实例
- 获取对象实例

通常情况下，相同地方不会同时需要这两种不同方法。通常框架使用容器来获取对象构建应用，而终端用户倾向于使用它来配置对象。

这是为什么这个接口只关注从容器获取对象的原因。

### 3.2. 不包括的目的

对象在容器中怎么保存和怎么配置不是 PSR 规范的范围。这也是不同容器可有的独特之处。一些容器根本没有配置（它依赖自动装载），一些依赖 PHP 回调来定义，另一些依赖配置文件... PSR 规范只关注怎么从容器中获取对象。

此外，对象的命名约定也不在 PSR 规范的范围内。事实上，你可以发现有下面两种命名策略：

- 实体标识符为类名或者接口名（大多数可以自动装载的框架这么用）
- 实体标识符为一个普通的名称（更接近于变量名），大多数依赖配置的框架这样使用。

两种方式都有各自的优点和缺点。PSR 规范的目的不是从中选择一个作为规范。相反，用户可以使用别名的方式在两个不同命名策略的容器间做兼容。

## 4. 推荐用法：容器 PSR 和服务定位器

PSR 指出：

> 「用户**不应该**将容器作为参数传入对象然后在对象中通过容器获得对象的依赖。这样是把容器当作服务定位器来使用，而服务定位器是不受欢迎的模式」

```php
// 这是不推荐的，容器被当作服务定位器来使用了
class BadExample
{
    public function __construct(ContainerInterface $container)
    {
        $this->db = $container->get('db');
    }
}

// 可以考虑使用直接注入的方式，替代上面的方式
class GoodExample
{
    public function __construct($db)
    {
        $this->db = $db;
    }
}
// 然后，你可以使用容器来将 $db 对象注入到 $goodExample 类中。
```

不应该在 `BadExample` 类注入容器的原因：

- 这样减少了代码的兼容性：通过注入容器，你不得不使用兼容 PSR 规范的容器。而通过直接注入方式，你的代码可以使用**任何**容器。
- 这样将强制使开发者使用「db」作为数据库的实体标识符。这个命名可能与其他包（使用 「db」 来获取其他服务）产生冲突。
- 这样将使测试变得困难。
- 这样在代码中不能明显看出 `BadExample` 类依赖 「db」服务。依赖关系被隐藏了。

通常， `ContainerInterface` 接口是被其他包使用。而作为使用框架的 PHP 开发者，不太可能需要直接使用 `ContainerInterface` 的接口和类型提示。

判断你的代码是否合理的使用了容器，归结于知道在容器中查找的对象是否为当前对象的依赖。下面是几个实例：

```php
class RouterExample
{
    // ...

    public function __construct(ContainerInterface $container)
    {
        $this->container = $container;
    }

    public function getRoute($request)
    {
        $controllerName = $this->getContainerEntry($request->getUrl());
        // 这是正确的，路由通过容器查找对应的控制器对象，
        // 而路由不依赖控制器
        $controller = $this->container->get($controllerName);
        // ...
    }
}
```

在这个示例中，路由将 URL 转换为控制器类名，然后从容器中获得控制器对象。但路由并不真正的依赖控制器。大致的原则是，如果对象需要*计算*并从一系列的对象列表中得到对应的对象，你的使用通常是合理的。

有一个例外，作为只是单纯创建和返回对象实例的工厂类是可以使用服务定位器的。工厂类必须实现一个接口，以至于它可以被实现相同接口的其它工厂类替换。

```php
// 这是合理的：一个创建对象的工厂接口和它的实现
interface FactoryInterface
{
    public function newInstance();
}

class ExampleFactory implements FactoryInterface
{
    protected $container;

    public function __construct(ContainerInterface $container)
    {
        $this->container = $container;
    }

    public function newInstance()
    {
        return new Example($this->container->get('db'));
    }
}
```

## 5. 历史

在提交容器 PSR 到 PHP-FIG 组织之前，`ContainerInterface` 接口是在名为 [container-interop](https://github.com/container-interop/container-interop/) 的项目中首先提出的。这个项目的目的是为实施 `ContainerInterface` 接口提供实验平台，并为容器 PSR 铺路。

在接下来的文档中，你会看到频繁的引用 `container-interop` 。

## 6. 接口名称

接口名称与 `container-interop` 中讨论的一致（只是为了符合其他 PSRs 而改变了命名空间）。接口名称是在 `container-interop` [[4\]](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container-meta.md#link_naming_discussion) 中彻底讨论，并投票决定的 [[5\]](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container-meta.md#link_naming_vote) 。

下面是他们投票选项的结果：

- `ContainerInterface`: +8
- `ProviderInterface`: +2
- `LocatorInterface`: 0
- `ReadableContainerInterface`: -5
- `ServiceLocatorInterface`: -6
- `ObjectFactory`: -6
- `ObjectStore`: -8
- `ConsumerInterface`: -9

## 7. 接口方法

接口需要包含那些方法是通过对现有的容器进行统计分析后得到的 [[6\]](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container-meta.md#link_statistical_analysis) 。

统计分析的概要如下：

- 所用的容器都提供了通过 id 获取对应对象的方法
- 大多数使用的方法名称是 `get()`
- 所有的容器中，`get()` 方法都有一个必须的字符串参数
- 一些容器的 `get()` 方法有一个其他可选的参数，但是不同容器的可选参数作用不一样
- 大多数容器都提供了一个测试是否可以通过 id 获取到对应对象的方法
- 大多数使用的方法名称为 `has()`
- 对于所有提供了 `has()` 方法的容器，它们都有一个字符串参数
- 大多数容器在 `get()` 方法没有查到对象时抛出异常，而不是返回 null
- 大多数容器都没有实现 `ArrayAccess` 接口

容器中是否需要提供方法来定义对象，在 container-interop 项目开始时已经被讨论过了 [[4\]](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container-meta.md#link_naming_discussion) 。讨论的结果是接口不需要提供这个方法，因为它不在容器接口的目的中（查看「目的」部分）。

结果，`ContainerInterface` 接口提供两个方法：

- `get()` 方法，有一个必须的字符串参数，可返回任何对象。如果没有找到参数对应的对象时抛出异常。
- `has()` 方法，有一个必须的字符串参数，返回布尔值。

### 7.1. `get()` 方法的参数个数

`ContainerInterface` 接口的 `get()` 方法只定义了一个必须的参数，这与当前有其他可选参数的容器不兼容。但 PHP 允许实现类拥有更多的参数，只要参数是可选的，因为实现类这样是符合接口的要求的。

PSR 容器与 container-interop 规范规定的不同， [ container-interop 规范](https://github.com/container-interop/container-interop/blob/master/docs/ContainerInterface.md) 指出：

> 尽管 `ContainerInterface` 接口的 `get()` 方法只定义了一个必须的参数，但它的实现类可以接受其他的可选参数。

但这个语句在 PSR-11 中被删除了，因为：

- 接受更多可选参数违背 PHP 面向对象原则，这和 PSR-11 没有直接关系
- 我们不鼓励接口的实现类添加额外的可选参数，因为我们推荐面向接口编程而不是面向实现编程

然而，一些已经实现的有其他可选参数的容器；这在技术上是合法的。这些容器也与 PSR-11 兼容 [[11\]](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container-meta.md#link_get_optional_parameters) 。

### 7.2. 参数 `$id` 的类型

在 container-interop 项目中已经讨论了 `get()` 和 `has()` 方法中 `$id` 参数的类型。

尽管所有分析的容器中 `$id` 参数都是 `string` 类型，但是建议允许它可以是任何类型（比如对象），这样将允许容器提供更多高级的查询 API。

例如使用容器来作为对象构造器，`$id` 参数是对象就可以告诉容器怎么去创建一个对象实例。

讨论的结果 [[7\]](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container-meta.md#link_method_and_parameters_details) 是这超出了 `$id` 是用来从容器获取对象的范围， `$id` 是不知道对象是怎么创建的。对象参数更适合工厂类。

### 7.3. 抛出异常

PSR 提供了 2 个用来被容器异常实现的接口。

#### 7.3.1 异常基类

`Psr\Container\ContainerExceptionInterface` 接口是异常基类。从容器中抛出的自定义异常都应该实现这个接口。

任何属于容器部分的异常都应该实现 `ContainerExceptionInterface` 接口，下面是几个例子：

- 如果容器依赖配置文件，而配置文件又存在缺陷时，容器可能会抛出一个实现 `ContainerExceptionInterface` 接口的  `InvalidFileException` 异常。
- 如果依赖关系中检测到存在循环依赖，容器可能会抛出一个实现 `ContainerExceptionInterface` 接口的 `CyclicDependencyException` 异常。

然而，如果抛出异常的代码在容器范围外（例如，初始化对象时抛出异常），这时容器抛出的自定义异常不要求实现 `ContainerExceptionInterface` 基类接口。

异常基类接口的作用被质疑：它不是通用的会被捕获的异常 [[8\]](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container-meta.md#link_base_exception_usefulness) 。

然而，大多数 PHP-FIG 成员认为异常基类是一个最佳实践。原有的 PSR 容器和几个成员的项目都已经实现了异常基类。因此异常基类被保留下来了。

#### 7.3.2 未找到异常 Not found exception

参数 id 对应的对象在容器中不存在时，  `get` 方法抛出的异常**必须**实现 `Psr\Container\NotFoundExceptionInterface` 接口。

对于给定的标识符：

- 如果 `has` 方法返回 `false` ， `get` 方法抛出的异常一定要实现 `Psr\Container\NotFoundExceptionInterface` 接口。
- 如果 `has` 方法返回 `true`，这并不意味 `get` 会成功且不会抛出异常。如果对象依赖的对象不存在时也会抛出 `Psr\Container\NotFoundExceptionInterface` 接口的异常。

因此，如果用户捕获到了实现 `Psr\Container\NotFoundExceptionInterface` 接口的异常，可能意味着两种情况 [[9\]](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container-meta.md#link_not_found_behaviour) ：

- 请求的对象不存在（错误的请求）
- 或者请求对象的依赖不存在（比如容器的配置错误）

用户可以通过 `has` 方法轻松地区分上面两种情况。

伪代码如下：

```php
if (!$container->has($id)) {
    // 请求的对象不存在
    return;
}
try {
    $entry = $container->get($id);
} catch (NotFoundExceptionInterface $e) {
    // 因为请求的对象存在，所以 NotFoundExceptionInterface 的异常表示这是容器配置错误或者请求对象的依赖不存在。
}
```

1. 已有的容器实现

------

在写这篇文字的时候，下列项目已经实现或者使用了  `container-interop` 版本的接口。

### 容器的实现

- [Acclimate](https://github.com/jeremeamia/acclimate-container)
- [Aura.DI](https://github.com/auraphp/Aura.Di)
- [dcp-di](https://github.com/estelsmith/dcp-di)
- [League Container](https://github.com/thephpleague/container)
- [Mouf](http://mouf-php.com/)
- [Njasm Container](https://github.com/njasm/container)
- [PHP-DI](http://php-di.org/)
- [PimpleInterop](https://github.com/moufmouf/pimple-interop)
- [XStatic](https://github.com/jeremeamia/xstatic)
- [Zend ServiceManager](https://github.com/zendframework/zend-servicemanager)

### 中间件

- [Alias-Container](https://github.com/thecodingmachine/alias-container)
- [Prefixer-Container](https://github.com/thecodingmachine/prefixer-container)

### 容器使用者

- [Behat](https://github.com/Behat/Behat)
- [interop.silex.di](https://github.com/thecodingmachine/interop.silex.di)
- [mindplay/middleman](https://github.com/mindplay-dk/middleman)
- [PHP-DI Invoker](https://github.com/PHP-DI/Invoker)
- [Prophiler](https://github.com/fabfuel/prophiler)
- [Silly](https://github.com/mnapoli/silly)
- [Slim](https://github.com/slimphp/Slim)
- [Splash](http://mouf-php.com/packages/mouf/mvc.splash-common/version/8.0-dev/README.md)
- [Zend Expressive](https://github.com/zendframework/zend-expressive)

这个列表不包含所有的容器实现和使用者，这里仅仅是一些对 PSR 有着巨大兴趣的项目例子。

## 8. 相关链接

1. [容器 PSR 和服务定位器的讨论](https://groups.google.com/forum/#!topic/php-fig/pyTXRvLGpsw)
2. [Container-interop 项目的  `ContainerI...`](https://github.com/container-interop/container-interop/blob/master/src/Interop/Container/ContainerInterface.php)
3. [所有的 issues](https://github.com/container-interop/container-interop/issues?labels=ContainerInterface&milestone=&page=1&state=closed)
4. [接口名称和 container-interop 项目范围的讨...](https://github.com/container-interop/container-interop/issues/1)
5. [接口名称的投票](https://github.com/container-interop/container-interop/wiki/%231-interface-name:-Vote)
6. [已有容器方法名称的统计分析](https://gist.github.com/mnapoli/6159681)
7. [方法名称和参数的讨论](https://github.com/container-interop/container-interop/issues/6)
8. [异常基类作用的讨论](https://groups.google.com/forum/#!topic/php-fig/_vdn5nLuPBI)
9. [`NotFoundExceptionInterface`...](https://groups.google.com/forum/#!topic/php-fig/I1a2Xzv9wN8)
10. 在 [container-interop](https://github.com/container-interop/container-interop/issues/6) 项目和 在 [PHP-FIG mailing list](https://groups.google.com/forum/#!topic/php-fig/zY6FAG4-oz8) 对 get 方法可选参数的讨论

