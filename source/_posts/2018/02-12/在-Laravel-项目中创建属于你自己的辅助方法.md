---
title: 在 Laravel 项目中创建属于你自己的辅助方法
date: 2018-02-12 11:23:44
tags: [Laravel,PHP,转载]
categories: [Laravel,PHP]
---

这是一篇社区协同翻译的文章，已完成翻译，更多信息请点击 [协同翻译介绍][3]。
___
<!-- more -->
![file](https://dn-phphub.qbox.me/uploads/images/201801/29/1/UJ2FXDsehS.png)

Laravel提供了许多优秀的 [辅助函数][4] 可以方便的操作数组、文件路径、字符串以及路由，比如我们爱用的`dd()`方法（并不。）。
你可以为你的laravel应用和PHP包定义自己的辅助函数，通过Composer自动加载引入。
如果你是Laravel或PHP新手，那么让我们介绍一下如何创建由Laravel自动加载的自己的辅助函数。
___
#### Laravel 应用中新建 helpers 文件
第一件事是要在你的 Laravel 应用中加入 helpers 文件。helpers 文件放在哪个文件夹下，这取决于你自己的个人喜好。这边有几个建议放置 helpers 文件的路径：
1.  `app/helpers.php`
2.  `app/Http/helpers.php`

本人更倾向于： `app/helpers.php`。
### 自动加载
要使用PHP辅助方法， 你需要在运行时将它们加载到程序中. 在我职业生涯的初期, 在文件顶部看到这样的代码并不少见:
```php
require_once ROOT . '/helpers.php';
```
PHP 函数不能自动加载. 然而，我们有一个比使用`require`或`require_once`更好的解决方案。
如果你创建了一个新的Laravel项目, 你将会在`composer.json`文件中看到 `autoload` 和 `autoload-dev`的健:
```php
"autoload": {
    "classmap": [
        "database/seeds",
        "database/factories"
    ],
    "psr-4": {
        "App\\": "app/"
    }
},
"autoload-dev": {
    "psr-4": {
        "Tests\\": "tests/"
    }
},
```
如果你想添加一个helpers 文件, composer有一个 `files` 健 (是一个文件路径的数组) ，你可以在 `autoload` 中定义:
```php
"autoload": {
    "files": [
        "app/helpers.php"
    ],
    "classmap": [
        "database/seeds",
        "database/factories"
    ],
    "psr-4": {
        "App\\": "app/"
    }
},
```
一旦你在 `files` 数组中添加了一个新的路径, 你需要转储自动加载器:
```php
composer dump-autoload
```
现在每个需要helpers.php文件的请求将会自动加载 因为 Laravel需要在 `public/index.php` 中使用Composer的自动加载类:
```php
require __DIR__.'/../vendor/autoload.php';
```
### 定义函数
在你的辅助函数类中定义函数是非常简单的，尽管有一些值得注意的地方。所有的 laravel 辅助函数都应当做函数定义冲突的检查：
```php
if (! function_exists('env')) {
    function env($key, $default = null) {
        // ...
    }
}
```
这是很巧妙的做法，因为在当你不确定函数是否已经被定义的情况下依然可以确保程序正确运行。
我更喜欢用 `function_exists` 来检查我的辅助函数， 但如果你是在你的应用上下文中定义辅助函数， 你 _可以_ 不使用 `function_exists` 来做检查。
如果跳过检查， 每当你重复定义某个辅助函数的时候都会发生冲突， 可见检查是很有用的。
在实际情况中，冲突并不会像你想象的那样经常发生， 而你也应当确保你定义的函数名称不要过于常见。 你也可以为你的函数名称加上前缀以确保他们不会与其他相关函数发生冲突。
### Helper Example
我喜欢Rails的路径和URL辅助，当你定义一个丰富的路由时，你可以免费得到这些辅助。 例如, 一个 `photos` 资源路由会暴露路由的辅助方法， 就像 `new_photo_path`, `edit_photo_path`等等。
当我在Laravel里用资源路由的时候，我喜欢添加一些辅助方法，这些方法在我的模板里令定义路由更佳简单。在我的实现中，我喜欢利用我定义的规则，把URL辅助方法传递给一个Eloquent模型并且拿回一个资源路由。
```php
create_route($model);
edit_route($model);
show_route($model);
destroy_route($model);
```
这里你可能定义 `show_route` 在你的 `app/helpers.php` 文件里 (其他看起来差不多):
```php
if (! function_exists('show_route')) {
    function show_route($model, $resource = null)
    {
        $resource = $resource ?? plural_from_model($model);

        return route("{$resource}.show", $model);
    }
}

if (! function_exists('plural_from_model')) {
    function plural_from_model($model)
    {
        $plural = Str::plural(class_basename($model));

        return Str::kebab($plural);
    }
}
```
`plural_from_model()` 方法是仅仅一些可复用的代码，辅助路由的方法用这些代码来预测资源路由的基于我喜欢的命名规则的名字，这个命名规则是一个kebab-case(全小写并且分开的单词组，例如 "hello-world-hi")模型的复数。
例如，这里有一个来自模型资源名的例子:
```php
$model = new App\LineItem;
plural_from_model($model);
=> line-items

plural_from_model(new App\User);
=> users
```
用这规则来定义你的资源理由，就像 `routes/web.php` :
```php
Route::resource('line-items', 'LineItemsController');
Route::resource('users', 'UsersController');
```
然后在你的blade模板里面，你可以做以下操作:
```php
<a href="{{ show_route($lineItem) }}">
    {{ $lineItem->name }}
</a>
```
它会生成如下的HTML:
```php
<a href="http://localhost/line-items/1">
    Line Item #1
</a>
```
### Packages
你的Composer包也可以使用一个辅助文件来帮你在项目里使用任何你想要的辅助函数。
你将在 `composer.json` 文件采用相同的方法, 定义一个 `files` 键在你的辅助文件的一个数组。
添加`function_exists()` 来检查你的辅助函数是必要的，所以使用了你的代码的项目不会因命名冲突而中断。
你应该选择正确的在你包里是唯一的名字，并且如果你担心的方法名太普遍，可以考虑使用短前缀。
### 了解更多
查看 Composer的 [autoloading][5] 文档来了解更多关于引入文件,以及关于自动加载类的一些基本信息.
另一个推荐的资源是学习框架中所有适用的 [Laravel helpers][6] 并且通过查看 [Illuminate\Foundation helpers][7] 和 [Illuminate\Support helpers][8] 的源码学习他们是如何工作的.

> 注明：本文转自[Laravel-China][1]、[Summer][2]。


[1]: https://laravel-china.org/topics/8041/create-an-auxiliary-method-of-your-own-in-the-laravel-project
[2]: https://laravel-china.org/users/1
[3]: https://laravel-china.org/topics/7440/community-translation-function-online-star2
[4]: https://d.laravel-china.org/docs/5.5/helpers
[5]: https://getcomposer.org/doc/04-schema.md#autoload
[6]: https://d.laravel-china.org/docs/5.5/helpers
[7]: https://github.com/laravel/framework/blob/5.5/src/Illuminate/Foundation/helpers.php
[8]: https://github.com/laravel/framework/blob/5.5/src/Illuminate/Support/helpers.php
[9]: http://zh.wikipedia.org/wiki/Wikipedia:CC