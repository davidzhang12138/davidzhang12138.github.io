---
title: 手动实现 DI 容器（转载）
date: 2018-02-11 10:22:55
tags: [Laravel,PHP,转载]
categories: [Laravel,PHP]
---
> 注明：本文转自[Laravel-China][1]、[zhengzean][2]。

关于依赖注入相信大家应该都经常接触或者至少有所耳闻，比较知名的框架都支持依赖注入，比如Java的Spring，PHP的Laravel、Symfony等。现在我开始手动实现一个简陋的DI容器吧。
## 由开车开始
先开个车，为大家举个栗子：
<!-- more -->
```php
class Driver
{
    public function drive()
    {
        $car = new Car();
        echo ‘老司机正在驾驶’, $car->getCar(), PHP_EOL;
    }
}

class Car
{
    protected $name = ‘普通汽车’;

    public function getCar()
    {
        return $this->name;
    }
}
```
有两个类，Driver和Car，老司机Driver有个方法driver，在调用的时候首先得整辆车$car，然后发车。
大多数同学都写过这样或者类似的代码，这样的代码单看没啥毛病，挺正常的。但是，如果我要换辆车，开普通车撩不到妹。
```php
class Benz extends Car
{
    protected $name = ‘奔驰’;
}
```
这时候就需要做一个比较恶心的操作了，得改老司机的代码了。（老司机：我做错了什么？换辆车还得让我重学驾照……）。
因此我们需要把让Car为外界注入，将Driver和Car解耦，不是老司机自己开车的时候还得自己去造车。于是就有了下面的结果
```php
class Driver
{
    protected $car;

    public function __construct(Car $car)
    {
        $this->car = $car;
    }

    public function drive()
    {
        echo '老司机正在驾驶', $this->car->getCar(), PHP_EOL;
    }
}
```
此时Driver和Car两个类已经解耦，这两个类的依赖，依靠上层代码去管理。此时，老司机会这样“开车”：
```php
$car = new Car();
$driver = new Driver($car);
$driver->drive();
```
此时，我们创建Driver依赖的实例，并注入。上面的例子，我们实现了依赖注入，不过是手动的，写起来感觉还是不爽。
这么繁重的活怎么能手动来做呢，得让程序自己去做。于是乎，DI容器诞生。
## 依赖注入容器
依赖注入与IoC模式类似工厂模式，是一种解决调用者和被调用者依赖耦合关系的模式。
它解决了对象之间的依赖关系，使得对象只依赖IoC/DI容器，不再直接相互依赖，实现松耦合，
然后在对象创建时，由IoC/DI容器将其依赖（Dependency）的对象注入（Inject）其内，这样做可以最大程度实现松耦合。
依赖注入说白一点，就是容器将某个类依赖的其他类的实例注入到这个类的实例中。

这段话可能说的有点抽象，回到刚才的例子吧。刚刚我手动完成了依赖注入，比较麻烦，如果一个大型的项目这样做肯定会觉得很繁琐，而且不够优雅。
因此我们需要有一位总管代替我们去干这个，这个总管就是容器。类的依赖管理全部交给容器去完成。因此，一般来说容器是一个全局的对象，大家共有的。
## 做一个自己的DI容器
写一个功能，我们首先需要分析问题，因此我们先要明白，对于一个简单的DI容器需要哪些功能，这直接关系到我们代码的编写。
对于一个简单的容器，至少需要满足以下几点：

> 1.创建所需类的实例
> 2.完成依赖管理（DI）
> 3.可以获取单例的实例
> 4.全局唯一

综上，我们的容器类大约长这样：
```php
class Container
{
    /**
     * 单例
     * @var Container
     */
    protected static $instance;

    /**
     * 容器所管理的实例
     * @var array
     */
    protected $instances = [];

    private function __construct(){}

    private function __clone(){}

    /**
     * 获取单例的实例
     * @param string $class
     * @param array ...$params
     * @return object
     */
    public function singleton($class, ...$params)
    {}

    /**
     * 获取实例（每次都会创建一个新的）
     * @param string $class
     * @param array ...$params
     * @return object
     */
    public function get($class, ...$params)
    {}

    /**
     * 工厂方法，创建实例，并完成依赖注入
     * @param string $class
     * @param array $params
     * @return object
     */
    protected function make($class, $params = [])
    {}

    /**
     * @return Container
     */
    public static function getInstance()
    {
        if (null === static::$instance) {
            static::$instance = new static();
        }
        return static::$instance;
    }
}
```
大体骨架已经确定，接下来进入最核心的make方法：
```php
protected function make($class, $params = [])
{
  //如果不是反射类根据类名创建
  $class = is_string($class) ? new ReflectionClass($class) : $class;

  //如果传的入参不为空，则根据入参创建实例
  if (!empty($params)) {
    return $class->newInstanceArgs($params);
  }

  //获取构造方法
  $constructor = $class->getConstructor();

  //获取构造方法参数
  $parameterClasses = $constructor ? $constructor->getParameters() : [];

  if (empty($parameterClasses)) {
    //如果构造方法没有入参，直接创建
    return $class->newInstance();
  } else {
    //如果构造方法有入参，迭代并递归创建依赖类实例
    foreach ($parameterClasses as $parameterClass) {
      $paramClass = $parameterClass->getClass();
      $params[] = $this->make($paramClass);
    }
    //最后根据创建的参数创建实例，完成依赖的注入
    return $class->newInstanceArgs($params);
  }
}
```
为了容器的易用，我做了一些完善：
1. 实现ArrayAccess接口，使单例实例可以直接通过array的方式获取，如果该实例没有，则创建
2. 重写__get方法，更方便的获取
最终版：
```php
class Container implements ArrayAccess
{
    /**
     * 单例
     * @var Container
     */
    protected static $instance;

    /**
     * 容器所管理的实例
     * @var array
     */
    protected $instances = [];

    private function __construct(){}

    private function __clone(){}

    /**
     * 获取单例的实例
     * @param string $class
     * @param array  ...$params
     * @return object
     */
    public function singleton($class, ...$params)
    {
        if (isset($this->instances[$class])) {
            return $this->instances[$class];
        } else {
            $this->instances[$class] = $this->make($class, $params);
        }

        return $this->instances[$class];
    }

    /**
     * 获取实例（每次都会创建一个新的）
     * @param string $class
     * @param array  ...$params
     * @return object
     */
    public function get($class, ...$params)
    {
        return $this->make($class, $params);
    }

    /**
     * 工厂方法，创建实例，并完成依赖注入
     * @param string $class
     * @param array  $params
     * @return object
     */
    protected function make($class, $params = [])
    {
        //如果不是反射类根据类名创建
        $class = is_string($class) ? new ReflectionClass($class) : $class;

        //如果传的入参不为空，则根据入参创建实例
        if (!empty($params)) {
            return $class->newInstanceArgs($params);
        }

        //获取构造方法
        $constructor = $class->getConstructor();

        //获取构造方法参数
        $parameterClasses = $constructor ? $constructor->getParameters() : [];

        if (empty($parameterClasses)) {
            //如果构造方法没有入参，直接创建
            return $class->newInstance();
        } else {
            //如果构造方法有入参，迭代并递归创建依赖类实例
            foreach ($parameterClasses as $parameterClass) {
                $paramClass = $parameterClass->getClass();
                $params[] = $this->make($paramClass);
            }
            //最后根据创建的参数创建实例，完成依赖的注入
            return $class->newInstanceArgs($params);
        }
    }

    /**
     * @return Container
     */
    public static function getInstance()
    {
        if (null === static::$instance) {
            static::$instance = new static();
        }

        return static::$instance;
    }

    public function __get($class)
    {
        if (!isset($this->instances[$class])) {
            $this->instances[$class] = $this->make($class);
        }
        return $this->instances[$class];
    }

    public function offsetExists($offset)
    {
        return isset($this->instances[$offset]);
    }

    public function offsetGet($offset)
    {
        if (!isset($this->instances[$offset])) {
            $this->instances[$offset] = $this->make($offset);
        }
        return $this->instances[$offset];
    }

    public function offsetSet($offset, $value)
    {
    }

    public function offsetUnset($offset) {
        unset($this->instances[$offset]);
    }
}
```
现在借助容器我们写一下上面的代码：
```php
$driver = $app->get(Driver::class);
$driver->drive();

//output:老司机正在驾驶普通汽车
```
就这么简单，老司机就能发车。这里默认注入的是Car的实例，如果需要开奔驰，那只需要这样：
```php
$benz = $app->get(Benz::class);
$driver = $app->get(Driver::class, $benz);
$driver->drive();

//output:老司机正在驾驶奔驰
```
按照PSR-11的要求，依赖注入容器需要实现Psr\Container\ContainerInterface接口，这里只是演示并未去实现，
因为那需要引入Psr依赖库，比较麻烦，其实也很简单，只是多了几个方法，有兴趣的可以自己去了解下PSR-11的要求([传送门][3])
这里只是实现了一个非常简陋的DI容器，实际中还需要考虑很多，而且这里的容器功能上还很简陋。还有一些坑没处理，比如出现循环依赖怎么处理、延迟加载的机制……
这里只是我周末闲暇练手的一点记录，大家如果有兴趣可以阅读以下Laravel或者Symfony容器那块的源码，或者了解一下Spring的容器。后续有空我也会继续完善。


[1]: https://laravel-china.org/articles/7573/manual-implementation-of-the-di-container
[2]: https://laravel-china.org/users/18655
[3]: https://github.com/container-interop/fig-standards/blob/master/proposed/container.md