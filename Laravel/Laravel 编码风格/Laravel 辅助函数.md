## 存放位置

Laravel 提供了很多 
[辅助函数](https://laravel.com/docs/9.x/helpers),
有时候我们也需要创建自己的辅助函数。

**必须** 把所有的『自定义辅助函数』存放于 `app` 文件夹中。

并在 `composer.json` 文件中加载，如下：

1. 创建文件 app/helpers.php

```php
<?php

// 示例函数
function foo() {
    return "foo";
}
```

2. 修改项目 composer.json

在项目 composer.json 中 autoload 部分里的 files 字段加入该文件即可：

```php
{
    ...

    "autoload": {
        "files": [
            "app/helpers.php"
        ]
    }
    ...
}
```

3. 然后运行:

```php
composer dump-autoload
```

