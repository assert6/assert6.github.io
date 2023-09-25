---
title: 【Hyperf 入门到精通】协程 04
tags: [Hyperf, 入门到精通]
date: 2023-09-15
---
# 子协程的适用场景
Swoole/Swow 对比Python 及Nodejs 协程, 最大特点是可以[无侵入](https://wiki.swoole.com/#/runtime), 润物细无声

甚至绝大多数代码可以无缝由Laravel 复制到Hyperf, 直接获得几个数量级的性能提升

使用子协程的缺点:

1. 事务性无法保证
2. 顺序性无法保证
3. 理解成本高(需要明确知道**哪些**会触发协程切换)
4. 对端压力大

子协程的优点:
$time = x_1+x_2+x_3+....x_n$
$time = max(x_1,x_2,x_3,...x_n)$

1. 提高响应速度
2. 提高响应速度
3. 提高响应速度

综上所述, 在哪些场景推荐使用子协程呢?
在不需要事务性保证、顺序性没有影响、对子协程有一定理解、对端可以承受大并发、急需提高响应速度的场景中

- 爬虫
-


```php

use Hyperf\Engine\Coroutine;

Coroutine::run(function () {
    echo "1";
    Coroutine::create(function () {
        echo "2";
        sleep(1); // 触发协程调度
        echo "3";
    });
    Coroutine::create(function () {
        echo "4";
    });
});
\Swow\Sync\waitAll();
```

**非必要不使用子协程**

## **TODO 手动使用子协程**
