# Laravel 安全实践

## 说明

没有绝对安全，只有相对安全。Laravel 相较于其他框架在安全方面已经做得很优秀，不过作为开发者，我们要在日常开发中对『安全』需怀着敬畏之心，积极培养自己的安全意识。以下是一些 Laravel 安全相关的规范。

## 关闭 DEBUG

Laravel Debug 开启时，会暴露很多能被黑客利用的服务器信息，所以，生产环境下请 **必须** 确保：

```php
APP_DEBUG=false
```

## XSS

跨站脚本攻击（cross-site scripting，简称 XSS），具体危害体现在黑客能控制你网站页面，包括使用 JS 盗取 Cookie 等，关于 XSS 的介绍请前往 [IBM 文档库：跨站点脚本攻击深入解析](https://www.ibm.com/developerworks/cn/rational/08/0325_segal/) 。

默认情况下，在无法保证用户提交内容是 100% 安全的情况下，**必须** 使用 Blade 模板引擎的 `{{ $content }}` 语法会对用户内容进行转义。

Blade 的 `{!! $content !!}` 语法会直接对内容进行 **非转义** 输出，使用此语法时，**必须** 使用 [HTMLPurifier for Laravel 5](https://github.com/mewebstudio/Purifier) 来为用户输入内容进行过滤。使用方法参见： [使用 HTMLPurifier 来解决 Laravel 5 中的 XS...](https://learnku.com/articles/4798/the-use-of-htmlpurifier-to-solve-the-xss-xss-attacks-of-security-problems-in-laravel)

## SQL 注入

Laravel 的 [查询构造器](http://learnku.com/docs/laravel/5.3/queries) 和 [Eloquent](http://learnku.com/docs/laravel/5.3/eloquent) 是基于 PHP 的 PDO，PDO 使用 `prepared` 来准备查询语句，保障了安全性。

在使用 `raw()` 来编写复杂查询语句时，**必须** 使用数据绑定。

错误的做法：

```php
Route::get('sql-injection', function() {
    $name = "admin"; // 假设用户提交
    $password = "xx' OR 1='1"; // // 假设用户提交
    $result = DB::select(DB::raw("SELECT * FROM users WHERE name ='$name' and password = '$password'"));
    dd($result);
});
```

以下是正确的做法，利用 [select 方法](http://d.laravel-china.org/api/5.3/Illuminate/Database/ConnectionInterface.html#method_select) 的第二个参数做数据绑定：

```php
Route::get('sql-injection', function() {
    $name = "admin"; // 假设用户提交
    $password = "xx' OR 1='1"; // // 假设用户提交
    $result = DB::select(
        DB::raw("SELECT * FROM users WHERE name =:name and password = :password"),
        [
            'name' => $name,
            'password' => $password,
        ]
    );
    dd($result);
});
```

`DB` 类里的大部分执行 SQL 的函数都可传参第二个参数 `$bindings` ，详见：[API 文档](http://d.laravel-china.org/api/5.3/Illuminate/Database/ConnectionInterface.html) 。

## 批量赋值

Laravel 提供白名单和黑名单过滤（`$fillable` 和 `$guarded`），开发者 **应该** 清楚认识批量赋值安全威胁的情况下合理灵活地运用。

批量赋值安全威胁，指的是用户可更新本来不应有权限更新的字段。举例，`users` 表里的 `is_admin` 字段是用来标识用户『是否是管理员』，某不怀好意的用户，更改了『修改个人资料』的表单，增加了一个字段：

```php
<input name="is_admin" value="1" />
```

这个时候如果你更新代码如下：

```php
Auth::user()->update(Request::all());
```

此用户将获取到管理员权限。可以有很多种方法来避免这种情况出现，最简单的方法是通过设置 User 模型里的 `$guarded` 字段来避免：

```php
protected $guarded = ['id', 'is_admin'];
```

## CSRF

CSRF 跨站请求伪造是 Web 应用中最常见的安全威胁之一，具体请见 [Wiki - 跨站请求伪造](https://zh.wikipedia.org/wiki/跨站请求伪造) 或者 [Web 应用程序常见漏洞 CSRF 的入侵检测与防范](https://www.ibm.com/developerworks/cn/rational/r-cn-webcsrf/)

Laravel 默认对所有『非幂等的请求』强制使用 `VerifyCsrfToken` 中间件防护，需要开发者做的，是区分清楚什么时候该使用『非幂等的请求』。

> 幂等请求指的是：’HEAD’, ‘GET’, ‘OPTIONS’，既无论你执行多少次重复的操作都不会给资源造成变更。

- 所有删除的动作，**必须** 使用 DELETE 作为请求方法；
- 所有对数据更新的动作，**必须** 使用 POST、PUT 或者 PATCH 请求方法。

