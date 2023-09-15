---
title: 【注解】Hyperf 入门到精通01
tags: [Hyperf]
date: 2023-09-15
---
# 注解 Annotation / Attribute
PHP8 之前注解称之为Annotation
使用`doctrine/annotations`组件, 通过特定文本格式`@Class`解析内容, 已经淘汰这里不展开
```php

use Hyperf\HttpServer\Annotation\Controller;

/**
 * @Controller(...)
 */
class IndexController
{
  
}
```

PHP8 之后注解称之为**Attribute**, PHP 官方提供, 使用方便, 性能优秀, 格式为`#[Class]`
```php

use Hyperf\HttpServer\Annotation\Controller;

#[Controller(prefix: '/index')]
class IndexController
{
  
}

# 反射获取属性
$reflectionClass = new ReflectionClass(IndexController::class);
$attributes = $reflectionClass->getAttributes();
foreach ($attributes as $attribute) {
    print_r([$attribute->getName(), $attribute->getArguments()]);
}
```
官方介绍:
> Attribute 提供了在代码中的声明上添加结构化、机器可读的元数据信息的能力：类、方法、函数、参数、属性和类常量可以是Attribute 的目标。然后可以使用反射 API在运行时检查Attribute 定义的元数据 。因此，Attribute 可以被认为是直接嵌入到代码中的配置语言。


注解可以简单理解为, 写死在代码(类、方法、函数、参数、属性和类常量)上的**配置项**

### 自定义一个Attribute
```php

use Attribute;

#[Attribute(Attribute::TARGET_CLASS_CONSTANT)]
class Message
{
    public function __construct(public string $text)
    {
    }
}
```

可以实现一个简易版[枚举类](https://hyperf.wiki/3.0/#/zh-cn/constants)
```php
use Message;

class ErrorCode
{
    #[Message('成功')]
    public const SUCCESS = 0;

    #[Message('系统参数错误')]
    public const ERROR = 1;

    public static function getMessage(int $code): string
    {
        $reflectionClass = new ReflectionClass(ErrorCode::class);
        $constants = $reflectionClass->getReflectionConstants();
        foreach ($constants as $constant) {
            if ($constant->getValue() === $code) {
                $attributes = $constant->getAttributes(Message::class);
                return $attributes[0]->newInstance()->text;
            }
        }
        return '';
    }
}
echo ErrorCode::getMessage(ErrorCode::SUCCESS);
```

注意, Hyperf 自带的一些注解, 功能强大, 比如:

1. Inject
2. Controller
3. Cacheable

但这是因为AOP 的强大, 并非注解强大, 后面介绍AOP