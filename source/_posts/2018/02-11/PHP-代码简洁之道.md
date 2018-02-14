---
title: PHP 代码简洁之道(PHP Clean Code)
date: 2018-02-11 13:42:55
tags: [PHP]
categories: [PHP]
---

## 介绍
Robert C.Martin's 的 软件工程师准则 [Clean Code][4] 同样适用于PHP。它并不是一个编码风格指南，它指导我们用PHP写出具有可读性，可复用性且可分解的代码。
并非所有的准则都必须严格遵守，甚至一些已经成为普遍的约定。这仅仅作为指导方针，其中许多都是 _Clean Code_ 作者们多年来的经验。
<!-- more -->
> 灵感来自于 [clean-code-javascript][5]

尽管许多开发者依旧使用 PHP 5版本，但是这篇文章中绝大多数例子都是只能在 PHP 7.1+版本下运行。
## 变量
### 使用有意义的且可读的变量名
**不友好的：**
```php
$ymdstr = $moment->format('y-m-d');
```
**友好的：**
```php
$currentDate = $moment->format('y-m-d');
```
### 对同类型的变量使用相同的词汇
**不友好的：**
```php
getUserInfo();
getUserData();
getUserRecord();
getUserProfile();
```
**友好的：**
```php
getUser();
```
### 使用可搜索的名称（第一部分）
我们阅读的代码超过我们写的代码。所以我们写出的代码需要具备可读性、可搜索性，这一点非常重要。要我们去理解程序中没有名字的变量是非常头疼的。让你的变量可搜索吧！
**不具备可读性的代码：**
```php
//  见鬼的 448 是什么意思？
$result = $serializer->serialize($data, 448);
```
**具备可读性的：**
```php
$json = $serializer->serialize($data, JSON_UNESCAPED_SLASHES | JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE);
```
### 使用可搜索的名称（第二部分）
**不好的：**
```php
// 见鬼的 4 又是什么意思？
if ($user->access & 4) {
    // ...
}
```
**好的方式：**
```php
class User
{
    const ACCESS_READ = 1;
    const ACCESS_CREATE = 2;
    const ACCESS_UPDATE = 4;
    const ACCESS_DELETE = 8;
}

if ($user->access & User::ACCESS_UPDATE) {
    // do edit ...
}
```
### 使用解释性变量
**不好：**
```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,]+,\s*(.+?)\s*(\d{5})$/';
preg_match($cityZipCodeRegex, $address, $matches);

saveCityZipCode($matches[1], $matches[2]);
```
**一般：**
这个好点，但我们仍严重依赖正则表达式。
```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,]+,\s*(.+?)\s*(\d{5})$/';
preg_match($cityZipCodeRegex, $address, $matches);

[, $city, $zipCode] = $matches;
saveCityZipCode($city, $zipCode);
```
**很棒：**
通过命名子模式减少对正则表达式的依赖。
```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,]+,\s*(?<city>.+?)\s*(?<zipCode>\d{5})$/';
preg_match($cityZipCodeRegex, $address, $matches);

saveCityZipCode($matches['city'], $matches['zipCode']);
```
### 避免嵌套太深和提前返回 (第一部分)
使用太多`if else`表达式会导致代码难以理解。  
明确优于隐式。
**不好：**
```php
function isShopOpen($day): bool
{
    if ($day) {
        if (is_string($day)) {
            $day = strtolower($day);
            if ($day === 'friday') {
                return true;
            } elseif ($day === 'saturday') {
                return true;
            } elseif ($day === 'sunday') {
                return true;
            } else {
                return false;
            }
        } else {
            return false;
        }
    } else {
        return false;
    }
}
```
**很棒：**
```php
function isShopOpen(string $day): bool
{
    if (empty($day)) {
        return false;
    }

    $openingDays = [
        'friday', 'saturday', 'sunday'
    ];

    return in_array(strtolower($day), $openingDays, true);
}
```
### 避免嵌套太深和提前返回 (第二部分)
**不好：**
```php
function fibonacci(int $n)
{
    if ($n < 50) {
        if ($n !== 0) {
            if ($n !== 1) {
                return fibonacci($n - 1) + fibonacci($n - 2);
            } else {
                return 1;
            }
        } else {
            return 0;
        }
    } else {
        return 'Not supported';
    }
}
```
**很棒：**
```php
function fibonacci(int $n): int
{
    if ($n === 0 || $n === 1) {
        return $n;
    }

    if ($n > 50) {
        throw new \Exception('Not supported');
    }

    return fibonacci($n - 1) + fibonacci($n - 2);
}
```
### 避免心理映射
不要迫使你的代码阅读者翻译变量的意义。  
明确优于隐式。
**不好：**
```php
$l = ['Austin', 'New York', 'San Francisco'];

for ($i = 0; $i < count($l); $i++) {
    $li = $l[$i];
    doStuff();
    doSomeOtherStuff();
    // ...
    // ...
    // ...
    // Wait, what is `$li` for again?
    dispatch($li);
}
```
**很棒：**
```php
$locations = ['Austin', 'New York', 'San Francisco'];

foreach ($locations as $location) {
    doStuff();
    doSomeOtherStuff();
    // ...
    // ...
    // ...
    dispatch($location);
}
```
### 不要增加不需要的上下文
如果类名或对象名告诉你某些东西后，请不要在变量名中重复。
**小坏坏：**
```php
class Car
{
    public $carMake;
    public $carModel;
    public $carColor;

    //...
}
```
**好的方式：**
```php
class Car
{
    public $make;
    public $model;
    public $color;

    //...
}
```
### 使用默认参数而不是使用短路运算或者是条件判断
**不好的做法：**
这是不太好的因为 `$breweryName` 可以是 `NULL`.
```php
function createMicrobrewery($breweryName = 'Hipster Brew Co.'): void
{
    // ...
}
```
**还算可以的做法：**
这个做法比上面的更加容易理解，但是它需要很好的去控制变量的值.
```php
function createMicrobrewery($name = null): void
{
    $breweryName = $name ?: 'Hipster Brew Co.';
    // ...
}
```
**好的做法：**
你可以使用 [类型提示][6] 而且可以保证 `$breweryName` 不会为空 `NULL`.
```php
function createMicrobrewery(string $breweryName = 'Hipster Brew Co.'): void
{
    // ...
}
```
## 对比
### 使用 [相等运算符][7]
**不好的做法：**
```php
$a = '42';
$b = 42;
使用简单的相等运算符会把字符串类型转换成数字类型

if( $a != $b ) {
   //这个条件表达式总是会通过
}
```
表达式 $a != $b会返回false但实际上它应该是true !  
字符串类型 '42'是不同于数字类型的 42 
**好的做法：**
使用全等运算符会对比类型和值
```php
if( $a !== $b ) {
    //这个条件是通过的
}
```
表达式 $a !== $b 会返回true。
## 函数
### 函数参数（2 个或更少）
限制函数参数个数极其重要
这样测试你的函数容易点。有超过3个可选参数会导致一个爆炸式组合增长，你会有成吨独立参数情形要测试。
无参数是理想情况。1个或2个都可以，最好避免3个。
再多就需要加固了。通常如果你的函数有超过两个参数，说明他要处理的事太多了。 如果必须要传入很多数据，建议封装一个高级别对象作为参数。
**不友好的：**
```php
function createMenu(string $title, string $body, string $buttonText, bool $cancellable): void
{
    // ...
}
```
**友好的：**
```php
class MenuConfig
{
    public $title;
    public $body;
    public $buttonText;
    public $cancellable = false;
}

$config = new MenuConfig();
$config->title = 'Foo';
$config->body = 'Bar';
$config->buttonText = 'Baz';
$config->cancellable = true;

function createMenu(MenuConfig $config): void
{
    // ...
}
```
### 函数应该只做一件事情
这是迄今为止软件工程最重要的原则。函数做了超过一件事情时, 它们将变得难以编写、测试、推导。 而函数只做一件事情时，重构起来则非常简单，同时代码阅读起来也非常清晰。
掌握了这个原则，你就会领先许多其他的开发者。
**不好的：**
```php
function emailClients(array $clients): void
{
    foreach ($clients as $client) {
        $clientRecord = $db->find($client);
        if ($clientRecord->isActive()) {
            email($client);
        }
    }
}
```
**好的：**
```php
function emailClients(array $clients): void
{
    $activeClients = activeClients($clients);
    array_walk($activeClients, 'email');
}

function activeClients(array $clients): array
{
    return array_filter($clients, 'isClientActive');
}

function isClientActive(int $client): bool
{
    $clientRecord = $db->find($client);

    return $clientRecord->isActive();
}
```
### 函数的名称要说清楚它做什么
**不好的例子：**
```php
class Email
{
    //...

    public function handle(): void
    {
        mail($this->to, $this->subject, $this->body);
    }
}

$message = new Email(...);
// What is this? A handle for the message? Are we writing to a file now?
$message->handle();
```
**很好的例子：**
```php
class Email 
{
    //...

    public function send(): void
    {
        mail($this->to, $this->subject, $this->body);
    }
}

$message = new Email(...);
// Clear and obvious
$message->send();
```
### 函数只能是一个抽象级别
当你有多个抽象层次时，你的函数功能通常是做太多了。 分割函数功能使得重用性和测试更加容易。
**不好：**
```php
function parseBetterJSAlternative(string $code): void
{
    $regexes = [
        // ...
    ];

    $statements = explode(' ', $code);
    $tokens = [];
    foreach ($regexes as $regex) {
        foreach ($statements as $statement) {
            // ...
        }
    }

    $ast = [];
    foreach ($tokens as $token) {
        // lex...
    }

    foreach ($ast as $node) {
        // parse...
    }
}
```
**同样不是很好：**
我们已经完成了一些功能，但是 `parseBetterJSAlternative()` 功能仍然非常复杂，测试起来也比较麻烦。
```php
function tokenize(string $code): array
{
    $regexes = [
        // ...
    ];

    $statements = explode(' ', $code);
    $tokens = [];
    foreach ($regexes as $regex) {
        foreach ($statements as $statement) {
            $tokens[] = /* ... */;
        }
    }

    return $tokens;
}

function lexer(array $tokens): array
{
    $ast = [];
    foreach ($tokens as $token) {
        $ast[] = /* ... */;
    }

    return $ast;
}

function parseBetterJSAlternative(string $code): void
{
    $tokens = tokenize($code);
    $ast = lexer($tokens);
    foreach ($ast as $node) {
        // parse...
    }
}
```
**很好的：**
最好的解决方案是取出 `parseBetterJSAlternative()` 函数的依赖关系.
```php
class Tokenizer
{
    public function tokenize(string $code): array
    {
        $regexes = [
            // ...
        ];

        $statements = explode(' ', $code);
        $tokens = [];
        foreach ($regexes as $regex) {
            foreach ($statements as $statement) {
                $tokens[] = /* ... */;
            }
        }

        return $tokens;
    }
}

class Lexer
{
    public function lexify(array $tokens): array
    {
        $ast = [];
        foreach ($tokens as $token) {
            $ast[] = /* ... */;
        }

        return $ast;
    }
}

class BetterJSAlternative
{
    private $tokenizer;
    private $lexer;

    public function __construct(Tokenizer $tokenizer, Lexer $lexer)
    {
        $this->tokenizer = $tokenizer;
        $this->lexer = $lexer;
    }

    public function parse(string $code): void
    {
        $tokens = $this->tokenizer->tokenize($code);
        $ast = $this->lexer->lexify($tokens);
        foreach ($ast as $node) {
            // parse...
        }
    }
}
```
### 不要用标示作为函数的参数
标示就是在告诉大家，这个方法里处理很多事。前面刚说过，一个函数应当只做一件事。 把不同标示的代码拆分到多个函数里。
**不友好的：**
```php
function createFile(string $name, bool $temp = false): void
{
    if ($temp) {
        touch('./temp/'.$name);
    } else {
        touch($name);
    }
}
```
**友好的：**
```php
function createFile(string $name): void
{
    touch($name);
}

function createTempFile(string $name): void
{
    touch('./temp/'.$name);
}
```
### 避免副作用
一个函数应该只获取数值，然后返回另外的数值，如果在这个过程中还做了其他的事情，我们就称为副作用。副作用可能是写入一个文件，修改某些全局变量，或者意外的把你全部的钱给了陌生人。
现在，你的确需要在一个程序或者场合里要有副作用，像之前的例子，你也许需要写一个文件。你需要做的是把你做这些的地方集中起来。不要用几个函数和类来写入一个特定的文件。只允许使用一个服务来单独实现。
重点是避免常见陷阱比如对象间共享无结构的数据、使用可以写入任何的可变数据类型、不集中去处理这些副作用。如果你做了这些你就会比大多数程序员快乐。
**不好的：**
```php
// 这个全局变量在函数中被使用
// 如果我们在别的方法中使用这个全局变量，有可能我们会不小心将其修改为数组类型
$name = 'Ryan McDermott';

function splitIntoFirstAndLastName(): void
{
    global $name;

    $name = explode(' ', $name);
}

splitIntoFirstAndLastName();

var_dump($name); // ['Ryan', 'McDermott'];
```
**推荐的：**
```php
function splitIntoFirstAndLastName(string $name): array
{
    return explode(' ', $name);
}

$name = 'Ryan McDermott';
$newName = splitIntoFirstAndLastName($name);

var_dump($name); // 'Ryan McDermott';
var_dump($newName); // ['Ryan', 'McDermott'];
```
### 不要定义全局函数
在很多语言中定义全局函数是一个坏习惯，因为你定义的全局函数可能与其他人的函数库冲突，并且，除非在实际运用中遇到异常，否则你的API的使用者将无法觉察到这一点。
接下来我们来看看一个例子：当你想有一个配置数组，你可能会写一个 `config()` 的全局函数，但是这样会与其他人定义的库冲突。
**不好的：**
```php
function config(): array
{
    return  [
        'foo' => 'bar',
    ]
}
```
**好的：**
```php
class Configuration
{
    private $configuration = [];

    public function __construct(array $configuration)
    {
        $this->configuration = $configuration;
    }

    public function get(string $key): ?string
    {
        return isset($this->configuration[$key]) ? $this->configuration[$key] : null;
    }
}
```
获取配置需要先创建 `Configuration` 类的实例，如下：
```php
$configuration = new Configuration([
    'foo' => 'bar',
]);
```
现在，在你的应用中必须使用 `Configuration` 的实例了。
### 不要使用单例模式
单例模式是个 [反模式][8]。 以下转述 Brian Button 的观点：
1.  单例模式常用于 **全局实例**， 这么做为什么不好呢？ 因为在你的代码里 **你隐藏了应用的依赖关系**，而没有通过接口公开依赖关系 。避免全局的东西扩散使用是一种 [代码味道][9].
2.  单例模式违反了 **单一责任原则**： 依据的事实就是 **单例模式自己控制自身的创建和生命周期**.
3.  单例模式天生就导致代码紧 [耦合][10]。这使得在许多情况下用伪造的数据 **难于测试**。
4.  单例模式的状态会留存于应用的整个生命周期。 这会对测试产生第二次打击，**你只能让被严令需要测试的代码运行不了**收场，根本不能进行单元测试。为何？因为每一个单元测试应该彼此独立。

还有些来自 [Misko Hevery][11] 的深入思考，关于单例模式的 [问题根源][12]。
**不好的示范：**
```php
class DBConnection
{
    private static $instance;

    private function __construct(string $dsn)
    {
        // ...
    }

    public static function getInstance(): DBConnection
    {
        if (self::$instance === null) {
            self::$instance = new self();
        }

        return self::$instance;
    }

    // ...
}

$singleton = DBConnection::getInstance();
```
**好的示范：**
```php
class DBConnection
{
    public function __construct(string $dsn)
    {
        // ...
    }

     // ...
}
```
用 [DSN][13] 进行配置创建的 `DBConnection` 类实例。
```php
$connection = new DBConnection($dsn);
```
现在就必须在你的应用中使用 `DBConnection` 的实例了。
### 封装条件语句
**不友好的：**
```php
if ($article->state === 'published') {
    // ...
}
```
**友好的：**
```php
if ($article->isPublished()) {
    // ...
}
```
### 避免用反义条件判断
**不友好的：**
```php
function isDOMNodeNotPresent(\DOMNode $node): bool
{
    // ...
}

if (!isDOMNodeNotPresent($node))
{
    // ...
}
```
**友好的：**
```php
function isDOMNodePresent(\DOMNode $node): bool
{
    // ...
}

if (isDOMNodePresent($node)) {
    // ...
}
```
### 避免使用条件语句
这听起来像是个不可能实现的任务。 当第一次听到这个时，大部分人都会说，“没有`if`语句，我该怎么办？” 答案就是在很多情况下你可以使用多态性来实现同样的任务。 
接着第二个问题来了， “听着不错，但我为什么需要那样做？”，这个答案就是我们之前所学的干净代码概念：一个函数应该只做一件事情。如果你的类或函数有`if`语句，这就告诉了使用者你的类或函数干了不止一件事情。 记住，只要做一件事情。
**不好的：**
```php
class Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        switch ($this->type) {
            case '777':
                return $this->getMaxAltitude() - $this->getPassengerCount();
            case 'Air Force One':
                return $this->getMaxAltitude();
            case 'Cessna':
                return $this->getMaxAltitude() - $this->getFuelExpenditure();
        }
    }
}
```
**好的：**
```php
interface Airplane
{
    // ...

    public function getCruisingAltitude(): int;
}

class Boeing777 implements Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude() - $this->getPassengerCount();
    }
}

class AirForceOne implements Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude();
    }
}

class Cessna implements Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude() - $this->getFuelExpenditure();
    }
}
```
### 避免类型检测 (第 1 部分)
PHP 是无类型的，这意味着你的函数可以接受任何类型的参数。
有时这种自由会让你感到困扰，并且他会让你自然而然的在函数中使用类型检测。有很多方法可以避免这么做。
首先考虑 API 的一致性。
**不好的：**
```php
function travelToTexas($vehicle): void
{
    if ($vehicle instanceof Bicycle) {
        $vehicle->pedalTo(new Location('texas'));
    } elseif ($vehicle instanceof Car) {
        $vehicle->driveTo(new Location('texas'));
    }
}
```
**好的：**
```php
function travelToTexas(Traveler $vehicle): void
{
    $vehicle->travelTo(new Location('texas'));
}
```
### 避免类型检查（第 2 部分）
如果你正在使用像 字符串、数值、或数组这样的基础类型，你使用的是 PHP 版本是 PHP 7+，并且你不能使用多态，但仍然觉得需要使用类型检测，这时，你应该考虑 [类型定义][14] 或 严格模式。它为您提供了标准PHP语法之上的静态类型。
手动进行类型检查的问题是做这件事需要这么多的额外言辞，你所得到的虚假的『类型安全』并不能弥补丢失的可读性。保持你的代码简洁，编写良好的测试，并且拥有好的代码审查。
否则，使用PHP严格的类型声明或严格模式完成所有这些工作。
**不好的：**
```php
function combine($val1, $val2): int
{
    if (!is_numeric($val1) || !is_numeric($val2)) {
        throw new \Exception('Must be of type Number');
    }

    return $val1 + $val2;
}
```
**好的：**
```php
function combine(int $val1, int $val2): int
{
    return $val1 + $val2;
}
```
### 移除无用代码
死代码和重复代码一样糟糕。没有理由将它保留在你的代码库中。如果它没有被任何地方调用，那么就把他除掉！如果你任然需要它，它任然安全的保留在你的历史版本中。
**不好的：**
```php
function oldRequestModule(string $url): void
{
    // ...
}

function newRequestModule(string $url): void
{
    // ...
}

$request = newRequestModule($requestUrl);
inventoryTracker('apples', $request, 'www.inventory-awesome.io');
```
**好的：**
```php
function requestModule(string $url): void
{
    // ...
}

$request = requestModule($requestUrl);
inventoryTracker('apples', $request, 'www.inventory-awesome.io');
```
## 对象和数据结构
### 使用对象封装
在 PHP 中，你可以在方法中使用关键字，如 `public`, `protected` and `private`。
使用它们，你可以任意的控制、修改对象的属性。
*   当你除获取对象属性外还想做更多的操作时，你不需要修改你的代码
*   当 <code>set</code> 属性时，易于增加参数验证。
*   封装的内部表示。
*   容易在获取和设置属性时添加日志和错误处理。
*   继承这个类，你可以重写默认信息。
*   你可以延迟加载对象的属性，比如从服务器获取数据。
此外，这样的方式也符合OOP开发中的**开闭原则**
**不好的：**
```php
class BankAccount
{
    public $balance = 1000;
}

$bankAccount = new BankAccount();

// Buy shoes...
$bankAccount->balance -= 100;
```
**好的：**
```php
class BankAccount
{
    private $balance;

    public function __construct(int $balance = 1000)
    {
      $this->balance = $balance;
    }

    public function withdraw(int $amount): void
    {
        if ($amount > $this->balance) {
            throw new \Exception('Amount greater than available balance.');
        }

        $this->balance -= $amount;
    }

    public function deposit(int $amount): void
    {
        $this->balance += $amount;
    }

    public function getBalance(): int
    {
        return $this->balance;
    }
}

$bankAccount = new BankAccount();

// Buy shoes...
$bankAccount->withdraw($shoesPrice);

// Get balance
$balance = $bankAccount->getBalance();
```
### 让对象拥有privat/protected属性的成员
*   `public` 公有方法和属性对于变化来说是最危险的，因为一些外部的代码可能会轻易的依赖他们，但是你没法控制那些依赖他们的代码。 **类的变化对于类的所有使用者来说都是危险的。**
*   `protected` 受保护的属性变化和public公有的同样危险，因为他们在子类范围内是可用的。也就是说public和protected 之间的区别仅仅在于访问机制，只有封装才能保证属性是一致的。**任何在类内的变化对于所有继承子类来说都是危险的 。**
*   `private` 私有属性的变化可以保证代码 **只对单个类范围内的危险** (对于修改你是安全的，并且你不会有其他类似堆积木的影响 [Jenga effect][15]).
因此，请默认使用 `private` 属性，只有当需要对外部类提供访问属性的时候才采用 `public/protected` 属性。
更多的信息可以参考 [Fabien Potencier][16]写的针对这个专栏的文章 [blog post][17] .
**不好的：**
```php
class Employee
{
    public $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }
}

$employee = new Employee('John Doe');
echo 'Employee name: '.$employee->name; // Employee name: John Doe
```
**好的：**
```php
class Employee
{
    private $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }

    public function getName(): string
    {
        return $this->name;
    }
}

$employee = new Employee('John Doe');
echo 'Employee name: '.$employee->getName(); // Employee name: John Doe
```
## 类
### 组合优于继承
正如 the Gang of Four 所著的 [设计模式][18] 中所说,
我们应该尽量优先选择组合而不是继承的方式。使用继承和组合都有很多好处。
这个准则的主要意义在于当你本能的使用继承时，试着思考一下**组合**是否能更好对你的需求建模。
在一些情况下，是这样的。
接下来你或许会想，“那我应该在什么时候使用继承？”
答案依赖于你的问题，当然下面有一些何时继承比组合更好的说明：
1.  你的继承表达了“是一个”而不是“有一个”的关系（例如人类“是”动物，而用户“有”用户详情）。
2.  你可以复用基类的代码（人类可以像动物一样移动）。
3.  你想通过修改基类对所有派生类做全局的修改（当动物移动时，修改它们的能量消耗）。

**不好的：**
```php
class Employee 
{
    private $name;
    private $email;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
    }

    // ...
}

// 不好，因为Employees "有" taxdata
// 而EmployeeTaxData不是Employee类型的

class EmployeeTaxData extends Employee 
{
    private $ssn;
    private $salary;

    public function __construct(string $name, string $email, string $ssn, string $salary)
    {
        parent::__construct($name, $email);

        $this->ssn = $ssn;
        $this->salary = $salary;
    }

    // ...
}
```
**好的：**
```php
class EmployeeTaxData 
{
    private $ssn;
    private $salary;

    public function __construct(string $ssn, string $salary)
    {
        $this->ssn = $ssn;
        $this->salary = $salary;
    }

    // ...
}

class Employee 
{
    private $name;
    private $email;
    private $taxData;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
    }

    public function setTaxData(string $ssn, string $salary)
    {
        $this->taxData = new EmployeeTaxData($ssn, $salary);
    }

    // ...
}
```
### 避免流式接口
[流式接口][19] 是一种面向对象API的方法，旨在通过方法链[Method chaining][20]来提高源代码的可阅读性.
流式接口虽然需要一些上下文，需要经常构建对象，但是这种模式减少了代码的冗余度 (例如： [PHPUnit Mock Builder][21] 或 [Doctrine Query Builder][22])
但是同样它也带来了很多麻烦:
1.  破坏了封装[Encapsulation][23]
2.  破坏了原型[Decorators][24]
3.  难以模拟测试[mock][25]
4.  使得多次提交的代码难以理解

更多信息可以参考 [Marco Pivetta][26] 撰写的关于这个专题的文章 [blog post][27]
**不好的：**
```php
class Car
{
    private $make = 'Honda';
    private $model = 'Accord';
    private $color = 'white';

    public function setMake(string $make): self
    {
        $this->make = $make;

        // NOTE: Returning this for chaining
        return $this;
    }

    public function setModel(string $model): self
    {
        $this->model = $model;

        // NOTE: Returning this for chaining
        return $this;
    }

    public function setColor(string $color): self
    {
        $this->color = $color;

        // NOTE: Returning this for chaining
        return $this;
    }

    public function dump(): void
    {
        var_dump($this->make, $this->model, $this->color);
    }
}

$car = (new Car())
  ->setColor('pink')
  ->setMake('Ford')
  ->setModel('F-150')
  ->dump();
```
**好的：**
```php
class Car
{
    private $make = 'Honda';
    private $model = 'Accord';
    private $color = 'white';

    public function setMake(string $make): void
    {
        $this->make = $make;
    }

    public function setModel(string $model): void
    {
        $this->model = $model;
    }

    public function setColor(string $color): void
    {
        $this->color = $color;
    }

    public function dump(): void
    {
        var_dump($this->make, $this->model, $this->color);
    }
}

$car = new Car();
$car->setColor('pink');
$car->setMake('Ford');
$car->setModel('F-150');
$car->dump();
```
## SOLID
**SOLID** 是 Michael Feathers 推荐的便于记忆的首字母简写，它代表了 Robert Martin 命名的最重要的五个面向对象编程设计原则：
*   S: 职责单一原则 (SRP)
*   O: 开闭原则 (OCP)
*   L: 里氏替换原则 (LSP)
*   I: 接口隔离原则 (ISP)
*   D: 依赖反转原则 (DIP)

### 职责单一原则 Single Responsibility Principle (SRP)
正如 _Clean Code_ 书中所述，"修改一个类应该只为一个理由"。人们总是容易去用一堆方法 "塞满" 一个类，就好像当我们坐飞机上只能携带一个行李箱时，会把所有的东西都塞到这个箱子里。这样做带来的后果是：从逻辑上讲，这样的类不是高内聚的，并且留下了很多以后去修改它的理由。
将你需要修改类的次数降低到最小很重要，这是因为，当类中有很多方法时，修改某一处，你很难知晓在整个代码库中有哪些依赖于此的模块会被影响。
**不好的：**
```php
class UserSettings
{
    private $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }

    public function changeSettings(array $settings): void
    {
        if ($this->verifyCredentials()) {
            // ...
        }
    }

    private function verifyCredentials(): bool
    {
        // ...
    }
}
```
**好的：**
```php
class UserAuth 
{
    private $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }

    public function verifyCredentials(): bool
    {
        // ...
    }
}

class UserSettings 
{
    private $user;
    private $auth;

    public function __construct(User $user) 
    {
        $this->user = $user;
        $this->auth = new UserAuth($user);
    }

    public function changeSettings(array $settings): void
    {
        if ($this->auth->verifyCredentials()) {
            // ...
        }
    }
}
```
### 开闭原则 (OCP)
如 Bertrand Meyer 所述, "软件实体 (类, 模块, 功能, 等) 应该对扩展开放, 但对修改关闭. "这意味着什么? 这个原则大体上是指你应该允许用户在不修改已有代码情况下添加功能.
**不好的：**
```php
abstract class Adapter
{
    protected $name;

    public function getName(): string
    {
        return $this->name;
    }
}

class AjaxAdapter extends Adapter
{
    public function __construct()
    {
        parent::__construct();

        $this->name = 'ajaxAdapter';
    }
}

class NodeAdapter extends Adapter
{
    public function __construct()
    {
        parent::__construct();

        $this->name = 'nodeAdapter';
    }
}

class HttpRequester
{
    private $adapter;

    public function __construct(Adapter $adapter)
    {
        $this->adapter = $adapter;
    }

    public function fetch(string $url): Promise
    {
        $adapterName = $this->adapter->getName();

        if ($adapterName === 'ajaxAdapter') {
            return $this->makeAjaxCall($url);
        } elseif ($adapterName === 'httpNodeAdapter') {
            return $this->makeHttpCall($url);
        }
    }

    private function makeAjaxCall(string $url): Promise
    {
        // request and return promise
    }

    private function makeHttpCall(string $url): Promise
    {
        // request and return promise
    }
}
```
**好的：**
```php
interface Adapter
{
    public function request(string $url): Promise;
}

class AjaxAdapter implements Adapter
{
    public function request(string $url): Promise
    {
        // request and return promise
    }
}

class NodeAdapter implements Adapter
{
    public function request(string $url): Promise
    {
        // request and return promise
    }
}

class HttpRequester
{
    private $adapter;

    public function __construct(Adapter $adapter)
    {
        $this->adapter = $adapter;
    }

    public function fetch(string $url): Promise
    {
        return $this->adapter->request($url);
    }
}
```
### 里氏代换原则 (LSP)
这是一个简单概念的可怕术语。它通常被定义为“如果S是T的一个子类型，则T型对象可以替换为S型对象”
(i.e., S类型的对象可以替换T型对象)在不改变程序的任何理想属性的情况下 (正确性,任务完成度,etc.)." 这个一个更可怕的定义.
这个的最佳解释是，如果你有个父类和一个子类,然后基类和子类可以互换使用而不会得到不正确的结果.这或许依然令人疑惑, 所以我们来看下经典的正方形-矩形例子。几何定义, 正方形是矩形, 但是，如果你通过继承建立了“IS-a”关系的模型，你很快就会陷入麻烦。.
**不好的：**
```php
class Rectangle
{
    protected $width = 0;
    protected $height = 0;

    public function render(int $area): void
    {
        // ...
    }

    public function setWidth(int $width): void
    {
        $this->width = $width;
    }

    public function setHeight(int $height): void
    {
        $this->height = $height;
    }

    public function getArea(): int
    {
        return $this->width * $this->height;
    }
}

class Square extends Rectangle
{
    public function setWidth(int $width): void
    {
        $this->width = $this->height = $width;
    }

    public function setHeight(int $height): void
    {
        $this->width = $this->height = $height;
    }
}

/**
 * @param Rectangle[] $rectangles
 */
function renderLargeRectangles(array $rectangles): void
{
    foreach ($rectangles as $rectangle) {
        $rectangle->setWidth(4);
        $rectangle->setHeight(5);
        $area = $rectangle->getArea(); // BAD: Will return 25 for Square. Should be 20.
        $rectangle->render($area);
    }
}

$rectangles = [new Rectangle(), new Rectangle(), new Square()];
renderLargeRectangles($rectangles);
```
**好的：**
```php
abstract class Shape
{
    abstract public function getArea(): int;

    public function render(int $area): void
    {
        // ...
    }
}

class Rectangle extends Shape
{
    private $width;
    private $height;

    public function __construct(int $width, int $height)
    {
        $this->width = $width;
        $this->height = $height;
    }

    public function getArea(): int
    {
        return $this->width * $this->height;
    }
}

class Square extends Shape
{
    private $length;

    public function __construct(int $length)
    {
        $this->length = $length;
    }

    public function getArea(): int
    {
        return pow($this->length, 2);
    }
}

/**
 * @param Rectangle[] $rectangles
 */
function renderLargeRectangles(array $rectangles): void
{
    foreach ($rectangles as $rectangle) {
        $area = $rectangle->getArea(); 
        $rectangle->render($area);
    }
}

$shapes = [new Rectangle(4, 5), new Rectangle(4, 5), new Square(5)];
renderLargeRectangles($shapes);
```
### 接口隔离原则 (ISP)
ISP 指出 "客户不应该被强制依赖于他们用不到的接口." 
一个好的例子来观察证实此原则的是针对需要大量设置对象的类，不要求客户端设置大量的选项是有益的, 因为多数情况下他们不需要所有的设置. 使他们可选来避免产生一个“臃肿的接口”.
**不好的：**
```php
interface Employee
{
    public function work(): void;

    public function eat(): void;
}

class Human implements Employee
{
    public function work(): void
    {
        // ....working
    }

    public function eat(): void
    {
        // ...... eating in lunch break
    }
}

class Robot implements Employee
{
    public function work(): void
    {
        //.... working much more
    }

    public function eat(): void
    {
        //.... robot can't eat, but it must implement this method
    }
}
```
**好的：**
并不是每个工人都是雇员, 但每个雇员都是工人.
```php
interface Workable
{
    public function work(): void;
}

interface Feedable
{
    public function eat(): void;
}

interface Employee extends Feedable, Workable
{
}

class Human implements Employee
{
    public function work(): void
    {
        // ....working
    }

    public function eat(): void
    {
        //.... eating in lunch break
    }
}

// robot can only work
class Robot implements Workable
{
    public function work(): void
    {
        // ....working
    }
}
```
### 依赖反转原则 (DIP)
这一原则规定了两项基本内容:
1.  高级模块不应依赖于低级模块. 两者都应该依赖于抽象.
2.  抽象类不应依赖于实例. 实例应该依赖于抽象.

一开始可能很难去理解, 但是你如果工作中使用过php框架（如Symfony）,你应该见过以依赖的形式执行这一原则
依赖注入 (DI). 虽然他们不是相同的概念, DIP可以让高级模块不需要了解其低级模块的详细信息而安装它们.
通过依赖注入可以做到. 这样做的一个巨大好处是减少了模块之间的耦合. 耦合是一种非常糟糕的开发模式，因为它使您的代码难以重构.
**不好的：**
```php
class Employee
{
    public function work(): void
    {
        // ....working
    }
}

class Robot extends Employee
{
    public function work(): void
    {
        //.... working much more
    }
}

class Manager
{
    private $employee;

    public function __construct(Employee $employee)
    {
        $this->employee = $employee;
    }

    public function manage(): void
    {
        $this->employee->work();
    }
}
```
**优秀的：**
```php
interface Employee
{
    public function work(): void;
}

class Human implements Employee
{
    public function work(): void
    {
        // ....working
    }
}

class Robot implements Employee
{
    public function work(): void
    {
        //.... working much more
    }
}

class Manager
{
    private $employee;

    public function __construct(Employee $employee)
    {
        $this->employee = $employee;
    }

    public function manage(): void
    {
        $this->employee->work();
    }
}
```
## 别写重复代码 (DRY)
试着去遵循 [DRY][28] 原则。
尽你最大的努力去避免复制代码，它是一种非常糟糕的行为，复制代码通常意味着当你需要变更一些逻辑时，你需要修改不止一处。
试想一下，如果你在经营一家餐厅，并且你需要记录你仓库的进销记录：包括所有的土豆，洋葱，大蒜，辣椒，等等。如果你使用多个表格来管理进销记录，当你用其中一些土豆做菜时，你需要更新所有的表格。如果你只有一个列表的话就只需要更新一个地方。
通常情况下你复制代码的原因可能是它们大多数都是一样的，只不过有两个或者多个略微不同的逻辑，但是由于这些区别，最终导致你写出了两个或者多个隔离的但大部分相同的方法，移除重复的代码意味着用一个 function/module/class 创建一个能处理差异的抽象。
正确的抽象是非常关键的，这正是为什么你必须学习遵守在 **Classes** 章节展开讨论的的**SOLID**原则，不合理的抽象比复制代码更糟糕，所以请务必谨慎！说了这么多，如果你能设计一个合理的抽象，就去实现它！最后再说一遍，不要写重复代码，否则你会发现当你想修改一个逻辑时，你必须去修改多个地方！
**不好的：**
```php
function showDeveloperList(array $developers): void
{
    foreach ($developers as $developer) {
        $expectedSalary = $developer->calculateExpectedSalary();
        $experience = $developer->getExperience();
        $githubLink = $developer->getGithubLink();
        $data = [
            $expectedSalary,
            $experience,
            $githubLink
        ];

        render($data);
    }
}

function showManagerList(array $managers): void
{
    foreach ($managers as $manager) {
        $expectedSalary = $manager->calculateExpectedSalary();
        $experience = $manager->getExperience();
        $githubLink = $manager->getGithubLink();
        $data = [
            $expectedSalary,
            $experience,
            $githubLink
        ];

        render($data);
    }
}
```
**好的：**
```php
function showList(array $employees): void
{
    foreach ($employees as $employee) {
        $expectedSalary = $employee->calculateExpectedSalary();
        $experience = $employee->getExperience();
        $githubLink = $employee->getGithubLink();
        $data = [
            $expectedSalary,
            $experience,
            $githubLink
        ];

        render($data);
    }
}
```
**非常好：**
最好让你的代码紧凑一点。
```php
function showList(array $employees): void
{
    foreach ($employees as $employee) {
        render([
            $employee->calculateExpectedSalary(),
            $employee->getExperience(),
            $employee->getGithubLink()
        ]);
    }
}
```
> 注明：本文转自[Laravel-China][1]、[Summer][2]。
                

[1]: https://laravel-china.org/topics/7774/the-conciseness-of-the-php-code-php-clean-code
[2]: https://laravel-china.org/users/1
[3]: https://laravel-china.org/topics/7440/community-translation-function-online-star2
[4]: https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882
[5]: https://github.com/ryanmcdermott/clean-code-javascript
[6]: http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration
[7]: http://php.net/manual/en/language.operators.comparison.php
[8]: https://en.wikipedia.org/wiki/Singleton_pattern
[9]: https://en.wikipedia.org/wiki/Code_smell
[10]: https://en.wikipedia.org/wiki/Coupling_%28computer_programming%29
[11]: http://misko.hevery.com/about/
[12]: http://misko.hevery.com/2008/08/25/root-cause-of-singletons/
[13]: http://php.net/manual/en/pdo.construct.php#refsect1-pdo.construct-parameters
[14]: http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration
[15]: http://www.urbandictionary.com/define.php?term=Jengaphobia&amp;defid=2494196
[16]: https://github.com/fabpot
[17]: http://fabien.potencier.org/pragmatism-over-theory-protected-vs-private.html
[18]: https://en.wikipedia.org/wiki/Design_Patterns
[19]: https://en.wikipedia.org/wiki/Fluent_interface
[20]: https://en.wikipedia.org/wiki/Method_chaining
[21]: https://phpunit.de/manual/current/en/test-doubles.html
[22]: http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/query-builder.html
[23]: https://en.wikipedia.org/wiki/Encapsulation_%28object-oriented_programming%29
[24]: https://en.wikipedia.org/wiki/Decorator_pattern
[25]: https://en.wikipedia.org/wiki/Mock_object
[26]: https://github.com/Ocramius
[27]: https://ocramius.github.io/blog/fluent-interfaces-are-evil/
[28]: https://en.wikipedia.org/wiki/Don%27t_repeat_yourself
[29]: http://zh.wikipedia.org/wiki/Wikipedia:CC