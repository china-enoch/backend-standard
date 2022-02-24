# 数据填充
## factory 辅助函数

`必须` 使用 `factory` 方法来做数据填充，因为是框架提倡的，并且可以同时为测试代码服务。

## 运行效率

开发数据填充时，`必须` 特别注意 `php artisan db:seed` 的运行效率，否则随着项目的代码量越来越大，`db:seed` 的运行时间会变得越来越长，有些项目多达几分钟甚至几十分钟。

原则是：

> Keep it lighting speed.

只有当 `db:seed` 运行起来很快的时候，才能完全利用数据填充工具带来的便利，而不是累赘。

## 批量入库

所有假数据入库操作，都 **必须** 是批量操作，配合 `factory` 使用以下方法：

```php
$users = factory(User::class)->times(1000)->make();
User::insert($users->toArray());
```

以上只执行一条数据库语句，推荐阅读 [大批量假数据填充的正确方法](https://learnku.com/laravel/t/2066/the-correct-method-for-filling-large-quantity-of-false-data)