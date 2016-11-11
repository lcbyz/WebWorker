WebWorker
========

基于Workerman (http://www.workerman.net/) 实现的自带http server的web开发框架，用于开发高性能的api应用，例如app接口服务端等。

特性
========
* 天生继承workerman所拥有的特性
* 只实现了简单路由功能的小巧框架,便于开发者使用和扩展.demo1中只是目录示例，开发者可自行定义自己的应用目录结构
* 相比php-fpm或mod_php的方式性能有几十倍左右的提升
* 自带简单的单例redis操作类和单例mysqli操作类(支持自动重连)
* 可设置自动加载目录加载目录下的所有php文件(仅一级不支持递归)
* 自定义404响应
* 只支持php7
* 支持http协议1.1和1.0的长连接和短连接
* 集成了workerman-statistics项目，可以监控服务情况
* 支持中间件

框架由来
========
大家经常说php性能差，其实主要是在php-fpm或mod_php方式下的差，而php语言本身是不错的，尤其在未来加入JIT之后，性能会越来越好的。面对新兴的语言和开发方式，个人认为php应该抛弃php-fpm或mod_php的开发方式了，以主流的守护进程的方式来开发，这样的方式性能会比php-fpm或mod_php有几十倍左右的提升.

测试对比(代码在benchmark目录中)
========
测试主机是4核8G阿里云主机,redis也安装在该主机上,ab测试也在同一主机上，默认配置，没做系统优化.go版本是1.7.1,node是4.4.7,php是7.0.11,只所以没做fpm或mod_php的对比是因为他们太差了，也就二百左右qps而已.

go的压力测试
--------
![image](https://github.com/xtgxiso/WebWorker/blob/master/benchmark/go/go.ab.png)

node的压力测试
--------
![image](https://github.com/xtgxiso/WebWorker/blob/master/benchmark/node/node.ab.png)

php的压力测试
--------
![image](https://github.com/xtgxiso/WebWorker/blob/master/benchmark/php/php.ab.png)


快速开始
======
demo.php
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http;
use WebWorker\Libs\Mredis;
use WebWorker\Libs\Mdb;
use WebWorker\Libs\Mmysqli;

//判断系统
if (strtoupper(substr(PHP_OS, 0, 3)) === 'WIN') {
    require_once __DIR__.'/workerman-for-win/Autoloader.php';
}else {
    require_once __DIR__.'/workerman/Autoloader.php';
    // 检查扩展
    if(!extension_loaded('pcntl'))
    {
        exit("Please install pcntl extension. See http://doc3.workerman.net/install/install.html\n");
    }

    if(!extension_loaded('posix'))
    {
        exit("Please install posix extension. See http://doc3.workerman.net/install/install.html\n");
    }
}

$app = new WebWorker\App("http://0.0.0.0:1215");

//进程数
$app->count = 4;

//自动加载目录--会加载目录下的所有php文件
$app->autoload = array();

//应用级中间件--对所有访问启用ip限制访问
$app->AddFunc("/",function() use($app){
    if ( $_SERVER['REMOTE_ADDR'] != '127.0.0.1' ) {
        $app->ServerHtml("禁止访问");
        return true;//返回ture,中断执行后面的路由或中间件，直接返回给浏览器
    }   
});

//注册路由hello
$app->HandleFunc("/hello",function() use($app){
    $app->ServerHtml("Hello World WorkerMan WebWorker!");
});

//注册路由json
$app->HandleFunc("/json",function() use($app){
     //以json格式响应
     $app->ServerJson(array("name"=>"WebWorker"));
});

//注册路由input
$app->HandleFunc("/input",function() use($app){
    //获取body
     $body = $GLOBALS['HTTP_RAW_POST_DATA'];
     $app->ServerHtml($body);
});

$config = array();
$config["redis"]["host"] = "127.0.0.1";
$config["redis"]["port"] = 6379;
$config["redis"]["password"] = "123456";
$config["redis"]["db"] = 1;
$config["db"]["host"] = "127.0.0.1";
$config["db"]["user"] = "root";
$config["db"]["password"] = "123456";
$config["db"]["db"] = "test";
$config["db"]["port"] = 3306;
$config["db"]["charset"] = "utf8";

//redis示例
$app->HandleFunc("/redis",function() use($app,$config){
     $redis = Mredis::getInstance($config["redis"]);
     $app->ServerHtml($redis->get("xtgxiso"));
});

//mysql示例
$app->HandleFunc("/mysql",function() use($app,$config){
     $db = Mdb::getInstance($config["db"]);
     $list = $db->query("select * from test")->fetch_all(MYSQLI_ASSOC);
     $app->ServerJson($list);
});

//自定义404
$app->on404  = function() use($app){
    $app->ServerHtml("我的404");
};

// 如果不是在根目录启动，则运行runAll方法
if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}

// Run worker
Worker::runAll();
```


技术交流QQ群
========
517297682
