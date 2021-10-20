
# 基于PhalApi2的Redis拓展

![](http://webtools.qiniudn.com/master-LOGO-20150410_50.jpg)

## 1、前言

Redis在PHP开发中运用场景已经无处不在,小到简单缓存大到数据库或消息队列都可以使用Redis来进行实现,基于PhalApi2的出世,PhalApi2-Redis也紧接着进行了本次适配来提供更好的开发体验,PhalApi2-Redis提供相对于原生PhalApi2-RedisCache缓存更强大的Redis操作以及完善的封装机制,帮助开发者更好的使用Redis低成本的来解决实际的业务问题.

附上:

官网地址:[http://www.phalapi.net/](http://www.phalapi.net/ "PhalApi官网")

项目GitHub地址:[https://github.com/wenzhenxi/phalapi2-redis](https://github.com/wenzhenxi/phalapi2-redis "项目Git地址")

项目码云地址  : [https://gitee.com/wenzhenxi/phalapi2-redis](https://gitee.com/wenzhenxi/phalapi2-redis "项目码云地址")

## 2、安装配置Redis

基于Liunx强烈推荐使用oneinstack在配置php 和 Redis同事会将依赖打包好:

**oneinstack**:[https://oneinstack.com/](https://oneinstack.com/ "oneinstack")

手动安装redis网上有很多教程这里不再提及,主要注意一下配置文件:

```
databases 100                      #redis库的最大数量默认16推荐修改100
requirepass phalapi                #连接此redis的连接密码默认无密码推荐设置
```

手动安装php-redis依赖如下:

```
//下载phpredis解压安装
wget https://github.com/nicolasff/phpredis/archive/master.zip
unzip master.zip -d phpredis
cd phpredis/phpredis-master
phpize
./configure
make && make install
//在php.ini中注册phpredis
extension = redis.so
```
此后可以在phpinfo()中看到redis即可


## 3、安装PhalApi2-Redis：项目根目录composer.json添加

```
{
    "require": {
        "phalapi/redis": "2.0.*"
    }
}
```

+ 执行composer update更新

## 4、惰性加载Redis配置：**/config/di.php** 

 ```
 // 惰性加载Redis
 $di->redis = function () {
     return new \Xuepengdong\Phalapiredis\Lite(\PhalApi\DI()->config->get("app.redis.servers"));
 };
 ```



## 5、配置redis账号密码：/config/app.php
```
    /**
     * 扩展类库 - Redis扩展
     */
    'redis' => array(
        //Redis链接配置项
        'servers'  => array(
            'host'   => '127.0.0.1',        //Redis服务器地址
            'port'   => '6379',             //Redis端口号
            'prefix' => 'PhalApi_',         //Redis-key前缀
            'auth'   => 'phalapi',          //Redis链接密码
        ),
        // Redis分库对应关系操作时直接使用名称无需使用数字来切换Redis库
        'DB'       => array(
            'developers' => 1,
            'user'       => 2,
            'code'       => 3,
        ),
        //使用阻塞式读取队列时的等待时间单位/秒
        'blocking' => 5,
    ),

```

## 6、入门使用

+ **永久键值队**

  ```
  // 存入永久的键值队
  \PhalApi\DI()->redis->set_forever(键名,值,库名);
  
  // 获取永久的键值队
  \PhalApi\DI()->redis->get_forever(键名, 库名);
  ```

+ **有时效性键值队**

  ```
  // 存入一个有时效性的键值队,默认600秒
  \PhalApi\DI()->redis->set_Time(键名,值,有效时间,库名);
  
  // 获取一个有时效性的键值队
  \PhalApi\DI()->redis->get_Time(键名, 库名);
  ```

+ **写入队列**

  ```
  // 插入集合：写入队列左边 并根据名称自动切换库
  \PhalApi\DI()->redis->set_Lpush(队列键名,值, 库名);
  
  //插入集合：写入队列右边 并根据名称自动切换库
  \PhalApi\DI()->redis->set_rPush(队列键名, 值, 库名);
  
  // 读取队列右边 如果没有读取到阻塞一定时间(阻塞时间或读取配置文件blocking的值)
  \PhalApi\DI()->redis->get_Brpop(队列键名,值, 库名);
  ```

+ **获取队列**

  ```
  // 读取队列左边
  \PhalApi\DI()->redis->get_lpop(队列键名, 库名);
  
  // 读取队列右边
  \PhalApi\DI()->redis->get_rPop(队列键名, 库名);
  ```

+ **获取指定位置**

  ```
  //返回名称为key的list中start至end之间的元素（end为 -1 ，返回所有）
  \PhalApi\DI()->redis->get_lRange(队列键名, $start, $end);
  ```

+ **截取指定位置**

  ```
  //截取名称为key的list，保留start至end之间的元素,end为 -1 ，返回所有
   \PhalApi\DI()->redis->get_lTrim(键名,$start, $end, 库名);
  ```

+ **获取key的生存时间**

  ```
  \PhalApi\DI()->redis->get_lTrim(键名, 库名);
  ```

+ **判断key是否存在**

  ```
  \PhalApi\DI()->redis->get_exists(键名, 库名);
  ```

+ **其他**

  ```
  // 删除一个键值队适用于所有
  \PhalApi\DI()->redis->del(键名, 库名);
  // 自动增长
  \PhalApi\DI()->redis->get_incr(键名, 库名);
  // 切换DB并且获得操作实例
  \PhalApi\DI()->redis->get_redis(键名, 库名);
  ```


**如果大家有更好的建议可以私聊或加入到PhalApi大家庭中前来一同维护PhalApi**
**注:笔者能力有限有说的不对的地方希望大家能够指出,也希望多多交流!**
