# 日期和时间

**必须** 使用 [Carbon](https://github.com/briannesbitt/Carbon) 来处理日期和时间相关的操作。

Laravel 5.1 中文的 `diffForHumans` 可以使用 [jenssegers/date](https://github.com/jenssegers/date)。

Laravel 5.3 及以上版本的 `diffForHumans`，只需要在 `config/app.php` 文件中配置 `locale` 选项即可 ：

```php
'locale' => 'zh_CN',
```

