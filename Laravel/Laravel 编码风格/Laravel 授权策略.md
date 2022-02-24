## 授权策略

**必须** 使用 [授权策略](https://laravel.com/docs/9.x/authorization#via-the-user-model) 类来做用户授权。

## 使用基类

所有 Policy 授权策略类 **必须** 继承 `app/Policies/Policy.php` 基类。基类文件如下：

```php
<?php

namespace App\Policies;

use Illuminate\Auth\Access\HandlesAuthorization;

class Policy
{
  use HandlesAuthorization;

  public function __construct()
  {
      //
  }

  public function before($user, $ability)
  {
      if ($user->isAdmin()) {
          return true;
      }
  }
}
```

## 授权策略命名

Policy 授权策略类 **必须** 遵循 **资源路由** 方式进行命名，`photos` 对应 `/app/Policies/PhotoPolicy.php` 。

## 类文件参考

Policy 授权策略类文件内容请参考以下：

```php
<?php

namespace App\Policies;

use App\Models\User;
use App\Models\Photo;

class PhotoPolicy extends Policy
{
  public function update(User $user, Photo $photo)
  {
      return $user->isAuthorOf($photo);
  }

  public function destroy(User $user, Photo $photo)
  {
      return $user->isAuthorOf($photo);
  }
}
```

## 自动判断授权策略

**应该** 使用 [自动判断授权策略方法](https://laravel.com/docs/9.x/authorization), 这样控制器和授权类的方法名就统一起来了。

```php
/**
* 更新指定的文章。
*
* @param  int  $id
* @return Response
*/
public function update($id)
{
  $post = Post::findOrFail($id);

  // 会自动调用 `PostPolicy` 类中的 `update` 方法。
  $this->authorize($post);

  // 更新文章...
}
```

