# PSR-17 HTTP 工厂

# HTTP 工厂

这个文档描述了创建符合 [PSR-7](https://www.php-fig.org/psr/psr-7/) 规范的 HTTP 对象的工厂通用标准。

PSR-7 没有包含有关如何创建 HTTP 对象的建议，这导致需要在与 PSR-7 的特定实现无关的组件内创建新 HTTP 对象时会遇到困难。

本文档中概述的接口描述了可以实例化 PSR-7 对象的方法。

本文件中的 `必须`，`不得`，`需要`，`应`，`不应`，`应该`，`不应该`，`推荐`，`可能` 和 `可选` 等能愿动词按照 [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt) 中的描述进行解释。

\1. 详细描述

HTTP 工厂是可以创建由 PSR-7 定义的 HTTP 对象的方法。HTTP 工厂 **必须** 实现包中提供的所有对象类型。

\2. 接口
下面的接口 **可能** 在一个类中实现，也可以在分开的多个类中实现。

### 2.1 RequestFactoryInterface

用来创建客户端请求。

```php
namespace Psr\Http\Message;

use Psr\Http\Message\RequestInterface;
use Psr\Http\Message\UriInterface;

interface RequestFactoryInterface
{
    /**
     * 创建一个新的请求
     *
     * @param string $method 请求使用的 HTTP 方法。
     * @param UriInterface|string $uri 请求关联的 URI。
     */
    public function createRequest(string $method, $uri): RequestInterface;
}
```

### 2.2 ResponseFactoryInterface

用来创建响应对象。

```php
namespace Psr\Http\Message;

use Psr\Http\Message\ResponseInterface;

interface ResponseFactoryInterface
{
    /**
     * 创建一个响应对象。
     *
     * @param int $code HTTP 状态码，默认值为 200。
     * @param string $reasonPhrase 与状态码关联的原因短语。如果未提供，实现 **可能** 使用 HTTP 规范中建议的值。
     */
    public function createResponse(int $code = 200, string $reasonPhrase = ''): ResponseInterface;
}
```

### 2.3 ServerRequestFactoryInterface

用来创建服务端请求。

```php
namespace Psr\Http\Message;

use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\UriInterface;

interface ServerRequestFactoryInterface
{
    /**
     * 创建一个服务端请求。
     *
     * 注意服务器参数要精确的按给定的方式获取 - 不执行给定值的解析或处理。
     * 尤其是不要从中尝试获取 HTTP 方法或 URI，这两个信息一定要通过函数参数明确给出。
     *
     * @param string $method 与请求关联的 HTTP 方法。
     * @param UriInterface|string $uri 与请求关联的 URI。
     * @param array $serverParams 用来生成请求实例的 SAPI 参数。
     */
    public function createServerRequest(string $method, $uri, array $serverParams = []): ServerRequestInterface;
}
```

### 2.4 StreamFactoryInterface

为请求和响应创建流。

```php
namespace Psr\Http\Message;

use Psr\Http\Message\StreamInterface;

interface StreamFactoryInterface
{
    /**
     * 从字符串创建一个流。
     *
     * 流 **应该** 使用临时资源来创建。
     *
     * @param string $content 用于填充流的字符串内容。
     */
    public function createStream(string $content = ''): StreamInterface;

    /**
     * 通过现有文件创建一个流。
     *
     * 文件 **必须** 用给定的模式打开文件，该模式可以是 `fopen` 函数支持的任意模式。
     *
     * `$filename` **可能** 是任意被 `fopen()` 函数支持的字符串。
     *
     * @param string $filename 用作流基础的文件名或 URI。
     * @param string $mode 用于打开基础文件名或流的模式。
     *
     * @throws \RuntimeException 如果文件无法被打开时抛出。
     * @throws \InvalidArgumentException 如果模式无效会被抛出。
     */
    public function createStreamFromFile(string $filename, string $mode = 'r'): StreamInterface;

    /**
     * 通过现有资源创建一个流。
     *
     * 流必须是可读的并且可能是可写的。
     *
     * @param resource $resource 用作流的基础的 PHP 资源。
     */
    public function createStreamFromResource($resource): StreamInterface;
}
```

在从字符串创建流时接口的实现 **应该** 使用临时资源。这个方法的 **推荐** 实现是：

```php
$resource = fopen('php://temp', 'r+');
```

### 2.5 UploadedFileFactoryInterface

用来创建上传文件创建流。

```php
namespace Psr\Http\Message;

use Psr\Http\Message\StreamInterface;
use Psr\Http\Message\UploadedFileInterface;

interface UploadedFileFactoryInterface
{
    /**
     * 创建一个上传文件接口的对象。
     *
     * 如果未提供大小，将通过检查流的大小来确定。
     *
     * @link http://php.net/manual/features.file-upload.post-method.php
     * @link http://php.net/manual/features.file-upload.errors.php
     *
     * @param StreamInterface $stream 表示上传文件内容的流。
     * @param int $size 文件的大小，以字节为单位。
     * @param int $error PHP 上传文件的错误码。
     * @param string $clientFilename 如果存在，客户端提供的文件名。
     * @param string $clientMediaType 如果存在，客户端提供的媒体类型。
     *
     * @throws \InvalidArgumentException 如果文件资源不可读时抛出异常。
     */
    public function createUploadedFile(
        StreamInterface $stream,
        int $size = null,
        int $error = \UPLOAD_ERR_OK,
        string $clientFilename = null,
        string $clientMediaType = null
    ): UploadedFileInterface;
}
```

### 2.6 UriFactoryInterface

为客户端和服务器请求创建 URI。

```php
namespace Psr\Http\Message;

use Psr\Http\Message\UriInterface;

interface UriFactoryInterface
{
    /**
     * 创建一个 URI。
     *
     * @param string $uri 要解析的 URI。
     *
     * @throws \InvalidArgumentException 如果给定的 URI 无法被解析时抛出。
     */
    public function createUri(string $uri = '') : UriInterface;
}
```

