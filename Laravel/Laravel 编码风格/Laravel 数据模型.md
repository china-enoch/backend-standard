# 模型

## 放置位置

所有的数据模型文件，都 **必须** 存放在：`app/Models/` 文件夹中。

命名空间：

```php
namespace App\Models;
```

## User.php

Laravel 5.1 默认安装会把 `User` 模型存放在 `app/User.php`，**必须** 移动到 `app/Models` 文件夹中，并修改命名空间声明为 `App\Models`，同上。

为了不破坏原有的逻辑点，**必须** 全局搜索 `App\User` 并替换为 `App\Models\User`。

## 使用基类

所有的 **Eloquent 数据模型** 都 **必须** 继承统一的基类 `App\Models\Model`，此基类存放位置为 `/app/Models/Model.php`，内容参考以下：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model as EloquentModel;

class Model extends EloquentModel
{
    public function scopeRecent($query)
    {
        return $query->orderBy('created_at', 'desc');
    }
}
```

以 Photo 数据模型作为例子继承 Model 基类：

```php
<?php

namespace App\Models;

class Photo extends Model
{
    protected $fillable = ['id', 'user_id'];

    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```

## 命名规范

数据模型相关的命名规范：

- 数据模型类名 `必须` 为「单数」, 如：`App\Models\Photo`
- 类文件名 `必须` 为「单数」，如：`app/Models/Photo.php`
- 数据库表名字 `必须` 为「复数」，多个单词情况下使用「[Snake Case](https://en.wikipedia.org/wiki/Snake_case) 」 如：`photos`, `my_photos`
- 数据库表迁移名字 `必须` 为「复数」，如：`2014_08_08_234417_create_photos_table.php`
- 数据填充文件名 `必须` 为「复数」，如：`PhotosTableSeeder.php`
- 数据库字段名 `必须` 为「[Snake Case](https://en.wikipedia.org/wiki/Snake_case) 」，如：`view_count`, `is_vip`
- 数据库表主键 `必须` 为「id」
- 数据库表外键 `必须` 为「resource_id」，如：`user_id`, `post_id`
- 数据模型变量 `必须` 为「resource_id」，如：`$user_id`, `$post_id`

## 利用 Trait 来扩展数据模型

有时候数据模型里的代码会变得很臃肿，**应该** 利用 Trait 来精简逻辑代码量，提高可读性，类似于 [Ruby China 源码](https://github.com/ruby-china/ruby-china/blob/master/app/models/topic.rb#L11-L17)

> 借鉴于 Rails 的设计理念：「Fat Models, Skinny Controllers」。

所有模型相关的 Traits 必须存放于 `app/Models/Traits` 目录下。

## Repository

**绝不** 使用 Repository，因为我们不是在写 JAVA 代码，太多封装就成了「过度设计（Over Designed）」，极大降低了编码愉悦感，使用 MVC 够傻够简单。

代码的可读性，维护和开发的便捷性，直接关系到程序员开发时的愉悦感，直接影响到项目推进效率和程序 Debug 的速度。

## 关于 SQL 文件

- **绝不** 使用命令行或者 PHPMyAdmin 直接创建索引或表。**必须** 使用 [数据库迁移](https://laravel.com/docs/9.x/migrations) 去创建表结构，并提交版本控制器中；
- **绝不** 为了共享对数据库更改就直接导出 SQL，所有修改都 **必须** 使用 [数据库迁移](https://laravel.com/docs/9.x/migrations) ，并提交版本控制器中；
- **绝不** 直接向数据库手动写入伪造的测试数据。**必须** 使用 [数据填充](https://laravel.com/docs/9.x/seeding) 来插入假数据，并提交版本控制器中。

## 全局作用域

Laravel 的 [Model 全局作用域](https://laravel.com/docs/9.x/eloquent#global-scopes) 允许我们为给定模型的所有查询添加默认的条件约束。

所有的全局作用域都 **必须** 统一使用 `闭包定义全局作用域`，如下：

```php
/**
 * 数据模型的启动方法
 *
 * @return void
 */
protected static function boot()
{
    parent::boot();

    static::addGlobalScope('age', function(Builder $builder) {
        $builder->where('age', '>', 200);
    });
}
```

## 数据层无状态

先看一段代码，以下是 Post 模型里创建文章评论的方法：

```php
    public function createComment($content)
    {
        return $this->comments()->create([
            'content' => $content,
            'user_id' => Auth::user()->id
        ]);
    }
```

注意 `Auth::user()->id` ，在数据层里使用当前登录用户状态，是默认假设这段代码永远是在 Web 用户请求下执行的。

然而事实并非如此，有时候你可能会在命令行下触发调用这个 `createComment()` 方法，有时候是管理员在后台触发，有时候是队列里触发。

一个最佳实践的做法是， **绝不** 在数据层里使用用户登录状态信息。如果需要用户信息，**必须** 将其作为依赖进行传参，如以上代码可修改为：

```php
    public function createComment($content, $user)
    {
        return $this->comments()->create([
            'content' => $content,
            'user_id' => $user->id
        ]);
    }
```

在有需要的地方调用时，以参数传入：

```php
Post::createComment($content, Auth::user())
```

命令行书写某些特殊逻辑时，例如使用 1 号用户的身份创建评论：

```php
Post::createComment($content, User::find(1))
```

数据层，也就是模型里，不能写