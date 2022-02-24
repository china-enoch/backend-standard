# 视图

## 统一布局

相似的页面下，**必须** 使用 `layouts` 文件来统一页面头部与尾部。

## 优先使用 Blade

视图文件 **必须** 优先考虑使用 `.blade.php` 后缀来指定使用 Blade 模板引擎。

## 保持目录清晰

- layouts - 页面布局文件 **必须** 放置于此目录下；
- common - 存放页面通用元素；
- pages - 简单的页面存放文件夹，如：about、contact 等；
- resources - 对应 Restful 路由的资源路径名称，以 URI `photos/create` 为例，对应 `create.blade.php` 文件，存放在文件夹 `photos` 下。

**必须** 避免在 `resources/views` 目录下直接放置视图文件。

## 局部视图

局部视图文件 **必须** 使用 `_` 前缀来命名，如：`photos/_upload_form.blade.php`

## 视图命名要释义

为了和 Restful 路由器和资源控制器保持一致，视图命名也 **必须** 使用资源视图的命名方式。以 `photos` 为例：

```
photos/index.blade.php
```

- 内容列表视图
- 对应路由器 `/photos`，命名 `photos.index`
- 控制器方法 `PhotosController@index`

```
photos/show.blade.php
```

- 单个内容视图
- 对应路由器 `/photos/{id}`，命名 `photos.show`
- 控制器方法 `PhotosController@show`

```
photos/create.blade.php
```

- 内容创建视图
- 对应路由器 `/photos/create`，命名 `photos.create`
- 控制器方法 `PhotosController@create`

```
photos/edit.blade.php
```

- 内容编辑的视图
- 对应路由器 `/photos/edit`，命名 `photos.edit`
- 控制器方法 `PhotosController@edit`

## `create_and_edit` 视图

很多情况下，创建和编辑视图里的页面结构接近相似，在这种情况下，**应该** 使用 `create_and_edit` 视图。以 `photos` 为例：

- `PhotosController@create` 对应视图：`/photos/create_and_edit.blade.php`
- `PhotosController@edit` 对应 视图：`/photos/create_and_edit.blade.php`

这样一来，通常情况下，一个完整的 `photos` 资源对应的视图文件为以下：


```php

├── photos
│   ├── create_and_edit.blade.php
│   ├── index.blade.php
│   └── show.blade.php
```

