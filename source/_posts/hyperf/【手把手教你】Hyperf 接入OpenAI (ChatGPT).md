---
title: 【手把手教你】Hyperf 接入OpenAI ChatGPT
tags: [Hyperf, ChatGPT, OpenAI]
date: 2023-03-28
---

# OpenAI 是什么
OpenAI是一家致力于人工智能研究和发展的公司，其目标是创建普适人工智能，能够为人类创造更加繁荣、安全和可持续的未来。

为了实现这一目标，OpenAI不仅开展基础研究，还提供各种人工智能相关的工具和服务，包括API接口，以方便开发者快速应用最新的人工智能技术。
## OpenAI API
OpenAI API是一种面向开发者的人工智能服务，提供各种语言模型和自然语言处理工具。开发者可以使用OpenAI API来创建智能应用程序，例如聊天机器人、文本摘要器和情感分析器等。

目前，OpenAI API提供了以下几种模型：

- GPT-3：目前最大的语言模型，可生成高质量的自然语言文本。
- DALL-E：一种基于GPT-3的图像生成模型，可以根据文本输入生成独特的图像。
- CLIP：一种视觉模型，可以通过图片识别标签和描述，用于图像分类、搜索和相似度匹配。
- Codex：一个基于GPT-3的代码生成模型，可以通过简短的自然语言指令生成代码。
- Triton：一种用于推荐系统和个性化推荐的模型。
使用OpenAI API需要先注册一个开发者账户并创建API密钥。注册成功后，开发者可以使用API密钥来调用OpenAI API提供的各种服务。
## 接入OpenAI API
以下是接入OpenAI API的一般步骤：

1. 注册开发者账户并创建API密钥。在OpenAI网站上注册并创建一个API密钥，用于调用API服务。

2. 安装OpenAI API SDK。OpenAI提供了各种语言的SDK，包括Python、JavaScript、Java、Ruby等，开发者可以选择适合自己的SDK进行安装。

3. 调用API服务。使用SDK提供的API接口和密钥，开发者可以调用OpenAI API提供的各种服务。


>>> **注意, 以上全部是由OpenAI ChatGPT生成**

# Hyperf 接入OpenAI

接入OpenAI 的步骤非常简单, 上面ChatGPT 也已经给出了大概步骤, 下面举例展示一下, 重点在于类似ChatGPT 逐字回复的`EventStream` 的实现.


## 生成API密钥
我们需要在OpenAI官网注册一个账号, 然后生成一个[API密钥](https://platform.openai.com/account/api-keys), 保存好后面会用到, 注册账号的教程非常多, 这里就不再赘述了.

## 安装OpenAI SDK
OpenAI 官方提供了[PHP SDK](https://github.com/openai-php/client), 但要求PHP 8.1+ 版本, 满足要求的话可以直接安装使用
```bash
composer require openai-php/client
```
或者基于Hyperf 封装的[Client组件](https://github.com/friendsofhyperf/openai-client)
```bash
composer require friendsofhyperf/openai-client
```

SDK 代码非常清晰, 这里以官方SDK为例, 创建Client 配置参数即可请求OpenAI, 各个参数的含义可以参考[官方文档](https://platform.openai.com/docs/api-reference/completions/create)
```php
    public function index()
    {
        $client = OpenAI::client('替换第一步生成的API密钥');

        $response = $client->completions()->create([
            'model' => 'text-davinci-003',
            'prompt' => 'Hyperf 是什么?',
            'max_tokens' => 2048,
        ]);
        return $response->toArray();
    }
```
启动Hyperf 后调用一下, 就可以看到OpenAI 的回复了
![响应](./img.png)

## 逐字回复 EventStream
完成上面的步骤后, 会发现等待响应大概6秒左右特别久, 是因为我们同步阻塞等待OpenAI 所有响应, 在全部发送给客户端,  
而ChatGPT 采用了`EventStream`可以异步响应逐字回复, 所以在等待回复时可以看到ChatGPT 正在处理请求, 这样体验会更好一些.
同样Hyperf [最新版本](https://github.com/hyperf/engine/pull/16)也支持了`EventStream`, 可以非常方便的实现逐字回复.
稍微修改下上面的代码
```php
    public function index()
    {
        $client = OpenAI::client('替换第一步生成的API密钥');

        // 注意这里改为了createStreamed 方法
        $result = $client->completions()->createStreamed([
            'model' => 'text-davinci-003',
            'prompt' => 'Hyperf 是什么?',
            'max_tokens' => 2048,
        ]);
        
        // 如果报错Argument #1 ($connection) must be of type Hyperf\Engine\Contract\Http\Writable
        // 或者没有\Hyperf\Engine\Http\EventStream
        // 请更新engine(v1.10.0或v2.8.0)、engine-contract(v1.7.0)、hyperf/http-server(v3.0.14)
        $response = ApplicationContext::getContainer()->get(\Hyperf\HttpServer\Contract\ResponseInterface::class);
        
        $eventStream = new \Hyperf\Engine\Http\EventStream($response->getConnection());
        foreach ($result as $stream) {
            $eventStream->write($stream['choices'][0]['text']);
        }
        $eventStream->end();
    }
```
注意代码改动了三处, 其中第一处是`create` 方法改为`createStreamed` 方法, 第二处是去掉了返回值`return $response->toArray()`, 第三处是使用`EventStream` 逐字响应.
启动Hyperf 后调用一下, 就可以看到OpenAI 的回复了, 并且可以看到逐字响应的效果:

![响应](./eventStream.gif)

PS: [mozilla EventStream 格式](https://developer.mozilla.org/zh-CN/docs/Web/API/Server-sent_events/Using_server-sent_events#%E4%BA%8B%E4%BB%B6%E6%B5%81%E6%A0%BC%E5%BC%8F) 
