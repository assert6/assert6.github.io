---
title: 【手把手教你】自定义Swoft 注解和AOP
tags: [Swoft, 手把手教你]
date: 2018-07-20
---

# 序言
前两篇关于[Swoft RPC 的使用教程](/2018-06-07/swoft/用Swoft%20搭建微服务(TCP%20RPC)2)得到很多小伙伴的支持

并且反映网上很多关于Swoft 的讲解太过于深♂入, 很多小伙伴看不懂

这是很正常的, 因为开发者们对于使用难度的理解跟我们有些偏差

身为开发者自然知道如何使用, 认为不需要讲解, 也就是所谓的**知识的诅咒**

所以很多关于Swoft 的讲解都是底层原理

那在这我就开一个大佬们不屑的新手向讲解教程 (￣_ ￣ )

叫做`[手把手教你]`系列

只要跟着一点点练习, 熟练使用是没问题滴

# 前置技能

1. 会使用Swoft 的MVC
2. 使用了Swoft 的注解
3. 理解注解只是配置的另一种展现方式
4. PHPStorm 安装了PHP Annotations 插件

下面就让我们开始吧

# 注解类
想要定义一个注解是非常简单的

新建注解类`app\Module\Test\Annotations\Test.php`

```php
    <?php
    
    namespace App\Module\Test\Annotations;
    
    /**
     * 测试注解
     * @Annotation
     * @Target("ALL")
     */
    class Test
    {
        /**
         * @var string 构造参数
         */
        private $name = '';
    
        public function __construct(array $values)
        {
            if (isset($values['name'])) {
                $this->name = $values['name'];
            }
        }
    
        /**
         * @return string
         */
        public function getName(): string
        {
            return $this->name;
        }
    
        /**
         * @param string $name
         */
        public function setName(string $name)
        {
            $this->name = $name;
        }
    }
```

这里有三点需要注意:

1 类注解要加`@Annotation`, 用来声明这是一个注解类

2 类名不需要加`Annotation`后缀

3 类注解`@Target()`为`Doctrine\Common\Annotations\Annotation\Target.php`, 参数可以填ALL|CLASS|METHOD|PROPERTY|ANNOTATION, 表示该注解使用的级别, 类注解还是方法注解,属性注解,或者全部都能使用

这样我们自定义的注解标签就完成了

去`IndexController.php`里试一下

![title](5b52dc8aab64414100000d1f.png)

注意不要忘记`use`我们的注解类

注解类`Test`的`@Target`设置为`ALL`, 可以同时在类和方法上使用(属性也可以的)

为了体现使用了注解, 可以在注解类`Test`的构造函数中进行输出
![title](5b52c3e5ab644142f5000912.png)

重启一下程序

![title](5b52dcb0ab64414100000d28.png)

这样就是成功了, 输出多个是正常现象, 毕竟多进程, 也是`Swoft`高性能的原因

# 注解解析类 Parser
正如前置技能里所说

注解只是配置的另一种展现方式

任何逻辑都**不要**在注解类里处理

新建注解解析类`app\Module\Test\Parser\TestParser.php`

```php
    <?php
    
    namespace App\Module\Test\Parser;
    
    use App\Module\Test\Collector\TestCollector;
    use Swoft\Bean\Parser\AbstractParser;
    
    class TestParser extends AbstractParser
    {
        public function parser(
            string $className,
            $objectAnnotation = null,
            string $propertyName = '',
            string $methodName = '',
            $propertyValue = null
        )
        {
            TestCollector::collect($className, $objectAnnotation, $propertyName, $methodName, $propertyValue);
        }
    }
```

解释一下这几个参数的意义:

> `$className` 当前注解所在的类名
>
> `$objectAnnotation` 当前注解所实例化的注解类`new Test([name="666"])`
>
> `$propertyName` 当前注解所在的属性名(如果是属性注解)
>
> `$methodName` 当前注解所在的方法名(如果是方法注解)
>
> `$propertyValue` 当前注解所在的属性(如果是属性注解)

这里也不要处理逻辑, 因为此刻程序还处于初始化阶段, 没有请求数据

注解解析类`Parser`只做了一件事, 就是把注解类存入注解收集类

什么是注解收集类呢?

# 注解收集类 Collector
注解收集类`Collector`非常简单, 相当于一个全局数组方便我们后续处理而已

新建注解收集类`app\Module\Test\Collector\TestCollector.php`
```php
    <?php
    
    namespace App\Module\Test\Collector;
    
    use Swoft\Bean\CollectorInterface;
    
    class TestCollector implements CollectorInterface
    {
        private static $test = [];
    
        public static function collect(
            string $className,
            $objectAnnotation = null,
            string $propertyName = '',
            string $methodName = '',
            $propertyValue = null
        )
        {
            self::$test[$className][$methodName] = $objectAnnotation;
        }
    
        public static function getCollector()
        {
            return self::$test;
        }
    }
```

正如所见, 只是存取`$objectAnnotation`注解实例, 方便我们后面使用

本篇举例的是方法注解, 所以默认`$methodName`不为空

注解收集类`Collector`被注解解析类`Parser`调用

那注解解析类`Parser`被谁调用呢?

# 注解封装类 Wrapper
新建注解封装类`app\Module\Test\Wrapper\TestWrapper.php`
```php
    <?php
    
    namespace App\Module\Test\Wrapper;
    
    use App\Module\Test\Annotations\Test;
    use Swoft\Bean\Wrapper\AbstractWrapper;
    
    class TestWrapper extends AbstractWrapper
    {
        /**
         * @var array 解析哪些注解(类级)
         */
        protected $classAnnotations = [];
    
        /**
         * @var array 解析哪些注解(属性级)
         */
        protected $propertyAnnotations = [];
    
        /**
         * @var array 解析哪些注解(方法级)
         */
        protected $methodAnnotations = [
            Test::class,
        ];
    
        /**
         * 是否解析类注解
         * @param array $annotations
         * @return bool
         */
        public function isParseClassAnnotations(array $annotations): bool
        {
            return false;
        }
    
        /**
         * 是否解析属性注解
         * @param array $annotations
         * @return bool
         */
        public function isParsePropertyAnnotations(array $annotations): bool
        {
            return false;
        }
    
        /**
         * 是否解析方法注解
         * @param array $annotations
         * @return bool
         */
        public function isParseMethodAnnotations(array $annotations): bool
        {
            return true;
        }
    }
```

当我们的类注解被实例化时, 会触发注解封装类`{注解标签名}Wrapper`

注解封装类`Wrapper`来决定是否触发注解解析类`Parser`

解释一下, 可能不好理解

> 这里`isParseClassAnnotations` 返回`false`, 意味着**类注解**略过不解析;
>
> 同理`isParsePropertyAnnotations` 返回`false`, **属性注解**略过不解析;
>
> 而`isParseMethodAnnotations` 返回`true`,
> 那么对应的`$methodAnnotations`里的注解类全部触发解析,
> 这里的`Test::class`会触发对应的`TestParser`;

抽象的说, 注解封装类`Wrapper`回答了两个问题: `是否解析`, `解析哪些`

具象的说, 这里的封装类`TestWrapper`回答的就是:"只解析方法注解`@Test`"

可以在注解解析类`Parser`的构造方法中进行输出
![title](5b52e317ab64414100000f23.png)

重启程序, 我们来看一下!
![title](5b52e34bab64414100000f2f.png)
这里可以看到, 在注解类`Test`里输出的内容还有类注解的`666`, 到了注解解析类`TestParser`, 就没有了`666`, 只有方法注解`777`

# 总结

> 实例化注解类`Test`, 询问注解封装类`TestWrapper`解析哪些注解

> 注解封装类`TestWrapper`回答解析方法上的`Test`, 于是方法级的注解`Test`被注解解析类`TestParser`存进了注解收集类`TestCollector`

### 注意!!!!
### 程序只会主动扫描类注解, 然后扫描类注解Wrapper下指定的方法注解和属性注解!!
### 如果你的方法注解不存在于任何类注解的Wrapper下, 则是不会被解析的!!

# 如何使用
收集完毕下面举例如何使用

## 中间件
新建中间件`app\Module\Test\Middlewares\TestMiddleware.php`
```php
    <?php
    
    namespace App\Module\Test\Middlewares;
    
    use App\Module\Test\Annotations\Test;
    use App\Module\Test\Collector\TestCollector;
    use Psr\Http\Message\ResponseInterface;
    use Psr\Http\Message\ServerRequestInterface;
    use Psr\Http\Server\RequestHandlerInterface;
    use Swoft\Bean\Annotation\Bean;
    use Swoft\Core\RequestContext;
    use Swoft\Http\Message\Middleware\MiddlewareInterface;
    
    /**
     * @Bean()
     */
    class TestMiddleware implements MiddlewareInterface
    {
        public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface
        {
            echo "in Middleware\n";
            $response = $handler->handle($request);
    
            // 当前请求的控制器
            $controllerClass  = RequestContext::getContextDataByKey('controllerClass');
            // 当前请求的方法
            $controllerAction = RequestContext::getContextDataByKey('controllerAction');
    
            $collector = TestCollector::getCollector();
    
            /* @var Test $test*/
            if ($test = $collector[$controllerClass][$controllerAction]){
                print_r($test->getName()." in Middleware\n");
            }
            return $response;
        }
    }
```

这里可以看到, 之前负责收集注解的注解收集类`TestCollector` 派上用处了, 可以非常方便的通过他获取到当前请求方法的注解信息

这需要注意, `RequestContext::getContextDataByKey('controllerClass')`获取请求控制器的方法要在`$handler->handle($request)`后才有值, 因为负责路由的中间件还没执行到XD

![title](5b52fa1bab64414100001286.png)
切勿忘记`use` 中间件

重启完访问一下试试
![title](5b52fe45ab6441410000130e.png)

## AOP 切面
新建切面类`app\Module\Test\Aspect\TestAspect.php`
```php
    <?php
    
    namespace App\Module\Test\Aspect;
    
    use App\Module\Test\Collector\TestCollector;
    use Swoft\Aop\ProceedingJoinPoint;
    use Swoft\Bean\Annotation\Around;
    use Swoft\Bean\Annotation\Aspect;
    use Swoft\Bean\Annotation\PointAnnotation;
    use App\Module\Test\Annotations\Test;
    use Swoft\Core\RequestContext;
    
    /**
     * 测试切面类
     * @Aspect()
     * @PointAnnotation(
     *      include={
     *          Test::class
     *      }
     *  )
     * @package App\Module\Test\Aspect
     */
    class TestAspect
    {
        /**
         * @Around()
         * @param ProceedingJoinPoint $proceedingJoinPoint
         * @return mixed
         */
        public function around(ProceedingJoinPoint $proceedingJoinPoint)
        {
            $controllerClass  = RequestContext::getContextDataByKey('controllerClass');
            $controllerAction = RequestContext::getContextDataByKey('controllerAction');
    
            $collector = TestCollector::getCollector();
    
            /* @var Test $test*/
            $test = $collector[$controllerClass][$controllerAction];
    
            echo $test->getName(), "around-before\n";
            $result = $proceedingJoinPoint->proceed();
            echo $test->getName(), "around-after\n";
            return $result;
        }
    }
```

> `@Aspect()` 声明这是一个切面类
>
> `@PointAnnotation` 设置了注解切入点
>
> `@Before()` 设置了通知点

其他设置可以参考Swoft文档 [AOP章节](https://doc.swoft.org/master/zh-CN/core/aop/overview.html);

还需要改一下注解解析类`TestParser`

在`parser` 方法中加入`Collector::$methodAnnotations[$className][$methodName][] = get_class($objectAnnotation);`
![title](5b52faa7ab64414100001297.png)

这一步的作用是将使用注解的方法加入到允许切入的数组, 只有在`Collector::$methodAnnotations`变量中存在的方法才会调用切面

重启程序请求一下
![title](5b52fddfab644141000012f7.png)
## 控制器
当然也可以在控制器用
![title](5b53009eab6441410000133c.png)
![title](5b5300beab644142f500133a.png)

个人还是推荐用切面, 控制器里只需要加入注解即可