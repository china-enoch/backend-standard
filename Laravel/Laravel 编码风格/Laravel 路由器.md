# 路由器
## 路由闭包

**绝不** 在路由配置文件里书写『闭包路由』或者其他业务逻辑代码，因为一旦使用将无法使用 [路由缓存](https://laravel.com/docs/9.x/routing#route-caching) 。

路由器要保持干净整洁，**绝不** 放置除路由配置以外的其他程序逻辑。

## Restful 路由

**必须** 优先使用 Restful 路由，配合资源控制器使用


[![file](https://cdn.learnku.com/uploads/images/201705/19/1/09GHC72ygP.png)](https://cdn.learnku.com/uploads/images/201705/19/1/09GHC72ygP.png)



超出 Restful 路由的，**应该** 模仿上图的方式来定义路由。

## resource 方法正确使用

一般资源路由定义：

```php
Route::resource('photos', 'PhotosController');
```

等于以下路由定义：

```php
Route::get('/photos', 'PhotosController@index')->name('photos.index');
Route::get('/photos/create', 'PhotosController@create')->name('photos.create');
Route::post('/photos', 'PhotosController@store')->name('photos.store');
Route::get('/photos/{photo}', 'PhotosController@show')->name('photos.show');
Route::get('/photos/{photo}/edit', 'PhotosController@edit')->name('photos.edit');
Route::put('/photos/{photo}', 'PhotosController@update')->name('photos.update');
Route::delete('/photos/{photo}', 'PhotosController@destroy')->name('photos.destroy');
```

使用 `resource` 方法时，如果仅使用到部分路由，**必须** 使用 `only` 列出所有可用路由：

```php
Route::resource('photos', 'PhotosController', ['only' => ['index', 'show']]);
```

**绝不** 使用 `except`，因为 `only` 相当于白名单，相对于 `except` 更加直观。路由使用白名单有利于养成『安全习惯』。

## 单数 or 复数？

资源路由路由 URI **必须** 使用复数形式，如：

- `/photos/create`
- `/photos/{photo}`

错误的例子如：

- `/photo/create`
- `/photo/{photo}`

## 路由模型绑定

在允许使用路由 [模型绑定](https://laravel.com/docs/9.x/routing#route-model-binding) 的地方 **必须** 使用。

模型绑定代码 **必须** 放置于 `app/Providers/RouteServiceProvider.php` 文件的 `boot` 方法中：

```php
    public function boot()
    {
        Route::bind('user_name', function ($value) {
            return User::where('name', $value)->first();
        });

        Route::bind('photo', function ($value) {
            return Photo::find($value);
        });

        parent::boot();
    }
```

## 全局路由器参数

出于安全考虑，**应该** 使用全局路由器参数限制，详见 [文档](https://laravel.com/docs/9.x/routing#basic-routing)

**必须** 在 `RouteServiceProvider` 文件的 boot 方法里定义模式：

```php
/**
 * 定义你的路由模型绑定，模式过滤器等。
 *
 * @param  \Illuminate\Routing\Router  $router
 * @return void
 */
public function boot(Router $router)
{
    $router->pattern('id', '[0-9]+');

    parent::boot();
}
```

模式一旦被定义，便会自动应用到所有使用该参数名称的路由上：

```php
Route::get('users/{id}', 'UsersController@show');
Route::get('photos/{id}', 'PhotosController@show');
```

只有在 `id` 为数字时，才会路由到控制器方法中，否则 404 错误。

## 路由命名

除了 `resource` 资源路由以外，其他所有路由都 **必须** 使用 `name` 方法进行命名。

**必须** 使用『资源前缀』作为命名规范，如下的 `users.follow`，资源前缀的值是 `users.`：

```php
Route::post('users/{id}/follow', 'UsersController@follow')->name('users.follow');
```

## 获取 URL

获取 URL **必须** 遵循以下优先级：

1. `$model->link()`
2. `route` 方法
3. `url` 方法

在 Model 中创建 `link()` 方法：

```php
public function link($params = [])
{
    $params = array_merge([$this->id], $params);
    return route('models.show', $params);
}
```

所有单个模型数据链接使用：

```php
$model->link();

// 或者添加参数
$model->link($params = ['source' => 'list'])
```

『单个模型 URI』经常会发生变化，这样做将会让程序更加灵活。

除了『单个模型 URI』，其他路由 **必须** 使用 `route` 来获取 URL：

```php
$url = route('profile', ['id' => 1]);
```

无法使用 `route` 的情况下，**可以** 使用 `url` 方法来获取 URL：

```php
url('profile', [1]);
```