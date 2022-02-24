# Laravel 程序优化
## 说明

作为优秀的开发者，在日常编码中，应积极培养书写高执行效率代码的意识。不过项目运行效率是一个系统性工程，不应该只停留在代码层面上，有时更应该考虑整个项目架构，包括项目中使用的软件等。

本文罗列了一些常见的优化项目，并且对其做了约束。

## 1. 配置信息缓存

生产环境中的 **应该** 使用『配置信息缓存』来加速 Laravel 配置信息的读取。

使用以下 Artisan 自带命令，把 `config` 文件夹里所有配置信息合并到一个文件里，减少运行时文件的载入数量：

```php
php artisan config:cache
```

缓存文件存放在 `bootstrap/cache/` 文件夹中。

可以使用以下命令来取消配置信息缓存：

```php
php artisan config:clear
```

注意：配置信息缓存不会随着更新而自动重载，所以，开发时候建议关闭配置信息缓存，一般在生产环境中使用。可以配合 [Envoy 任务运行器](https://laravel.com/docs/9.x/envoy) 使用，在每次上线代码时执行 `config:clear` 命令。

## 2. 路由缓存

生产环境中的 **应该** 使用『路由缓存』来加速 Laravel 的路由注册。

路由缓存可以有效的提高路由器的注册效率，在大型应用程序中效果越加明显，可以使用以下命令：

```php
php artisan route:cache
```

缓存文件存放在 `bootstrap/cache/` 文件夹中。另外，路由缓存不支持路由匿名函数编写逻辑，详见：[文档 - 路由缓存](https://laravel.com/docs/9.x/routing#route-caching)

可以使用下面命令清除路由缓存：

```php
php artisan route:clear
```

注意：路由缓存不会随着更新而自动重载，所以，开发时候建议关闭路由缓存，一般在生产环境中使用。可以配合 [Envoy 任务运行器](https://laravel.com/docs/9.x/envoy) 使用，在每次上线代码时执行 `route:clear` 命令。

## 3. 类映射加载优化

`optimize` 命令把常用加载的类合并到一个文件里，通过减少文件的加载，来提高运行效率。生产环境中的 **应该** 使用 optimize 命令来优化类的加载速度：

```php
php artisan optimize --force
```

以上命令会在 `bootstrap/cache/` 文件夹中生成缓存文件。你可以通过修改 `config/compile.php` 文件来添加要合并的类。在 `production` 环境中，参数 `--force` 不需要指定，文件就会自动生成。

要清除类映射加载优化，请运行以下命令：

```php
php artisan clear-compiled
```

此命令会删除上面 `optimize` 生成的两个文件。

注意：此命令要运行在 `php artisan config:cache` 后，因为 `optimize` 命令是根据配置信息（如：`config/app.php` 文件的 `providers` 数组）来生成文件的。

## 4. 自动加载优化

此命令不止针对于 Laravel 程序，适用于所有使用 `composer` 来构建的程序。此命令会把 `PSR-0` 和 `PSR-4` 转换为一个类映射表，来提高类的加载速度。

```php
composer dumpautoload -o
```

> 注意：`php artisan optimize --force` 命令里已经做了这个操作。

## 5. 使用 Memcached 来存储会话

每一个 Laravel 的请求，都会产生会话，修改会话的存储方式能有效提高程序效率。会话的配置文件是 `config/session.php`。生产环境中的 **必须** 使用 Memcached 或者 Redis 等专业的缓存软件来存储会话，**应该** 优先选择 Memcached：

```php
'driver' => 'memcached',
```

## 6. 使用专业缓存驱动器

「缓存」是提高应用程序运行效率的法宝之一，Laravel 默认缓存驱动是 `file` 文件缓存，生产环境中的 **必须** 使用专业的缓存系统，如 Redis 或者 Memcached。**应该** 优先考虑 Redis。**应该** 避免使用数据库缓存。

```php
'default' => 'redis',
```

## 7. 数据库请求优化

关联模型数据读取时 **必须** 使用 [延迟预加载](https://laravel.com/docs/9.x/eloquent-relationships#lazy-eager-loading) 和 [预加载](https://laravel.com/docs/9.x/eloquent-relationships#eager-loading) 。

临近上线时 **必须** 使用 [Laravel Debugbar](https://github.com/barryvdh/laravel-debugbar) 或者 [Clockwork](https://learnku.com/laravel/t/23) 留意每一个页面的总 SQL 请求条数，进行数据库请求调优。

## 8. 为数据集书写缓存逻辑

**应该** 合理的使用 Laravel 提供的缓存层操作，把从数据库里面拿出来的数据集合进行缓存，减少数据库的压力，运行在内存上的专业缓存软件对数据的读取也远远快于数据库。

```php
$hot_posts = Cache::remember('posts.hot_posts', $minutes = 30, function()
{
    return Post::getHotPosts();
});
```

`remember` 甚至连数据关联模型也都一并缓存了，多么方便呀。

## 9. 使用即时编译器

**可以** 使用 OpCache 进行优化。OpCache 都能轻轻松松的让你的应用程序在不用做任何修改的情况下，直接提高 50% 或者更高的性能。

## 10. 前端资源合并

作为优化的标准：

- 一个页面 **应该** 只加载一个 CSS 文件；
- 一个页面 **应该** 只加载一个 JS 文件。

另外，为了文件要能方便走 CDN，需要文件名 **应该** 随着修改而变化。

