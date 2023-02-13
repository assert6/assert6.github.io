---
title: 【手把手教你】使用TCP Client 调用Swoft RPC 服务
tags: [Swoft, 手把手教你]
date: 2018-06-08
---

前两篇教大家[如何搭建和使用Swoft RPC 服务](/2018-06-06/swoft/用Swoft%20搭建微服务), 这里简单说一下用其他TCP 客户端如何调用Swoft 的RPC 服务

这里用`swoole_client` 举个例子
```php

    $client = new \swoole_client(SWOOLE_SOCK_TCP);

    if (! $client->connect('192.168.1.214', 8099, 0.5))
    {
        return "connect failed. Error: {$client->errCode}\n";
    }

    $client->send(json_encode([
        "interface" => "App\Lib\MemberInterface",
        "version" => "0",
        "method" => "getMemberByID",
        "params" => [$mid],
    ])."\r\n");// 旧版不需要追加\r\n

    $result = $client->recv();
    $client->close();
    return $result;
```

非常的简单易用, 但是需要注意一点:

`interface` 参数是**服务端**带命名空间的类名, 跟客户端没任何关系, 完全取决于服务端把RPC 接口类放在哪儿...
