---
title: 【AOP】Hyperf 入门到精通02
tags: [Hyperf]
date: 2023-09-15
---
# AOP 使用入门及适用场景
AOP 与Middleware 非常相似, 当你熟练掌握Middleware 时, 相当于也掌握了AOP
我们先回顾下Middleware 调用流程:
![](https://hyperf.wiki/3.0/zh-cn/middleware/middleware.jpg)
可以通过配置或者注解使用Middleware
```php

Router::get('/', 'App\Controller\IndexController::index', ['middleware' => [CorsMiddleware::class]]);


#[Middleware(CorsMiddleware::class)]
class IndexController
{
  ...
}
```

Middleware 作为解耦神器, 非常普遍的应用在各个服务中
但是, 这么好用的功能仅能作用在Controller 上, 岂不是很可惜?

AOP 可以简单理解为, 可以**作用在任何Class** 的Middleware:
```php

use Hyperf\Di\Aop\ProceedingJoinPoint;

class A
{
    public function test()
    {
        echo 'this A'.PHP_EOL;
    }
}

class B
{
    public function test()
    {
        (new A())->test();
        echo 'this B'.PHP_EOL;
    }
}

(new B())->test();

class AOP
{
    public array $classes = [
        A::class,
        B::class,
    ];
  
    public function process(ProceedingJoinPoint $proceedingJoinPoint)
    {
        $className = $proceedingJoinPoint->className;
        $methodName = $proceedingJoinPoint->methodName;

        echo "{$className}::{$methodName} Before".PHP_EOL;

        # 相当于 $handler->handle($request);
        $result = $proceedingJoinPoint->process();

        echo "{$className}::{$methodName} After".PHP_EOL;
        return $result;
    }
}


// B::test Before
// A::test Before
// this A
// A::test After
// this B
// B::test After

```


AOP 使用场景有哪些? 可以思考Middleware 使用场景有哪些:

- Token 鉴权
- 签名/验签, 加密/解密
- 日志/Tracer
- CORS
- ....

以上场景抽象一下, 得到结论: Middleware 适用于**系统功能**

同理, AOP 也适用于以上场景, 以及各种系统功能

- Cacheable
- AsyncQueueMessage
- Retry
- Breaker
- Transaction
- RateLimit
- ...

以常用的事务举例
```php

use Hyperf\DbConnection\Db;

class A
{
    // 普通写法
    public function foo()
    {
        Db::beginTransaction();
        try{
            $model = Model::query()->where('id', 1)->first();
            $model->name = 'Hyperf';
            $model->save();
            Db::commit();
        } catch(\Throwable $e){
            Db::rollBack();
        }
        return $model;
    }

  
    // 闭包写法
    public function bar()
    {
        return Db::transaction(function () {
            $model = Model::query()->where('id', 1)->first();
            $model->name = 'Hyperf';
            $model->save();
            return $model;
        });
    }
}
```

AOP 搭配注解写法
```php

use Hyperf\DbConnection\Annotation\Transactional;

class A
{
    #[Transactional]
    public function test()
    {
        $model = Model::query()->where('id', 1)->first();
        $model->name = 'Hyperf';
        $model->save();
        return $model;
    }
}


// 讲解说明, 非实际代码
class TransactionAspect
{
    public array $annotations = [
        Transactional::class,
    ];

    public function process(ProceedingJoinPoint $proceedingJoinPoint)
    {
        return Db::transaction(function () use ($proceedingJoinPoint) {
                return $proceedingJoinPoint->process();
            }
        );
    }
}
```

单一场景下, 闭包写法与注解AOP 差别不大

但如果此时需要**加需求**, 易读性会截然不同:
```php

class A
{
    public function bar()
    {
        return Cache::remember(function () {
            Db::transaction(function () {
                $model = Model::query()->where('id', 1)->first();
                $model->name = 'Hyperf';
                $model->save();
                return $model;
            });
        });
    }
}
```
```php
class A
{
    #[Cacheable]
    #[Transactional]
    public function test()
    {
        $model = Model::query()->where('id', 1)->first();
        $model->name = 'Hyperf';
        $model->save();
        return $model;
    }
}
```

AOP 避免回调地狱, 举例, 非正常业务场景
```php
class A
{
    #[Transactional]
    #[Cacheable]
    #[AtomicLock]
    #[Retry]
    #[RateLimit(create: 1, capacity: 3)]
    public function test()
    {
        $model = Model::query()->where('id', 1)->first();
        $model->name = 'Hyperf';
        $model->save();
        return $model;
    }
}
```
