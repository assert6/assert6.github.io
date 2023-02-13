---
title: 【手把手教你】用Swoft 搭建微服务(TCP RPC)2
tags: [Swoft, 手把手教你]
date: 2018-06-07
---

上一篇讲了[如何用Swoft 搭建RPC 服务](/2018-06-06/swoft/用Swoft%20搭建微服务(TCP%20RPC))

本篇介绍如何使用微服务

## 微服务流程
首先讲一下微服务的流程

弄清楚流程, 开发起来就行云流水

这是官方给出的目录结构

```bash
    app/
      - Lib/          // 服务的公共接口定义目录，里面通常只有php接口类
      - Pool/         // 服务池配置，里面可以配置不同服务的连接池，参考里面的 UserServicePool
      - Services/     // 具体的服务接口实现类，里面的类通常实现了 Lib 中定义的接口
```

> 当然在多个服务中使用时， 要将lib库 app/Lib移到一个公共的git仓库里，然后各个服务通过 composer 来获取使用

这里有3个目录

`Lib` 是接口, 也是客户端和服务端必须的, 用来规定数据结构, 一般由服务端提供

服务端还需要实现`Services`, 也就是相当于`Model` 层

客户端可以通过配置`Pool` 来调用对应的`Services`, 如同调用自身的`Model` 一样方便

整个流程不是很难理解, 但是官方把三个目录放在一起, 并且没有详细说明, 导致很多同学搞不清楚

下面我们来写一个例子试一下吧
## 定义一个接口

> 接口定义和普通接口完全一致，唯一不一样的是:
- 需要在类注释上定义类似 `deferGetUser` 方法，对应类里面方法 `getUser` 且首字母大写。这种 `defer*` 方法，一般用于业务延迟收包和并发使用。
- 这些方法是不需要实现的，仅用于提供IDE提示。内部调用逻辑由框架帮你完成


在`app/Lib` 新建接口文件`MemberInterface`

定义一个`getMemberByID`方法, 并且根据官方提示, 在接口注释里定义一个`deferGetMemberByID` 方法

```php
    <?php
    
    namespace App\Lib;
    
    use Swoft\Core\ResultInterface;
    
    /**
     * @method ResultInterface deferGetMemberByID(int $id)
     */
    interface MemberInterface
    {
        public function getMemberByID(int $id);
    }
```

这样就可以了, 新建的**每个方法**都要在接口注释里加一个同名的`defer*`方法, 用来做延迟收包方法

把接口文件复制到`swoft-http`的`app/Lib`里, 且之后接口文件若有变动, 都要报证相同

## 服务端实现Services
接口定义好了, 下面我们在Services 里实现这个接口

在`app/Services` 新建类文件`MemberService`, 实现`MemberInterface` 接口, 并继承其注释

```php
    <?php
    
    namespace App\Services;
    
    use App\Lib\MemberInterface;
    use Swoft\Core\ResultInterface;
    
    /**
     * @method ResultInterface deferGetMemberByID(int $id)
     * @Service()
     */
    class MemberService implements MemberInterface
    {
        public function getMemberByID(int $id)
        {
            // TODO: Implement getMemberByID() method.
            return [
                "小红",
                "小红1",
                "小红2",
                "小红3",
                "小红4",
            ][$id] ?? "查无此人";
        }
    
    }
```

这里需要注意:
> 注解里的`@Service()`默认为0, 用来区分接口的不同版本

也就是你可以建多个类来实现这个接口, `@Service(version="1.0.1")` 版本号不同即可

## 客户端配置Pool
服务端的`Service`实现就是这么简单

反倒是客户端的连接池实现有些麻烦

客户端连接池需要搞清两个问题:

1. 连谁?
2. 连什么?

`app/Pool/Config` 是连接池配置文件, 来解决第一个问题

同一个连接我们可以用同一个连接池, 刚才在服务端只是新增了一个接口, 并不是新建了一个RPC 服务, 所以这里不需要改动(只是我懒)

下面来解决第二个问题, 连什么?

在`app/Controller` 里新建一个控制器`MemberController`

```php
    <?php
    
    namespace App\Controllers;
    
    use App\Lib\MemberInterface;
    use Swoft\Http\Server\Bean\Annotation\Controller;
    use Swoft\Http\Server\Bean\Annotation\RequestMapping;
    use Swoft\Http\Server\Bean\Annotation\RequestMethod;
    use Swoft\Rpc\Client\Bean\Annotation\Reference;
    
    /**
     * @Controller()
     */
    class MemberController
    {
        /**
         * @Reference(name="user", version="0")
         * @var MemberInterface
         */
        private $memberService;
    
        /**
         * @RequestMapping(route="/member/{mid}", method={RequestMethod::GET})
         * @return array
         */
        public function getMemberByID(int $mid)
        {
            $result  = $this->memberService->getMemberByID($mid);
            return [$result];
        }
    }
```

**注意成员**`memberService` 的注解

1. `@Reference(name="user")` 用来指定连接谁, `user` 是别名, 在`app/Lib/UserServicePool.php` 的`@Pool`注解里命名
2. `@var MemberInterface` 用来指定连接什么, 也就是哪个接口
3. `version="0"` 用来指定该接口的某个版本, 在服务端`Services` 的注解里声明


## 测试
打开连接`http://127.0.0.1:9501/member/2`
![title](https://leanote.com/api/file/getImage?fileId=5b17a847ab6441649600174a)
成功!