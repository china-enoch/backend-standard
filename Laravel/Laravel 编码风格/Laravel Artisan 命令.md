# Artisan 命令行

所有的自定义命令，都 **必须** 有项目的命名空间。

如：

```php
php artisan phphub:clear-token
php artisan phphub:send-status-email
...
```

错误的例子为：

```php
php artisan clear-token
php artisan send-status-email
...
```

