---
title: 【生命周期】Hyperf 入门到精通03
tags: [Hyperf]
date: 2023-09-15
---
# 生命周期与变量作用范围
Laravel 生命周期是由FPM 决定的, 同样Hyperf 生命周期是由[Swoole 生命周期](https://wiki.swoole.com/wiki/page/354.html)决定的

以HTTP 服务为例, 通过代码解释一下生命周期, 如果使用不当, 会造成内存泄漏、数据混淆, 引发生产事故

- 全局生命周期(进程生命周期)
- 请求生命周期
- ~~协程生命周期~~

```php

require_once __DIR__ . '/vendor/autoload.php';

use Hyperf\Engine\Http\Server;
use Hyperf\HttpMessage\Server\Response;
use Psr\Http\Message\ServerRequestInterface;
use Swow\Psr7\Server\ServerConnection;

$a = 0;

$server = new Server();
$server->bind('0.0.0.0', 9502);
$server->handle(function (ServerRequestInterface $request, ServerConnection $connection) {
    $b = 0;
    $response = new Response();
    $connection->sendHttpResponse($response->withContent('Hello World'))->close();
});
$server->start();

```

全局生命周期, 内存里一直存在, 程序结束才会释放

- start() 前赋值的变量
- 类静态属性
- 超全局变量

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';

use Hyperf\Engine\Http\Server;
use Hyperf\HttpMessage\Server\Response;
use Psr\Http\Message\ServerRequestInterface;
use Swow\Psr7\Server\ServerConnection;

class Count
{
    public static int $count = 0;
}

$count = 0;
$globalCount = 0;

$_GET['count'] = 0;

$server = new Server();
$server->bind('0.0.0.0', 9502);
$server->handle(function (ServerRequestInterface $request, ServerConnection $connection) use (&$count) {
    global $globalCount;
  
    // start() 前声明的变量
    $globalCount++;
    $count++;
  
    // 类静态变量
    Count::$count++;
  
    // 超全局变量
    $_GET['count']++;

    echo "Global \$count: {$globalCount}\n";
    echo "Global \$globalCount: {$globalCount}\n";
    echo "Global Count::count " . Count::$count . "\n";
    echo "Global \$_GET {$_GET['count']} \n";

    $connection->sendHttpResponse((new Response())->withContent('Hello World'))->close();
});
$server->start();

```

代码生命周期小于[变量作用范围](https://www.php.net/manual/zh/language.variables.scope.php)才可能出现内存泄漏
比如**请求**生命周期内, 操作**全局**变量
```php

require_once __DIR__ . '/vendor/autoload.php';

use Hyperf\Engine\Http\Server;
use Hyperf\HttpMessage\Server\Response;
use Psr\Http\Message\ServerRequestInterface;
use Swow\Psr7\Server\ServerConnection;

$globalCount = [];

$server = new Server();
$server->bind('0.0.0.0', 9502);
$server->handle(function (ServerRequestInterface $request, ServerConnection $connection) {
    global $globalCount;
    $globalCount[] = $request;
    $connection->sendHttpResponse((new Response())->withContent('Hello World'))->close();
});
$server->start();

```
注意, 上述代码中的`$request`本身是请求生命周期, 但是赋值给全局生命周期的变量, 导致变量作用范围逃逸

举个常用的例子
```php

namespace App\Controller;

use Hyperf\HttpServer\Annotation\Controller;
use Hyperf\HttpServer\Annotation\GetMapping;

#[Controller]
class IndexController extends AbstractController
{
    public int $count = 0;

    #[GetMapping("/add")]
    public function add()
    {
        echo $this->count++, PHP_EOL;
        sleep(1);
        return $this->count;
    }
}

```
为什么以上代码会出现数据混淆?

其实`$count` 作用范围隶属于`IndexController` 的
`IndexController` 从`$container->$resolvedEntries` 获取
`$container` 在`Swoole::start()` 前声明, 所以是全局生命周期变量
`ApplicationContext::getContainer()` 只是方便使用, 并不是根本原因
