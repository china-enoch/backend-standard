# PSR-18 HTTP 客户端

# HTTP 客户端

这个文档描述了发送 HTTP 请求和接收 HTTP 响应的共同接口。

本文件中的 「必须」，「不得」，「需要」，「应」，「不应」，「应该」，「不应该」，「推荐」，「可能」 和 「可选」 等能愿动词按照 [RFC 2119](http://tools.ietf.org/html/rfc2119) 中的描述进行解释。

## 目标

这个 PSR 的目标就是让开发者能够开发一个与 HTTP 客户端解耦的程序库。它使得程序库可重用性更高，因为它降低了依赖的数量以及降低了版本冲突的可能性。

第二个目标是 HTTP 客户端可以按照 [里氏替换原则](https://en.wikipedia.org/wiki/Liskov_substitution_principle) 进行替换。这意味着所有的客户端在发送请求时行为都是一样的。

## 定义

- 客户端 - 客户端是实现了本规范的一个程序库，它能够发送符合 PSR-7 标准的 HTTP 请求，同时能够返回一个符合 PSR-7 标准的 HTTP 响应给调用方。
- 调用方 - 调用方就是使用客户端的代码。它不用实现本规范，但是它使用了一个实现了本规范的对象（客户端）。

## 客户端

客户端是实现了客户端接口 `ClientInterface` 的对象。

客户端 **可能** 实现了以下功能：

- 从提供的 HTTP 请求中发送更改过的那个。例如，将消息体压缩后再发出去。
- 在 HTTP 响应返回到调用库之前改变它。例如，将请求的消息体解压缩。

如果客户端选择更改 HTTP 请求或 HTTP 响应，它 **必须** 确保对象保持内部一致。例如，如果客户端解压缩了消息体，那么它 **必须** 删除请求头里面的 `Content-Encoding` 并调整 `Content-Length` 。

注意，由于 [PSR-7 对象是不可变的](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-7-http-message-meta.md#why-value-objects) ，因此调用库 **不能** 假定传递给 `ClientInterface::sendRequest()` 的对象与实际发送的 PHP 对象相同。例如，异常返回的请求对象 **可能** 和传递给 `sendRequest()` 的对象不一样，因此不能通过全等号 (===) 进行比较。

客户端 **必须** 实现以下功能：

- 重新组装 HTTP 临时响应码（1xx）以便于返回 200 或者更实用的响应码给客户端。

## 错误处理

客户端一定不能将正确格式的 HTTP 请求或 HTTP 响应当做错误条件。
例如，400 和 500 范围内的响应状态代码不能触发异常，必须正常返回到调用库。

当且仅当客户端完全无法发送 HTTP 请求或者无法将 HTTP 响应解析为 PSR-7 格式响应对象时，客户端才必须抛出 `Psr\Http\Client\ClientExceptionInterface` 实例。

如果由于请求体不是正常的 HTTP 请求或缺失某些关键信息（例如主机地址或方法名称）而无法发送请求，客户端必须抛出 `Psr\Http\Client\RequestExceptionInterface` 实例。

如果由于任何类型的网络故障（包括超时）而无法发送请求，则客户端必须抛出一个 `Psr\Http\Client\NetworkExceptionInterface` 实例。

在按照上面定义去实现适当的接口前提下，客户端可能抛出比这里定义更具体的异常（例如超时异常 TimeOutException 或主机不存在异常 HostNotFoundException）。

## 接口

### 客户端接口

```php
namespace Psr\Http\Client;

use Psr\Http\Message\RequestInterface;
use Psr\Http\Message\ResponseInterface;

interface ClientInterface
{
    /**
     * 发送一个 PSR-7 标准的请求，返回一个 PSR-7 格式的响应.
     *
     * @param RequestInterface $request  
     *
     * @return ResponseInterface
     *
     * @throws \Psr\Http\Client\ClientExceptionInterface 发生错误将抛出客户端异常接口对象
     * 
     */
    public function sendRequest(RequestInterface $request): ResponseInterface;
}
```

### 客户端异常接口

```php
namespace Psr\Http\Client;

/**
 * 每个HTTP客户端相关的异常都必须实现此接口.
 */
interface ClientExceptionInterface extends \Throwable
{
}
```

### 请求异常接口

```php
namespace Psr\Http\Client;

use Psr\Http\Message\RequestInterface;

/**
 * 请求失败时的异常.
 *
 * 举例:
 *      - 请求无效 (e.g. 参数缺失)
 *      - 请求运行错误 (e.g. 响应体不可见)
 */
interface RequestExceptionInterface extends ClientExceptionInterface
{
    /**
     * 获取请求对象.
     * 
     * 请求对象可能和客户端接口发送的对象不一致.
     *
     * @return RequestInterface
     */
    public function getRequest(): RequestInterface;
}
```

### 网络异常接口

```php
namespace Psr\Http\Client;

use Psr\Http\Message\RequestInterface;

/**
 * 因网络原因导致请求无法完成时抛出该异常.
 *
 * 抛出该异常将没有响应体，因为收不到响应体时也会抛出这个异常.
 *
 * 举例：域名不能解析或连接失败.
 */
interface NetworkExceptionInterface extends ClientExceptionInterface
{
    /**
     * 返回请求对象.
     *
     * 返回的请求对象可能和客户端接口发送的对象不一致.
     *
     * @return RequestInterface
     */
    public function getRequest(): RequestInterface;
}
```

