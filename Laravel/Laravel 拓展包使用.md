## 说明

我们都知道 Laravel 扩展包的注册会对应用造成消耗。有一些扩展包是开发环境中专用，生产环境中并不会使用到，为了避免无用的负载， 必须严格控制其安装和加载。

## 安装

安装开发专用扩展包时 **必须** 使用 `--dev` 参数，如：

```php
composer require laracasts/generators --dev
```

## 加载

开发专用的 provider `绝不`在 `config/app.php` 里面注册，`必须` 在 `app/Providers/AppServiceProvider.php` 文件中使用如以下方式：

```php
public function register()
{
    if ($this->app->environment() == 'local') {
        $this->app->register('Laracasts\Generators\GeneratorsServiceProvider');
    }
}
```

