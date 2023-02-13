---
title: 【手把手教你】Swoft 多库连接池配置
tags: [Swoft, 手把手教你]
date: 2018-09-27
---
这里的多库是指数据不同的两个主库

并不是数据相同的主从库

多库的配置非常简单, 只涉及到配置文件和两个类`PoolConfig` `Pool`

# 配置文件
编辑配置文件 `config/properties/db.php`
新增数据库配置
```php
    'dev' => [
        'master' => [
            'name' => 'master2',
            'uri' => [
                '192.168.1.135:3306/test?user=tender&password=123456&charset=utf8',
            ],
        ],
        'slave' => [
            'name' => 'slave3',
            'uri' => [
                '192.168.1.135:3306/test?user=tender&password=123456&charset=utf8',
            ],
        ],
    ],
```

加完如下
![title](https://leanote.com/api/file/getImage?fileId=5baee7bbab644173f20008d1)

> 默认的`master`和`slave`应为`default`下的配置, 为了兼容历史版本

# 连接池配置类
新建`app/Pool/Config/DevPoolConfig.php`
```php
    <?php
    
    namespace App\Pool\Config;
    
    use Swoft\Bean\Annotation\Bean;
    use Swoft\Bean\Annotation\Value;
    use Swoft\Db\Pool\Config\DbPoolProperties;
    
    /**
     * DevPoolConfig
     * @Bean()
     */
    class DevPoolConfig extends DbPoolProperties
    {
        /**
         * @Value(name="${config.db.dev.master.name}")
         * @var string
         */
        protected $name = '';
    
        /**
         * @Value(name="${config.db.dev.master.uri}")
         * @var array
         */
        protected $uri = [];
    }
```

连接池配置类只是把配置文件由数组改写为类, 有其他配置需要全部加进去。 这里只改了name 和uri

# 连接池
新建`app/Pool/DevPool.php`
```php
    <?php
    
    namespace App\Pool;
    
    use App\Pool\Config\DevPoolConfig;
    use Swoft\Bean\Annotation\Inject;
    use Swoft\Bean\Annotation\Pool;
    use Swoft\Db\Pool\DbPool;
    
    /**
     * DevPool
     *
     * @Pool("dev.master")
     */
    class DevPool extends DbPool
    {
        /**
         * @Inject()
         * @var DevPoolConfig
         */
        public $poolConfig;
    }
```

更简单, 指定该连接池使用哪个配置
注意下`@Pool` 注解连接池别名, 要带`.master`或者`.salve`

# 使用连接池
有两种方法使用连接池
第一种是在实体类`@Entity` 注解里配置`instance`字段:
```php
    /**
     * 用户实体
     * @Entity(instance="dev.master")
     * @Table(name="user")
     */
    class User extends Model
    {
    ...
```

这样调用该实体就会走`dev`的配置

第二种是在`Db::query()` 查询时指定连接池, 直接在方法第三个参数配置即可:
```php
    Db::query("show databases;",[], 'dev.master')->getResult();
```