---
title: "tp6 & RabbitMQ企业最佳实践"
date: 2021-12-14T21:11:37+08:00
Description: ""
Tags: ["tp6", "rabbitmq", "mq"]
Categories: ["mq"]
DisableComments: false
---


> 代码包地址
> [GITEE](https://gitee.com/hzbskak/tp6_rbmq)

目录
@[TOC](消息队列中间件)

# 消息队列中间件-前言

消息队列中间件是分布式系统中重要的组件，主要解决应用耦合，异步消息，流量削锋等问题。实现高性能，高可用，可伸缩和最终一致性架构，是大型分布式系统不可缺少的中间件。

RabbitMQ——Rabbit Message Queue的简写，RabbitMQ 是一个由 Erlang 语言开发的AMQP（高级消息队列协议）的开源实现。

> RabbitMQ提供了可靠的消息机制、跟踪机制和灵活的消息路由，支持消息集群和分布式部署。

**适用于排队算法、秒杀活动、消息分发、异步处理、数据同步、处理耗时任务、CQRS等应用场景!**

**MQ使用场景**

---

> 1、异步处理

不用MQ时可能的处理流程：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513092713786.png)

> 代码包地址
> [GITEE](https://gitee.com/hzbskak/tp6_rbmq)
> [GITHUB](https://github.com/qnmdca/tp6_rbmq)

目录
@[TOC](消息队列中间件)

# 消息队列中间件-前言

消息队列中间件是分布式系统中重要的组件，主要解决应用耦合，异步消息，流量削锋等问题。实现高性能，高可用，可伸缩和最终一致性架构，是大型分布式系统不可缺少的中间件。

RabbitMQ——Rabbit Message Queue的简写，RabbitMQ 是一个由 Erlang 语言开发的AMQP（高级消息队列协议）的开源实现。

> RabbitMQ提供了可靠的消息机制、跟踪机制和灵活的消息路由，支持消息集群和分布式部署。

**适用于排队算法、秒杀活动、消息分发、异步处理、数据同步、处理耗时任务、CQRS等应用场景!**

**MQ使用场景**

---

> 1、异步处理

不用MQ时可能的处理流程：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513092713786.png)
使用MQ处理：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513092721659.png)

> 2、 流量削峰

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513092931133.png)

> 3、应用解耦

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513092943133.png)

# RabbitMQ安装

RabbitMQ 是 Erlang 语言写的，首先我们需要安装Erlang 环境
RabbitMQ 不同版本也对应着不同的Erlang 版本，进入官网：[RabbitMQ](https://www.rabbitmq.com/which-erlang.html)
查看对照表：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513093354483.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
**一、安装erlang**

---

1、进入package Cloud，这里会很详细的解释，安装过程中需要哪些和生成一些文件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513093449794.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513093457401.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
2、在命令行中执行脚本

```bash
curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash
```

3、安装

```bash
sudo yum install erlang-21.3.8.16-1.el7.x86_64
```

4、测试\
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513093553730.png)
**二、安装rabbitmq**

---

1、进入package Cloud，这里会很详细的解释，安装过程中需要哪些和生成一些文件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513093618664.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513093625407.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
2、在命令行中执行脚本

```bash
curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash
```

3、安装

```bash
sudo yum install rabbitmq-server-3.8.4-1.el7.noarch
```

4、启动

```bash
#启动服务
service rabbitmq-server start
#查看状态
service rabbitmq-server status
#设置开机自启动
chkconfig rabbitmq-server on
```

5、开启web管理界面，增加用户

```bash
#开启web管理界面
rabbitmq-plugins enable rabbitmq_management
#添加用户名和密码
abbitmqctl add_user zq 123456
#将用户admin设置为管理员
rabbitmqctl set_user_tags zq administrator
```

6、测试

在浏览器访问http://127.0.0.1:15672，登录 Web 管理界面
输入上面添加的用户名/密码:zq/123456
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513102531983.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513093812782.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)

# PHP安装rabbitmq、php-amqplib扩展

**1、安装依赖库rabbitmq-c**

---

```bash
yum install librabbitmq-devel.x86_64 -y  
wget http://pecl.php.net/get/amqp-1.10.0.tgz 
tar zxvf amqp-1.10.0.tgz  
cd amqp-1.10.0  
./configure --with-php-config=/usr/local/php/bin/php-config --with-amqp  
make  
make install  
vim /usr/local/php/etc/php.ini  
extension=amqp.so
```

**2、重启php-fpm**

---

```bash
#重启php-fpm
service php-fpm restart
#查看是否安装了amqp扩展
php -m
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513093859620.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
**3、安装php-amqplib扩展**

---

> 进入tp6项目的根目录，安装扩展

```bash
 composer require php-amqplib/php-amqplib
```

> 安装完成后会在vendor增加一个php-amqplib目录

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513093949708.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)

# RabbitMQ入门

> RabbitMQ 是一个消息代理：它接收并转发消息。

RabbitMQ 你可以把它当成一个邮箱，当你把你想发送的邮件投进邮箱时，你可以确定邮递员最终会把邮件送到你的收件人。

RabbitMQ 它接收、存储和转发二进制的数据–即消息(message)。

RabbitMQ中一些常使用术语：

- Publisher：生产者，消息的发送方。
- Connection：网络连接。
- Channel：信道，多路复用连接中的一条独立的双向数据流通道。
- Exchange：交换机（路由器），负责消息的路由到相应队列。
- Binding：队列与交换器间的关联绑定。消费者将关注的队列绑定到指定交换器上，以便Exchange能准确分发消息到指定队列。
- Queue：队列，消息的缓冲存储区。
- Virtual Host：虚拟主机，虚拟主机提供资源的逻辑分组和分离。包含连接，交换，队列，绑定，用户权限，策略等。
- Broker：消息队列的服务器实体。
- Consumer：消费者，消息的接收方。

RabbitMQ 消息结构：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513094308112.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)

**hello world**

现在我们来实现一个生产者负责发送5条消息，一个消费者负责接收消息并打印出来

**1、生产者**

---

生产者将会连接 RabbitMQ、发送5条消息、然后关闭连接。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513094445517.png)

```php
<?php
declare(strict_types=1);

namespace app\index\controller;

use app\BaseController;
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Exchange\AMQPExchangeType;
use PhpAmqpLib\Message\AMQPMessage;
use PhpAmqpLib\Wire\AMQPTable;

class Mq extends BaseController
{
    public function send()
    {
        // 队列名 消息队列载体，每个消息都会被投入到一个或多个队列
        $queue = 'hello';
        $routing = 'hello';

        // 建立链接
        $connection = new AMQPStreamConnection('127.0.0.1', 5672, 'zq', '123456', '/');
        // 获取信道
        $channel = $connection->channel();

        // 声明创建队列
        $channel->queue_declare($queue, false, false, false, false);

        for ($i = 0; $i < 5; ++$i) {
            sleep(1); // 休眠1s
            // 消息内容
            $messageBody = 'hello,zq now time:' . date('H:i:s');
            // 将我们需要的消息标记为持久化 - 通过设置AMQPMESSAGE的参数delivery_mode = AMQPMessage::DELIVERY_MODE_PERSISTENT
            $message = new AMQPMessage($messageBody, [
                'content-type' => 'text/plain',
                'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
            ]);
            // 发送消息
            $channel->basic_publish($message, '', $routing);
            echo 'send message:' . $i . "\n";
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();

        return 'send success';


    }


}
```

在浏览器访问http://tp6.ws/index/mq/send，发送5条消息

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513101156342.png)

登陆RabbitMQ Web 管理界面

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513101331377.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
**2、消费者**

---

消费者从 RabbitMQ 中获取消息，与发布消息的生产者不同，消费者一直运行以监听消息，有新消息就立即处理
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513102136439.png)
利用TP6自定义命令行实现消费者

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class Mq extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('mq')
            ->setDescription('the mq command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 队列名 消息队列载体，每个消息都会被投入到一个或多个队列
        $queue = 'hello';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'zq', '123456', '/');
        // 获取信道
        $channel = $connection->channel();

        // 声明创建队列
        $channel->queue_declare($queue, false, false, false, false);

        // 消息消费
        $channel->basic_consume($queue, '', false, true, false, false, function ($msg) use ($output) {
            $output->writeln('received' . $msg->body . PHP_EOL);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();

    }


}
```

在控制台启动消费者：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513102311805.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)

切换到RabbitMQ Web 管理界面，消息已经被消费：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513102342722.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)

# 工作队列（Work Queues）

工作队列：其主要的思想是避免立即执行资源密集型任务，并且阻塞进程等待任务完成。相反，我们让这类型任务稍后执行，我们将它封装为消息，并将其添加到任务队列中。在后台启动一个或者多个工作进程，读取工作队列中的任务，并且执行，当你运行多个工作进程时，他们共享工作队列中的任务。

> 这个思想在Web应用程序中特别有用，用于处理在短时间的HTTP请求中无法处理的复杂任务。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513102710939.png)
我们将之前的消费者app\command\mq.php做一些修改：它需要为消息体中的每个消息模拟耗时三秒的工作时间。

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class Mq extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('mq')
            ->setDescription('the mq command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 队列名 消息队列载体，每个消息都会被投入到一个或多个队列
        $queue = 'hello';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'zq', '123456', '/');
        // 获取信道
        $channel = $connection->channel();

        // 声明创建队列
        $channel->queue_declare($queue, false, false, false, false);

        // 公平调度，新的消息将发送到一个处于空闲的消费者
        $channel->basic_qos(null, 1, null);

        // 消息消费
        $channel->basic_consume($queue, '', false, true, false, false, function ($msg) use ($output) {
            // 模拟耗时
            sleep(3);
            $output->writeln('received' . $msg->body . PHP_EOL);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();

    }


}
```

接着我们看看两个消费者控制台输出的内容：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513103145608.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513103233238.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
**消息确认机制 (ack)**

---

想一下如果一个消费者在执行一个长时间的任务的过程中异常退出会怎样？在我们目前的代码下，一旦RabbitMQ向消费者发送了消息后，此条消息将立即从内存中删除。在这种情况下，如果你杀掉一个正在处理消息的工作进程，那么我们将丢失这条消息！

为了确保消息永远不会丢失，RabbitMQ支持消息确认机制。消费者发送一个确认信息告诉RabbitMQ已经收到消息并且处理完成，此时RabbitMQ就可以删除这条消息了。

如果消费者进程异常退出(其通道关闭，连接断开或者TCP连接丢失)，而不发送确认信息，RabbitMQ将会认为此进程当前处理的消息未处理完成，就会将此消息重新放回消息队列。如果此时有其他消费者在线，则会迅速将这条消息重新提供给另一个消费者。这样就可以确保没有消息丢失。即使有消费者进程异常退出的情况。

在默认情况下，消息确认机制是关闭的。现在是时候开启消息确认机制，将basic_consumer的第四个参数设置为false(true表示不开启消息确认)，并且工作进程处理完消息后发送确认消息。

```php
//开启消息确认
$channel->basic_consume($queue, '', false, false, false, false, function ($msg) use ($output)  {
    //模拟耗时
    sleep(3);
    $output->writeln(" Received " . $msg->body .  PHP_EOL);

    //确认消息处理完成
    $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);

});
```

**消息持久化**

---

我们已经学习了如何确保即使消费者异常退出后，任务也不会丢失。但是如果RabbitMQ服务器挂了，我们的任务任然会丢失。

当RabbitMQ退出或崩溃时，它会丢失队列和其中的消息，除非你告诉他要持久化这些信息。

`我们需要做两件事来确保消息不会丢失：将队列和消息标记为 durable(持久化)。`

首先，我们需要确保RabbitMQ不会丢失消息队列。我们需要将队列声明为持久化，只要将 queue_declare 的第三个参数设置为 true 就行了，设置为 true 的标志需要同时应用于生产者和消费者中：

```php
// 声明创建队列
$channel->queue_declare($queue, false, true, false, false);
```

到这一步，我们确信即使RabbitMQ重启，hello队列也不会丢失。现在我们需要将我们需要的消息标记为持久化，通过设置AMQPMessage 的参数 delivery_mode = AMQPMessage::DELIVERY_MODE_PERSISTENT:

```php
// 将我们需要的消息标记为持久化 - 通过设置AMQPMESSAGE的参数delivery_mode = AMQPMessage::DELIVERY_MODE_PERSISTENT
$message = new AMQPMessage($messageBody, [
    'content-type' => 'text/plain',
    'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
]);
```

**关于队列的大小**

---

如果所有的消费者进程都处于忙碌状态，你的队列可能会溢出。你应该留意一下这种情况，也许你可以增加更多的消费者进程，或者采取一些其他的策略。

# 发布/订阅（Publish/Subscribe）

在这一节中，我们会向多个消费者传递相同的信息。这种模式被称为“发布/订阅”。

**交换机（Exchanges）**

---

在上一篇教程中，我们通过工作队列发送和接收消息。现在是时候在Rabbit中引入完整的消息传递模式了。

我们快速回顾一下前面教程中介绍的内容：

- 生产者是发送消息的用户应用程序
- 队列是存储消息的缓冲区
- 消费者是接收消息的用户应用程序

RabbitMQ中消息传递模型的核心思想是，生产者从不将任何消息直接发送到队列。实际上，生产者甚至通常不知道是否将消息传递到了队列中，生产者只能将信息发送到交换机(exchange)。交换机一方面，它接收来自生产者的消息; 另一方面将接收到的消息推送到队列。

交换机准确知道接收到的消息应该如何处理，应该添加到特定的队列， 应该添加到许多的队列中，或者应该丢弃，其规则由交换类型定义。

`有几种交换类型可用：direct, topic, headers, fanout. 我们将学习第一个 - fanout交换机`

**直接交换机 (Fanout exchange)**

---

fanout 交换非常简单，它是将所有的消息广播到所有已知的队列。
让我们创建一个这种类型的交换器，并且将其称为logs：

```php
$channel->exchange_declare('logs', 'fanout', false, false, false);
```

**默认交换**

---

在本教程的前面部分，我们对交换没有任何了解，但是任然能够将消息发送到队列。这是可能的，因为我们使用的是默认交换，我们通过空字符串”“标识。
回顾一下我们之前发布的消息：

```php
# 在这里，我们使用默认交换
$channel->basic_publish($msg, '');
```

现在，我们可以将消息发布到我们命名的交换机：

```php
$channel->exchange_declare('logs', 'fanout', false, false, false);
$channel->basic_publish($msg, 'logs');
```

**绑定**

---

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513104408810.png)
我们已经创建了一个fanout交换机和一个队列。现在我们需要告诉交换机发送消息到我们的队列。交换机和队列之间的关系称为绑定。

```php
$channel->queue_bind($queue_name, 'logs');
```

**总结**

---

我们现在把消息发布到我们的logs交换机。

```php
<?php
declare (strict_types=1);

namespace app\controller;

use app\BaseController;
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

class PubSub extends BaseController
{
    public function send()
    {
        // 交换机名
        $exchange = 'logs';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'fanout', false, false, false);

        for ($i = 0; $i < 5; ++$i) {
            sleep(1); // 休眠1s
            // 消息内容
            $messageBody = 'heelo,zq now time:' . date('h:i:s');
            // 将我们需要的消息标记为持久化
            $message = new AMQPMessage($messageBody, [
                'content_type' => 'text/plain',
                'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
            ]);

            // 发送消息
            $channel->basic_publish($message, $exchange);
            echo 'send message:' . $i . "\n";
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
        return 'send success';
    }
}
```

在浏览器访问http://tp6rbmq.ws/pubsub/send，发送5条消息
登陆RabbitMQ Web 管理界面，增加了一个logs交换机：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513113552366.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
然后我们看一下消费端：

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class PubSub extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('pubsub')
            ->setDescription('the pubsub command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 交换机名
        $exchange = 'logs';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'fanout', false, false, false);

        // 声明创建队列
        [$queue, ,] = $channel->queue_declare('', false, false, true, false);

        // 绑定交换机与队列，并指定路由键
        $channel->queue_bind($queue, $exchange);

        // 消息消费
        $channel->basic_consume($queue, '', false, false, false, false, function ($msg) use ($output) {
            // 模拟耗时
            sleep(3);

            $output->writeln('received:' . $msg->body . PHP_EOL);

            // 确认消息处理完成
            $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
    }
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513114320560.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513114326156.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
交换机日志数据转到具有服务器分配名称的两个队列。

# 直接交换机 (Direct exchange)

在这一节中，我们将实现能够只订阅一部分消息。例如，我们想能够仅接收关键的错误消息。

**直接交换机 (Direct exchange)**

---

我们上一节教程中的日志消息是向所有消费者广播所有日志消息。我们希望扩展上一节中的日志消息，使其允许基于消息的严重性过滤消息。

我们之前使用的fanout类型交换机，它没有什么灵活性 - 只能广播消息。

这里我们将使用direct交换机，直接交换背后的算法很简单 - 消息将传递到订阅routing key完全匹配的队列。

为了说明上面步骤，参考下面的设置：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513135556416.png)
在这设置中，我们可以看到X exchange上绑定了两个队列(Q1, Q2). 第一个队列绑定了 routing key 为orange, 第二个队列绑定了两个键，一个是black另一个是green。

在这样的设置中，发布到交换机上具有orange路由的消息将被路由到队列 Q1。消息具有black 或者 green 路由秘钥的消息将会路由到Q2。其它所有的消息将会被丢弃。

**多重绑定**

---

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513135651372.png)
使用相同的键绑定到多个队列上是完全合法的。在上面的示例中，我们可以在X和Q1之间添加绑定*black*。那样的话，*direct*交换机将能如*fanout*交换机一样，在多个匹配的队列之间广播相同的消息。具有*black*路由键的消息将会被发送到Q1和Q2中。

**发出日志**

---

我们将使用此模型构建日志记录系统。我们将把消息发送到*direct*类型交换机上, 而不是使用*fanout*类型。我们将提供日志严重性作为*routing key*。这样接收消息的脚本就能够按照要接受消息的严重性进行接收。

我们首先关注发送日志部分：

一如以往，我们首先要建立一个交换机：

```php
$channel->exchange_declare('direct_logs', 'direct', false, false, false);
```

然后，我们准备发送信息：

```php
$channel->exchange_declare('direct_logs', 'direct', false, false, false);
#绑定error路由
$channel->basic_publish($msg, 'direct_logs', 'error');
```

**订阅**

---

接收消息大致和上一节教程一样，除了一个例外 - 我们将为每个我们感兴趣的消息严重性创建绑定。

C1消费者绑定了1个路由

```php
#绑定error路由
$channel->queue_bind($queue_name, 'direct_logs', 'error');
```

C2消费者绑定了3个路由

```php
#绑定info路由
$channel->queue_bind($queue_name, 'direct_logs', 'info');
#绑定error路由
$channel->queue_bind($queue_name, 'direct_logs', 'error');
#绑定warning路由
$channel->queue_bind($queue_name, 'direct_logs', 'warning');
```

> 代码整合

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513140047324.png)
生产端：

```php
<?php
declare (strict_types=1);

namespace app\controller;

use app\BaseController;
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

class Direct extends BaseController
{
    public function send()
    {
        // 交换机名
        $exchange = 'direct_logs';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'direct', false, false, false);

        // 模拟发送error信息内容
        $messageBody = 'error, now time:' . date('h:i:s');
        // 将我们需要的消息标记为持久化 - 通过设置AMQPMessage的参数delivery_mode = AMQPMessage::DELIVERY_MODE_PERSISTENT
        $message = new AMQPMessage($messageBody, [
            'content_type' => 'text/plain',
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);

        // 绑定error路由
        $channel->basic_publish($message, $exchange, 'error');

        // 模拟发送info信息内容
        $messageBody = 'info, now time:' . date('h:i:s');
        // 将我们需要的消息标记为持久化 - 通过设置AMQPMessage的参数delivery_mode = AMQPMessage::DELIVERY_MODE_PERSISTENT
        $message = new AMQPMessage($messageBody, [
            'content_type' => 'text/plain',
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);

        // 绑定info路由
        $channel->basic_publish($message, $exchange, 'info');

        // 模拟发送warning信息内容
        $messageBody = 'warning, now time:' . date('h:i:s');
        // 将我们需要的消息标记为持久化 - 通过设置AMQPMessage的参数delivery_mode = AMQPMessage::DELIVERY_MODE_PERSISTENT
        $message = new AMQPMessage($messageBody, [
            'content_type' => 'text/plain',
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);

        // 绑定info路由
        $channel->basic_publish($message, $exchange, 'warning');

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
        return 'send success';
    }
}
```

消费端C1：

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class C1 extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('c1')
            ->setDescription('the c1 command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 交换机名
        $exchange = 'direct_logs';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'direct', false, false, false);

        // 声明创建队列
        // 队列名称为空时，回升后曾一个随机名称队列
        [$queue, ,] = $channel->queue_declare('', false, false, true, false);

        // 绑定交换机与队列，并指定路由error
        $channel->queue_bind($queue, $exchange, 'error');

        $channel->basic_consume($queue, '', false, false, false, false, function ($msg) use ($output) {
            // 模拟耗时
            sleep(3);
            $output->writeln('received:' . $msg->body . PHP_EOL);
            // 确认消息处理完成
            $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
    }
}
```

消费端C2：

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class C2 extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('c2')
            ->setDescription('the c2 command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 交换机名
        $exchange = 'direct_logs';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'direct', false, false, false);

        // 声明创建队列
        // 队列名称为空时，会生成一个随机名称队列
        [$queue, ,] = $channel->queue_declare('', false, false, true, false);

        // 绑定交换机与队列，并指定路由info
        $channel->queue_bind($queue, $exchange, 'info');
        // 绑定交换机与队列，并指定路由error
        $channel->queue_bind($queue, $exchange, 'error');
        // 绑定交换机与队列，并指定路由warning
        $channel->queue_bind($queue, $exchange, 'warning');

        // 消息消费
        $channel->basic_consume($queue, '', false, false, true, false, function ($msg) use ($output) {
            // 模拟耗时
            sleep(3);
            $output->writeln('receive:' . $msg->body . PHP_EOL);
            // 确认消息处理完成
            $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
    }
}
```

控制台分别开启C1、C2消费端

发现C1只消费了error的消息

C2消费了error、info、warning三种类型的消息

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513142559580.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513142606615.png)

# 通配符交换机（Topic exchange）

在上一节教程中，我们改进了我们的日志消息，我们使用可以选择性接收信息的*direct*类型交换机，而不是使用只能进行广播的*fanout*类型交换机。

虽然使用*direct*类型交换改进了我们的日志消息，但它仍有限制 - 它不能基于通配符匹配进行路由选择。

为了在我们的系统中实现这样的功能，我们需要学习一个更复杂的*topic exchange*。

**通配符交换机（Topic exchange）**

---

发送到 *topic exchange* 的消息不能任意命名一个*routing key* 它必须是由.划分单词列表。这些单词可以是任意的，但它们通常指定与消息相关联的一些功能。

*topic exchange*背后的逻辑类似于*direct exchange*- 但是，*routing key*有两个重要的特殊情况：

```.net
类似于正则
1. *可以代替一个单词
2. #可以匹配0个或多个单词
```

在下面的例子中简单解释一下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021051314351196.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
【a】一条以” com.register.mail”为routing key的消息将会匹配到Register Queue与SaveMail Queue两个队列上，所以消息会发送到消息接收者1和消息接收者2。routing key为“email.register.test”的消息同样会被推送到Register Queue与SaveMail Queue两个队列。

【b】如果routing key为”com.register.wsh”的话，消息只会被推送到Register Queue上；routing key为”email.com.wsh”的消息会被推送到SaveMail Queue上，routing key为"email.com.test”的消息也会被推送到SaveMail Queue上，但同一条消息只会被推送到SaveMail Queue上一次。

注意：如果在发送消息的时候没有匹配到符合条件的routing key,那么这条消息将会被废弃。如：com.register.wsh.test 消息不会被推送到Register Queue上，但是注意 email.com.wsh.test则可以推送到SaveMail Queue上。

```.net
注意事项
当routing key为 “#” 时，topic exchange如同fanout exchange
当不使用 “*” 和 “#” 特殊字符时，topic exchange如同direct exchange
```

**整合代码**

---

生产端：

```php
<?php
declare (strict_types=1);

namespace app\controller;

use app\BaseController;
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

class Topic extends BaseController
{
    public function send()
    {
        // 交换机名
        $exchange = 'topic_logs';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'topic', false, false, false);

        // 模拟发送com.register.mail信息内容
        $messageBody = 'com.register.mail, now time:' . date('h:i:s');
        // 将我们需要的消息标记为持久化 - 通过设置AMQPMessage的参数delivery_mode = AMQPMessage::DELIVERY_MODE_PERSISTENT
        $message = new AMQPMessage($messageBody, [
            'content_type' => 'text/plain',
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);

        // 绑定com.register.mail路由
        $channel->basic_publish($message, $exchange, 'com.register.mail');

        // 模拟发送email.register.test信息内容
        $messageBody = 'email.register.test, now time:' . date('h:i:s');
        // 将我们需要的消息标记为持久化 - 通过设置AMQPMessage的参数delivery_mode = AMQPMessage::DELIVERY_MODE_PERSISTENT
        $message = new AMQPMessage($messageBody, [
            'content_type' => 'text/plain',
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);

        // 绑定email.register.test路由
        $channel->basic_publish($message, $exchange, 'email.register.test');

        // 模拟发送com.register.wsh信息内容
        $messageBody = 'com.register.wsh, now time:' . date('h:i:s');
        // 将我们需要的消息标记为持久化 - 通过设置AMQPMessage的参数delivery_mode = AMQPMessage::DELIVERY_MODE_PERSISTENT
        $message = new AMQPMessage($messageBody, [
            'content_type' => 'text/plain',
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);

        // 绑定com.register.wsh路由
        $channel->basic_publish($message, $exchange, 'com.register.wsh');

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
        return 'send success';
    }
}
```

消费端C1：

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class TopicC1 extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('topicc1')
            ->setDescription('the topicc1 command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 交换机名
        $exchange = 'topic_logs';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'topic', false, false, false);

        // 声明创建队列
        // 队列名称为空时，回升后曾一个随机名称队列
        [$queue, ,] = $channel->queue_declare('', false, false, true, false);

        // 绑定交换机与队列，并指定路由*.register.*
        $channel->queue_bind($queue, $exchange, '*.register.*');

        $channel->basic_consume($queue, '', false, false, false, false, function ($msg) use ($output) {
            // 模拟耗时
            sleep(3);
            $output->writeln('received:' . $msg->body . PHP_EOL);
            // 确认消息处理完成
            $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
    }
}
```

消费端C2：

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class TopicC2 extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('topicc2')
            ->setDescription('the topicc2 command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 交换机名
        $exchange = 'topic_logs';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'topic', false, false, false);

        // 声明创建队列
        // 队列名称为空时，回升后曾一个随机名称队列
        [$queue, ,] = $channel->queue_declare('', false, false, true, false);

        // 绑定交换机与队列，并指定路由*.*.mail
        $channel->queue_bind($queue, $exchange, '*.*.mail');

        $channel->basic_consume($queue, '', false, false, false, false, function ($msg) use ($output) {
            // 模拟耗时
            sleep(3);
            $output->writeln('received:' . $msg->body . PHP_EOL);
            // 确认消息处理完成
            $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
    }
}
```

控制台分别开启C1、C2消费端

发现C1只消费了*.register.*的消息
C2消费了*.*.mail、email.#二种类型的消息
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021051315170877.png)

# 远程调用（RPC）

**RPC工作流程**

当客户端启动时，它创建一个匿名独占回调队列。

对于一个RPC请求，客户端发送一个具有两个属性的消息： *reply_to* 设置回调队列， *correlation_id* 设置每个请求的唯一值。

请求被发送到*rpc_queue*队列。

RPC服务端一直在等待队列上的请求，当请求出现时，它将执行请求的任务，并使用reply_to字段中的队列将结果返回给客户端。

客户端等待回调队列中的数据。当信息出现时，它检查correlation_id属性。如果它与请求中的值相匹配，则返回对应的应用程序的响应。

**回调队列**

---

客户端发送请求消息，服务器回复响应消息。为了收到响应，我们需要发送一个回调队列地址与请求：

```php
[$queue_name, ,] = $channel->queue_declare("", false, false, true, false);

$msg = new AMQPMessage("message", [
    'reply_to' => $queue_name
]);

$channel->basic_publish($msg, '', 'rpc_queue');
```

> **消息的属性**
> AMQP 0-9-1 协议为消息预定义了一组默认的14个属性。大多数属性很少使用，除了以下几个：
> +*delivery_mode*: 将消息标记为持久性(值为2)或者费持久(1). 你可能记得这个属性，我们在 第二节 教程中使用过。
> +*content_type*: 用于描述MIME类型的编码。例如，对于经常使用的JSON编码，将此属性设置为 “application/json” 是一个很好的做法。
> +*reply_to*: 主要用于命名一个回调队列
> +*correlation_id*: 用于将RPC响应与请求相关联

**correlation_id（关联ID）**

---

在上面提出的方法中，每个RPC请求将创建一个回调队列，这样效率非常低，我们需要为每个客户端创建同一个回调队列。

这样又引发了一个新的问题，在该队列中收到响应后，不能确定响应所属的请求，这就是 correlation_id 派上用场的时候。我们将为每个请求设置一个唯一值(correlation_id)。之后，当我们在回调队列中收到一条消息时，我们将查看此属性，并且基于此，我们能够将响应与请求向匹配。如果我们收到一个未知的 correlation_id 的值，我们就能安全的丢弃该消息 - 它不属于我们的请求。

**总结**

---

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513154202869.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)

**合并代码**

---

RPC服务端

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class RpcServer extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('rpcserver')
            ->setDescription('the rpcserver command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 队列名
        $queue = 'rpc_queue';
        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明创建队列
        $channel->queue_declare($queue, false, false, false, false);

        // 公平调度
        $channel->basic_qos(null, 1, null);

        // 消息消费
        $channel->basic_consume($queue, '', false, false, false, false, function ($msg) use ($output) {
            // 接收到RPC客户端发送的信息
            $output->writeln('received:' . $msg->body . PHP_EOL);

            // 执行方法，回调队列
            $reply = new AMQPMessage('rpc server reply message', [
                'correlation_id' => $msg->get('correlation_id')
            ]);

            $msg->delivery_info['channel']->basic_publish(
                $reply, '', $msg->get('reply_to'));

            // 确认消息处理完成
            $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
    }
}
```

RPC客户端：

```php
<?php
declare (strict_types=1);

namespace app\controller;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

class RpcClient
{
    public function call()
    {
        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();

        // 声明回调队列
        $callback_queue = $channel->queue_declare(
            'callback_queue', false, false, true, false);

        // RPC客户端请求的参数
        $corr_id = uniqid();
        $msg = new AMQPMessage('rpc client send message', [
            'correlation_id' => $corr_id,
            'reply_to' => 'callback_queue'
        ]);

        // 发送RPC请求
        $channel->basic_publish($msg, '', 'rpc_queue');

        // RPC服务端返回的内容
        $response = null;

        // 等待RPC回调
        $channel->basic_consume('callback_queue', '', false, false, false, false, function ($reply) use ($corr_id, &$response) {
            if ($reply->get('correlation_id') == $corr_id) {
                $response = $reply->body;
            }

            // 确认消息处理完成
            $reply->delivery_info['channel']->basic_ack($reply->delivery_info['delivery_tag']);
        });

        while (!$response) {
            $channel->wait();
        }

        // 输出RPC服务端返回的内容
        echo $response;

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
    }
}
```

启动服务端：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513161437987.png)
浏览器访问客户端：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513161526537.png)
返回控制台看服务器情况：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513161538135.png)

> 注意：
> 
> - 如果RPC服务处理太慢，你可以运行另一个服务来扩展。 尝试在新的控制台运行一个rpc_server
> - RPC客户端需要发送和接收一条消息。但是不需要像queue_declare这样的同步调用。

# 延迟队列、死信队列

**延迟队列**

---

延迟队列存储的对象为对应的延时消息，等待指定时间后，消费者才拿到这个消息进行消费。RabbitMQ本身没有直接支持的延迟队列功能，我们可以设置 x-message-ttl，来控制消息的生存时间，如果超时，则消息成为dead letter（死信），进入死信队列，将死信队列指定为普通的消费队列即可实现延迟消费。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513165943474.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
**死信队列**

---

死信队列中（dead letter）死信的消息来源：

1. 消息被拒绝（basic.reject或basic.nack）并且requeue=false
2. 消息TTL过期（x-message-ttl参数值即为超时时间）
3. 队列达到最大长度（队列满了，无法再添加数据到mq中）

普通队列指定死信队列的相关参数:
x-dead-letter-exchange：用来设置死信后发送的交换机
x-dead-letter-routing-key：用来设置死信的routing key
x-message-ttl：单位毫秒，消息、队列的生命周期。超时后消息会变成dead message

**合并代码**

---

```php
<?php
declare (strict_types=1);

namespace app\controller;

use app\BaseController;
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;
use PhpAmqpLib\Wire\AMQPTable;

class Delay extends BaseController
{
    public function send()
    {
        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();

        // 创建DLX及死信队列
        $channel->exchange_declare('dlx_exchange', 'direct', false, false, false);
        $channel->queue_declare('dlx_queue', false, true, false, false);
        $channel->queue_bind('dlx_queue', 'dlx_exchange', 'dlx_routing_key');

        // 创建延迟队列
        $channel->exchange_declare('delay_exchange', 'direct', false, false, false);
        $args = new AMQPTable();
        // 消息过期方式：设置 queue.normal队列中的消息5s后过期
        $args->set('x-message-ttl', 5000);
        // 设置队列最大长度方式：x-max-length
//        $args->set('x-max-length', 1);
        $args->set('x-dead-letter-exchange', 'dlx_exchange');
        $args->set('x-dead-letter-routing-key', 'dlx_routing_key');
        $channel->queue_declare('delay_queue', false, true, false, false, false, $args);
        $channel->queue_bind('delay_queue', 'delay_exchange', 'delay_routing_key');

        // 模拟发送消息内容
        $messageBody = "该消息将在5s后发送到延迟队列（" . date("h:i:s") . "）";
        // 将我们需要的消息标记为持久化
        $message = new AMQPMessage($messageBody, [
            'content_type' => 'text/plain',
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);

        // 绑定delay_routing_key路由
        $channel->basic_publish($message, 'delay_exchange', 'delay_routing_key');

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();

        return 'send success';
    }


}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021051409485730.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
5秒之后，消息会过期，然后被进入 dlx_exchange, 进而路由到 dlx_queue 队列中：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210514095005313.png)
消费者代码

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class Delay extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('delay')
            ->setDescription('the delay command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 交换机名
        $exchange = 'dlx_exchange';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'direct', false, false, false);
        $channel->queue_declare('dlx_queue', false, true, false, false);
        $channel->queue_bind('dlx_queue', $exchange, 'dlx_routing_key');

        // 消息消费
        $channel->basic_consume('dlx_queue', '', false, false, false, false, function ($msg) use ($output) {
//            $body = iconv('UTF-8', 'GB2312//IGNORE', $msg->body);
            $output->writeln(date('h:i:s') . 'received:' . $msg->body . PHP_EOL);

            // 确认消息处理完成
            $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();


    }
}
```

运行消费者代码之后，消息会从dlx_queue中消费掉：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021051409505622.png)

# 重试队列（可靠性投递，重试超过3次，入库告警）

**消息消费确认实现原理**

当消费者的消息消费异常时，消息进入延迟重试队列，待超时后重新发送到重试队列指定的死信队列，死信队列重新消费信息，如果又出现死信情况，继续进入延时重试队列，依次循环，当重试超过3次后，则入库告警，等待人工处理。

**生产者代码**

```php
<?php
declare (strict_types=1);

namespace app\controller;

use app\BaseController;
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;
use PhpAmqpLib\Wire\AMQPTable;

class Retry extends BaseController
{
    public function send()
    {
        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();

        // 创建DLX及死信队列
        $channel->exchange_declare('dlx_exchange', 'direct', false, false, false);
        $channel->queue_declare('dlx_queue', false, true, false, false);
        $channel->queue_bind('dlx_queue', 'dlx_exchange', 'dlx_routing_key');

        // 创建延迟队列
        $channel->exchange_declare('delay_exchange', 'direct', false, false, false);
        $args = new AMQPTable();
        // 消息过期方式：设置queue.normal队列中的消息5s之后过期
        $args->set('x-message-ttl', 5000);
        // 设置队列最大长度方式： x-max-length
//        $args->set('x-max-length', 1);
        $args->set('x-dead-letter-exchange', 'dlx_exchange');
        $args->set('x-dead-letter-routing-key', 'dlx_routing_key');
        $channel->queue_declare('delay_queue', false, true, false, false, false, $args);
        $channel->queue_bind('delay_queue', 'delay_exchange', 'delay_routing_key');

        // 模拟发送消息内容
        $messageBody = "该消息将在5s后发送到延迟队列（" . date("h:i:s") . "）";
        // 将我们需要的消息标记为持久化
        $message = new AMQPMessage($messageBody, [
            'content_type' => 'text/plain',
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);

        // 设置重试次数，默认0
        $headers = new AMQPTable([
            'retry_nums' => 1
        ]);

        $message->set('application_headers', $headers);

        // 绑定delay.routing_key路由
        $channel->basic_publish($message, 'delay_exchange', 'delay_routing_key');

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
        return 'send success';
    }


}
```

**消费者代码**

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Wire\AMQPTable;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class Retry extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('retry')
            ->setDescription('the retry command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 交换机名
        $exchange = 'dlx_exchange';
        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'direct', false, false, false);
        $channel->queue_declare('dlx_queue', false, true, false, false);
        $channel->queue_bind('dlx_queue', $exchange, 'dlx_routing_key');

        // 创建重试队列
        $channel->exchange_declare('delay_exchange', 'direct', false, false, false);

        // 消息消费
        $channel->basic_consume('dlx_queue', '', false, false, false, false, function ($msg) use ($output) {
//            $body = iconv("UTF-8", "GB2312//IGNORE", $msg->body);
            $output->writeln(date("h:i:s") . " Received " . $msg->body . PHP_EOL);

            // 确认消息处理完成
            $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);

            $msg_headers = $msg->get('application_headers')->getNativeData();

            // 重试次数超过3次，则入库告警
            if (intval($msg_headers['retry_nums']) > 3) {
//                $body = iconv("UTF-8", "GB2312//IGNORE", "重试次数超过3次，则入库告警");
                $output->writeln(date("h:i:s") . " Error " . "重试次数超过3次，则入库告警" . PHP_EOL);
            } else {
                // 建立连接
                $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
                // 获取信道
                $channel = $connection->channel();
                // 重试次数加1
                $headers = new AMQPTable([
                    "retry_nums" => intval($msg_headers['retry_nums']) + 1
                ]);
                $msg->set('application_headers', $headers);

                // 放回重试队列
                $channel->basic_publish($msg, "delay_exchange", "delay_routing_key");
            }
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
    }
}
```

运行消费者代码之后：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210514103458790.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)

# 消费幂等

由于网络闪断、MQ Broker端异常等原因可能导致消费端回送的ask确认消息失败或者异常，导致消费端再次收到实际已经消费的消息

> 解决思路
> 每个消息增加一个消息ID，消费端消费前读取缓存，查看消息ID是否存在，不存在则直接return，存在则消费，消费成功删除缓存数据，实现消费的幂等。

**生产者代码**

```php
<?php
declare (strict_types=1);

namespace app\controller;

use app\BaseController;
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;
use PhpAmqpLib\Wire\AMQPTable;
use think\facade\Cache;

class Idempotent extends BaseController
{
    public function send()
    {
        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', 5672, 'zq', '123456', '/');
        // 获取信道
        $channel = $connection->channel();

        // 创建DLX及死信队列
        $channel->exchange_declare('dlx_exchange', 'direct', false, false, false);
        $channel->queue_declare('dlx_queue', false, true, false, false);
        $channel->queue_bind('dlx_queue', 'dlx_exchange', 'dlx_routing_key');

        // 创建延迟队列
        $channel->exchange_declare('delay_exchange', 'direct', false, false, false);
        $args = new AMQPTable();
        // 消息过期方式：设置queue.normal队列中的消息5s之后过期
        $args->set('x-message-ttl', 5000);
        // 设置队列最大长度方式： x-max-length
//        $args->set('x-max-length', 1);
        $args->set('x-dead-letter-exchange', 'dlx_exchange');
        $args->set('x-dead-letter-routing-key', 'dlx_routing_key');
        $channel->queue_declare('delay_queue', false, true, false, false, false, $args);
        $channel->queue_bind('delay_queue', 'delay_exchange', 'delay_routing_key');

        // 模拟发送消息内容
        $messageBody = "重复消息发送（" . date("h:i:s") . "）";
        // 将我们需要的消息标记为持久化
        $message = new AMQPMessage($messageBody, [
            'content_type' => 'text/plain',
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);

        // 设置消息ID，防止重复消费
        $corr_id = uniqid();
        $headers = new AMQPTable([
            'correlation_id' => $corr_id
        ]);

        $message->set('application_headers', $headers);

        Cache::set($corr_id, $corr_id, 3600);

        // 绑定delay.routingKey路由
        $channel->basic_publish($message, 'delay_exchange', 'delay_routing_key');

        // 模拟发送重复消息
        $channel->basic_publish($message, 'delay_exchange', 'delay_routing_key');

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
        return 'send success';
    }
}
```

**消费者代码**

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Wire\AMQPTable;
use think\console\Command;
use think\console\Input;
use think\console\Output;
use think\facade\Cache;

class Idempotent extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('idempotent')
            ->setDescription('the idempotent command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', 5672, 'zq', '123456', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare('dlx_exchange', 'direct', false, false, false);
        $channel->queue_declare('dlx_queue', false, true, false, false);
        $channel->queue_bind('dlx_queue', 'dlx_exchange', 'dlx_routing_key');

        // 创建重试队列
        $channel->exchange_declare('delay_exchange', 'direct', false, false, false);
        $args = new AMQPTable();
        // 消息过期方式：设置queue.normal队列中的消息5s之后过期
        $args->set('x-message-ttl', 5000);
        // 设置队列最大长度方式： x-max-length
//        $args->set('x-max-length', 1);
        $args->set('x-dead-letter-exchange', 'dlx_exchange');
        $args->set('x-dead-letter-routing-key', 'dlx_routing_key');
        $channel->queue_declare('delay_queue', false, true, false, false, false, $args);
        $channel->queue_bind('delay_queue', 'delay_exchange', 'delay_routing_key');

        // 消息消费
        $channel->basic_consume('dlx_queue', '', false, false, false, false, function ($msg) use ($output) {
            $msg_headers = $msg->get('application_headers')->getNativeData();
            $corr_id = $msg_headers['correlation_id'];

            // 判断是否已经消费过
            if (Cache::get($corr_id) === null) {
//                $body = iconv("UTF-8", "GB2312//IGNORE", "该消息已消费，不再消费");
                $output->writeln(date("h:i:s") . '该消息已消费，不再消费' . PHP_EOL);
                // 确认消息处理完成
                $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
                return;
            }

//            $body = iconv("UTF-8", "GB2312//IGNORE", $msg->body);
            $output->writeln(date("h:i:s") . " Received " . $msg->body . PHP_EOL);

            // 确认消息处理完成
            $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
            Cache::delete($corr_id);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
    }
}
```

运行消费者代码之后：使用MQ处理：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513092721659.png)

> 2、 流量削峰

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513092931133.png)

> 3、应用解耦

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513092943133.png)

# RabbitMQ安装

RabbitMQ 是 Erlang 语言写的，首先我们需要安装Erlang 环境
RabbitMQ 不同版本也对应着不同的Erlang 版本，进入官网：[RabbitMQ](https://www.rabbitmq.com/which-erlang.html)
查看对照表：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513093354483.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
**一、安装erlang**

---

1、进入package Cloud，这里会很详细的解释，安装过程中需要哪些和生成一些文件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513093449794.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513093457401.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
2、在命令行中执行脚本

```bash
curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash
```

3、安装

```bash
sudo yum install erlang-21.3.8.16-1.el7.x86_64
```

4、测试\
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513093553730.png)
**二、安装rabbitmq**

---

1、进入package Cloud，这里会很详细的解释，安装过程中需要哪些和生成一些文件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513093618664.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513093625407.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
2、在命令行中执行脚本

```bash
curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash
```

3、安装

```bash
sudo yum install rabbitmq-server-3.8.4-1.el7.noarch
```

4、启动

```bash
#启动服务
service rabbitmq-server start
#查看状态
service rabbitmq-server status
#设置开机自启动
chkconfig rabbitmq-server on
```

5、开启web管理界面，增加用户

```bash
#开启web管理界面
rabbitmq-plugins enable rabbitmq_management
#添加用户名和密码
abbitmqctl add_user zq 123456
#将用户admin设置为管理员
rabbitmqctl set_user_tags zq administrator
```

6、测试

在浏览器访问http://127.0.0.1:15672，登录 Web 管理界面
输入上面添加的用户名/密码:zq/123456
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513102531983.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513093812782.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)

# PHP安装rabbitmq、php-amqplib扩展

**1、安装依赖库rabbitmq-c**

---

```bash
yum install librabbitmq-devel.x86_64 -y  
wget http://pecl.php.net/get/amqp-1.10.0.tgz 
tar zxvf amqp-1.10.0.tgz  
cd amqp-1.10.0  
./configure --with-php-config=/usr/local/php/bin/php-config --with-amqp  
make  
make install  
vim /usr/local/php/etc/php.ini  
extension=amqp.so
```

**2、重启php-fpm**

---

```bash
#重启php-fpm
service php-fpm restart
#查看是否安装了amqp扩展
php -m
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513093859620.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
**3、安装php-amqplib扩展**

---

> 进入tp6项目的根目录，安装扩展

```bash
 composer require php-amqplib/php-amqplib
```

> 安装完成后会在vendor增加一个php-amqplib目录

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513093949708.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)

# RabbitMQ入门

> RabbitMQ 是一个消息代理：它接收并转发消息。

RabbitMQ 你可以把它当成一个邮箱，当你把你想发送的邮件投进邮箱时，你可以确定邮递员最终会把邮件送到你的收件人。

RabbitMQ 它接收、存储和转发二进制的数据–即消息(message)。

RabbitMQ中一些常使用术语：

- Publisher：生产者，消息的发送方。
- Connection：网络连接。
- Channel：信道，多路复用连接中的一条独立的双向数据流通道。
- Exchange：交换机（路由器），负责消息的路由到相应队列。
- Binding：队列与交换器间的关联绑定。消费者将关注的队列绑定到指定交换器上，以便Exchange能准确分发消息到指定队列。
- Queue：队列，消息的缓冲存储区。
- Virtual Host：虚拟主机，虚拟主机提供资源的逻辑分组和分离。包含连接，交换，队列，绑定，用户权限，策略等。
- Broker：消息队列的服务器实体。
- Consumer：消费者，消息的接收方。

RabbitMQ 消息结构：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513094308112.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)

**hello world**

现在我们来实现一个生产者负责发送5条消息，一个消费者负责接收消息并打印出来

**1、生产者**

---

生产者将会连接 RabbitMQ、发送5条消息、然后关闭连接。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513094445517.png)

```php
<?php
declare(strict_types=1);

namespace app\index\controller;

use app\BaseController;
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Exchange\AMQPExchangeType;
use PhpAmqpLib\Message\AMQPMessage;
use PhpAmqpLib\Wire\AMQPTable;

class Mq extends BaseController
{
    public function send()
    {
        // 队列名 消息队列载体，每个消息都会被投入到一个或多个队列
        $queue = 'hello';
        $routing = 'hello';

        // 建立链接
        $connection = new AMQPStreamConnection('127.0.0.1', 5672, 'zq', '123456', '/');
        // 获取信道
        $channel = $connection->channel();

        // 声明创建队列
        $channel->queue_declare($queue, false, false, false, false);

        for ($i = 0; $i < 5; ++$i) {
            sleep(1); // 休眠1s
            // 消息内容
            $messageBody = 'hello,zq now time:' . date('H:i:s');
            // 将我们需要的消息标记为持久化 - 通过设置AMQPMESSAGE的参数delivery_mode = AMQPMessage::DELIVERY_MODE_PERSISTENT
            $message = new AMQPMessage($messageBody, [
                'content-type' => 'text/plain',
                'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
            ]);
            // 发送消息
            $channel->basic_publish($message, '', $routing);
            echo 'send message:' . $i . "\n";
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();

        return 'send success';


    }


}
```

在浏览器访问http://tp6.ws/index/mq/send，发送5条消息

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513101156342.png)

登陆RabbitMQ Web 管理界面

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513101331377.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
**2、消费者**

---

消费者从 RabbitMQ 中获取消息，与发布消息的生产者不同，消费者一直运行以监听消息，有新消息就立即处理
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513102136439.png)
利用TP6自定义命令行实现消费者

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class Mq extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('mq')
            ->setDescription('the mq command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 队列名 消息队列载体，每个消息都会被投入到一个或多个队列
        $queue = 'hello';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'zq', '123456', '/');
        // 获取信道
        $channel = $connection->channel();

        // 声明创建队列
        $channel->queue_declare($queue, false, false, false, false);

        // 消息消费
        $channel->basic_consume($queue, '', false, true, false, false, function ($msg) use ($output) {
            $output->writeln('received' . $msg->body . PHP_EOL);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();

    }


}
```

在控制台启动消费者：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513102311805.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)

切换到RabbitMQ Web 管理界面，消息已经被消费：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513102342722.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)

# 工作队列（Work Queues）

工作队列：其主要的思想是避免立即执行资源密集型任务，并且阻塞进程等待任务完成。相反，我们让这类型任务稍后执行，我们将它封装为消息，并将其添加到任务队列中。在后台启动一个或者多个工作进程，读取工作队列中的任务，并且执行，当你运行多个工作进程时，他们共享工作队列中的任务。

> 这个思想在Web应用程序中特别有用，用于处理在短时间的HTTP请求中无法处理的复杂任务。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513102710939.png)
我们将之前的消费者app\command\mq.php做一些修改：它需要为消息体中的每个消息模拟耗时三秒的工作时间。

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class Mq extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('mq')
            ->setDescription('the mq command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 队列名 消息队列载体，每个消息都会被投入到一个或多个队列
        $queue = 'hello';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'zq', '123456', '/');
        // 获取信道
        $channel = $connection->channel();

        // 声明创建队列
        $channel->queue_declare($queue, false, false, false, false);

        // 公平调度，新的消息将发送到一个处于空闲的消费者
        $channel->basic_qos(null, 1, null);

        // 消息消费
        $channel->basic_consume($queue, '', false, true, false, false, function ($msg) use ($output) {
            // 模拟耗时
            sleep(3);
            $output->writeln('received' . $msg->body . PHP_EOL);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();

    }


}
```

接着我们看看两个消费者控制台输出的内容：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513103145608.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513103233238.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
**消息确认机制 (ack)**

---

想一下如果一个消费者在执行一个长时间的任务的过程中异常退出会怎样？在我们目前的代码下，一旦RabbitMQ向消费者发送了消息后，此条消息将立即从内存中删除。在这种情况下，如果你杀掉一个正在处理消息的工作进程，那么我们将丢失这条消息！

为了确保消息永远不会丢失，RabbitMQ支持消息确认机制。消费者发送一个确认信息告诉RabbitMQ已经收到消息并且处理完成，此时RabbitMQ就可以删除这条消息了。

如果消费者进程异常退出(其通道关闭，连接断开或者TCP连接丢失)，而不发送确认信息，RabbitMQ将会认为此进程当前处理的消息未处理完成，就会将此消息重新放回消息队列。如果此时有其他消费者在线，则会迅速将这条消息重新提供给另一个消费者。这样就可以确保没有消息丢失。即使有消费者进程异常退出的情况。

在默认情况下，消息确认机制是关闭的。现在是时候开启消息确认机制，将basic_consumer的第四个参数设置为false(true表示不开启消息确认)，并且工作进程处理完消息后发送确认消息。

```php
//开启消息确认
$channel->basic_consume($queue, '', false, false, false, false, function ($msg) use ($output)  {
    //模拟耗时
    sleep(3);
    $output->writeln(" Received " . $msg->body .  PHP_EOL);

    //确认消息处理完成
    $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);

});
```

**消息持久化**

---

我们已经学习了如何确保即使消费者异常退出后，任务也不会丢失。但是如果RabbitMQ服务器挂了，我们的任务任然会丢失。

当RabbitMQ退出或崩溃时，它会丢失队列和其中的消息，除非你告诉他要持久化这些信息。

`我们需要做两件事来确保消息不会丢失：将队列和消息标记为 durable(持久化)。`

首先，我们需要确保RabbitMQ不会丢失消息队列。我们需要将队列声明为持久化，只要将 queue_declare 的第三个参数设置为 true 就行了，设置为 true 的标志需要同时应用于生产者和消费者中：

```php
// 声明创建队列
$channel->queue_declare($queue, false, true, false, false);
```

到这一步，我们确信即使RabbitMQ重启，hello队列也不会丢失。现在我们需要将我们需要的消息标记为持久化，通过设置AMQPMessage 的参数 delivery_mode = AMQPMessage::DELIVERY_MODE_PERSISTENT:

```php
// 将我们需要的消息标记为持久化 - 通过设置AMQPMESSAGE的参数delivery_mode = AMQPMessage::DELIVERY_MODE_PERSISTENT
$message = new AMQPMessage($messageBody, [
    'content-type' => 'text/plain',
    'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
]);
```

**关于队列的大小**

---

如果所有的消费者进程都处于忙碌状态，你的队列可能会溢出。你应该留意一下这种情况，也许你可以增加更多的消费者进程，或者采取一些其他的策略。

# 发布/订阅（Publish/Subscribe）

在这一节中，我们会向多个消费者传递相同的信息。这种模式被称为“发布/订阅”。

**交换机（Exchanges）**

---

在上一篇教程中，我们通过工作队列发送和接收消息。现在是时候在Rabbit中引入完整的消息传递模式了。

我们快速回顾一下前面教程中介绍的内容：

- 生产者是发送消息的用户应用程序
- 队列是存储消息的缓冲区
- 消费者是接收消息的用户应用程序

RabbitMQ中消息传递模型的核心思想是，生产者从不将任何消息直接发送到队列。实际上，生产者甚至通常不知道是否将消息传递到了队列中，生产者只能将信息发送到交换机(exchange)。交换机一方面，它接收来自生产者的消息; 另一方面将接收到的消息推送到队列。

交换机准确知道接收到的消息应该如何处理，应该添加到特定的队列， 应该添加到许多的队列中，或者应该丢弃，其规则由交换类型定义。

`有几种交换类型可用：direct, topic, headers, fanout. 我们将学习第一个 - fanout交换机`

**直接交换机 (Fanout exchange)**

---

fanout 交换非常简单，它是将所有的消息广播到所有已知的队列。
让我们创建一个这种类型的交换器，并且将其称为logs：

```php
$channel->exchange_declare('logs', 'fanout', false, false, false);
```

**默认交换**

---

在本教程的前面部分，我们对交换没有任何了解，但是任然能够将消息发送到队列。这是可能的，因为我们使用的是默认交换，我们通过空字符串”“标识。
回顾一下我们之前发布的消息：

```php
# 在这里，我们使用默认交换
$channel->basic_publish($msg, '');
```

现在，我们可以将消息发布到我们命名的交换机：

```php
$channel->exchange_declare('logs', 'fanout', false, false, false);
$channel->basic_publish($msg, 'logs');
```

**绑定**

---

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513104408810.png)
我们已经创建了一个fanout交换机和一个队列。现在我们需要告诉交换机发送消息到我们的队列。交换机和队列之间的关系称为绑定。

```php
$channel->queue_bind($queue_name, 'logs');
```

**总结**

---

我们现在把消息发布到我们的logs交换机。

```php
<?php
declare (strict_types=1);

namespace app\controller;

use app\BaseController;
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

class PubSub extends BaseController
{
    public function send()
    {
        // 交换机名
        $exchange = 'logs';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'fanout', false, false, false);

        for ($i = 0; $i < 5; ++$i) {
            sleep(1); // 休眠1s
            // 消息内容
            $messageBody = 'heelo,zq now time:' . date('h:i:s');
            // 将我们需要的消息标记为持久化
            $message = new AMQPMessage($messageBody, [
                'content_type' => 'text/plain',
                'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
            ]);

            // 发送消息
            $channel->basic_publish($message, $exchange);
            echo 'send message:' . $i . "\n";
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
        return 'send success';
    }
}
```

在浏览器访问http://tp6rbmq.ws/pubsub/send，发送5条消息
登陆RabbitMQ Web 管理界面，增加了一个logs交换机：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513113552366.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
然后我们看一下消费端：

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class PubSub extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('pubsub')
            ->setDescription('the pubsub command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 交换机名
        $exchange = 'logs';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'fanout', false, false, false);

        // 声明创建队列
        [$queue, ,] = $channel->queue_declare('', false, false, true, false);

        // 绑定交换机与队列，并指定路由键
        $channel->queue_bind($queue, $exchange);

        // 消息消费
        $channel->basic_consume($queue, '', false, false, false, false, function ($msg) use ($output) {
            // 模拟耗时
            sleep(3);

            $output->writeln('received:' . $msg->body . PHP_EOL);

            // 确认消息处理完成
            $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
    }
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513114320560.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513114326156.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
交换机日志数据转到具有服务器分配名称的两个队列。

# 直接交换机 (Direct exchange)

在这一节中，我们将实现能够只订阅一部分消息。例如，我们想能够仅接收关键的错误消息。

**直接交换机 (Direct exchange)**

---

我们上一节教程中的日志消息是向所有消费者广播所有日志消息。我们希望扩展上一节中的日志消息，使其允许基于消息的严重性过滤消息。

我们之前使用的fanout类型交换机，它没有什么灵活性 - 只能广播消息。

这里我们将使用direct交换机，直接交换背后的算法很简单 - 消息将传递到订阅routing key完全匹配的队列。

为了说明上面步骤，参考下面的设置：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513135556416.png)
在这设置中，我们可以看到X exchange上绑定了两个队列(Q1, Q2). 第一个队列绑定了 routing key 为orange, 第二个队列绑定了两个键，一个是black另一个是green。

在这样的设置中，发布到交换机上具有orange路由的消息将被路由到队列 Q1。消息具有black 或者 green 路由秘钥的消息将会路由到Q2。其它所有的消息将会被丢弃。

**多重绑定**

---

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513135651372.png)
使用相同的键绑定到多个队列上是完全合法的。在上面的示例中，我们可以在X和Q1之间添加绑定*black*。那样的话，*direct*交换机将能如*fanout*交换机一样，在多个匹配的队列之间广播相同的消息。具有*black*路由键的消息将会被发送到Q1和Q2中。

**发出日志**

---

我们将使用此模型构建日志记录系统。我们将把消息发送到*direct*类型交换机上, 而不是使用*fanout*类型。我们将提供日志严重性作为*routing key*。这样接收消息的脚本就能够按照要接受消息的严重性进行接收。

我们首先关注发送日志部分：

一如以往，我们首先要建立一个交换机：

```php
$channel->exchange_declare('direct_logs', 'direct', false, false, false);
```

然后，我们准备发送信息：

```php
$channel->exchange_declare('direct_logs', 'direct', false, false, false);
#绑定error路由
$channel->basic_publish($msg, 'direct_logs', 'error');
```

**订阅**

---

接收消息大致和上一节教程一样，除了一个例外 - 我们将为每个我们感兴趣的消息严重性创建绑定。

C1消费者绑定了1个路由

```php
#绑定error路由
$channel->queue_bind($queue_name, 'direct_logs', 'error');
```

C2消费者绑定了3个路由

```php
#绑定info路由
$channel->queue_bind($queue_name, 'direct_logs', 'info');
#绑定error路由
$channel->queue_bind($queue_name, 'direct_logs', 'error');
#绑定warning路由
$channel->queue_bind($queue_name, 'direct_logs', 'warning');
```

> 代码整合

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513140047324.png)
生产端：

```php
<?php
declare (strict_types=1);

namespace app\controller;

use app\BaseController;
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

class Direct extends BaseController
{
    public function send()
    {
        // 交换机名
        $exchange = 'direct_logs';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'direct', false, false, false);

        // 模拟发送error信息内容
        $messageBody = 'error, now time:' . date('h:i:s');
        // 将我们需要的消息标记为持久化 - 通过设置AMQPMessage的参数delivery_mode = AMQPMessage::DELIVERY_MODE_PERSISTENT
        $message = new AMQPMessage($messageBody, [
            'content_type' => 'text/plain',
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);

        // 绑定error路由
        $channel->basic_publish($message, $exchange, 'error');

        // 模拟发送info信息内容
        $messageBody = 'info, now time:' . date('h:i:s');
        // 将我们需要的消息标记为持久化 - 通过设置AMQPMessage的参数delivery_mode = AMQPMessage::DELIVERY_MODE_PERSISTENT
        $message = new AMQPMessage($messageBody, [
            'content_type' => 'text/plain',
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);

        // 绑定info路由
        $channel->basic_publish($message, $exchange, 'info');

        // 模拟发送warning信息内容
        $messageBody = 'warning, now time:' . date('h:i:s');
        // 将我们需要的消息标记为持久化 - 通过设置AMQPMessage的参数delivery_mode = AMQPMessage::DELIVERY_MODE_PERSISTENT
        $message = new AMQPMessage($messageBody, [
            'content_type' => 'text/plain',
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);

        // 绑定info路由
        $channel->basic_publish($message, $exchange, 'warning');

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
        return 'send success';
    }
}
```

消费端C1：

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class C1 extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('c1')
            ->setDescription('the c1 command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 交换机名
        $exchange = 'direct_logs';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'direct', false, false, false);

        // 声明创建队列
        // 队列名称为空时，回升后曾一个随机名称队列
        [$queue, ,] = $channel->queue_declare('', false, false, true, false);

        // 绑定交换机与队列，并指定路由error
        $channel->queue_bind($queue, $exchange, 'error');

        $channel->basic_consume($queue, '', false, false, false, false, function ($msg) use ($output) {
            // 模拟耗时
            sleep(3);
            $output->writeln('received:' . $msg->body . PHP_EOL);
            // 确认消息处理完成
            $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
    }
}
```

消费端C2：

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class C2 extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('c2')
            ->setDescription('the c2 command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 交换机名
        $exchange = 'direct_logs';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'direct', false, false, false);

        // 声明创建队列
        // 队列名称为空时，会生成一个随机名称队列
        [$queue, ,] = $channel->queue_declare('', false, false, true, false);

        // 绑定交换机与队列，并指定路由info
        $channel->queue_bind($queue, $exchange, 'info');
        // 绑定交换机与队列，并指定路由error
        $channel->queue_bind($queue, $exchange, 'error');
        // 绑定交换机与队列，并指定路由warning
        $channel->queue_bind($queue, $exchange, 'warning');

        // 消息消费
        $channel->basic_consume($queue, '', false, false, true, false, function ($msg) use ($output) {
            // 模拟耗时
            sleep(3);
            $output->writeln('receive:' . $msg->body . PHP_EOL);
            // 确认消息处理完成
            $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
    }
}
```

控制台分别开启C1、C2消费端

发现C1只消费了error的消息

C2消费了error、info、warning三种类型的消息

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513142559580.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513142606615.png)

# 通配符交换机（Topic exchange）

在上一节教程中，我们改进了我们的日志消息，我们使用可以选择性接收信息的*direct*类型交换机，而不是使用只能进行广播的*fanout*类型交换机。

虽然使用*direct*类型交换改进了我们的日志消息，但它仍有限制 - 它不能基于通配符匹配进行路由选择。

为了在我们的系统中实现这样的功能，我们需要学习一个更复杂的*topic exchange*。

**通配符交换机（Topic exchange）**

---

发送到 *topic exchange* 的消息不能任意命名一个*routing key* 它必须是由.划分单词列表。这些单词可以是任意的，但它们通常指定与消息相关联的一些功能。

*topic exchange*背后的逻辑类似于*direct exchange*- 但是，*routing key*有两个重要的特殊情况：

```.net
类似于正则
1. *可以代替一个单词
2. #可以匹配0个或多个单词
```

在下面的例子中简单解释一下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021051314351196.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
【a】一条以” com.register.mail”为routing key的消息将会匹配到Register Queue与SaveMail Queue两个队列上，所以消息会发送到消息接收者1和消息接收者2。routing key为“email.register.test”的消息同样会被推送到Register Queue与SaveMail Queue两个队列。

【b】如果routing key为”com.register.wsh”的话，消息只会被推送到Register Queue上；routing key为”email.com.wsh”的消息会被推送到SaveMail Queue上，routing key为"email.com.test”的消息也会被推送到SaveMail Queue上，但同一条消息只会被推送到SaveMail Queue上一次。

注意：如果在发送消息的时候没有匹配到符合条件的routing key,那么这条消息将会被废弃。如：com.register.wsh.test 消息不会被推送到Register Queue上，但是注意 email.com.wsh.test则可以推送到SaveMail Queue上。

```.net
注意事项
当routing key为 “#” 时，topic exchange如同fanout exchange
当不使用 “*” 和 “#” 特殊字符时，topic exchange如同direct exchange
```

**整合代码**

---

生产端：

```php
<?php
declare (strict_types=1);

namespace app\controller;

use app\BaseController;
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

class Topic extends BaseController
{
    public function send()
    {
        // 交换机名
        $exchange = 'topic_logs';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'topic', false, false, false);

        // 模拟发送com.register.mail信息内容
        $messageBody = 'com.register.mail, now time:' . date('h:i:s');
        // 将我们需要的消息标记为持久化 - 通过设置AMQPMessage的参数delivery_mode = AMQPMessage::DELIVERY_MODE_PERSISTENT
        $message = new AMQPMessage($messageBody, [
            'content_type' => 'text/plain',
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);

        // 绑定com.register.mail路由
        $channel->basic_publish($message, $exchange, 'com.register.mail');

        // 模拟发送email.register.test信息内容
        $messageBody = 'email.register.test, now time:' . date('h:i:s');
        // 将我们需要的消息标记为持久化 - 通过设置AMQPMessage的参数delivery_mode = AMQPMessage::DELIVERY_MODE_PERSISTENT
        $message = new AMQPMessage($messageBody, [
            'content_type' => 'text/plain',
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);

        // 绑定email.register.test路由
        $channel->basic_publish($message, $exchange, 'email.register.test');

        // 模拟发送com.register.wsh信息内容
        $messageBody = 'com.register.wsh, now time:' . date('h:i:s');
        // 将我们需要的消息标记为持久化 - 通过设置AMQPMessage的参数delivery_mode = AMQPMessage::DELIVERY_MODE_PERSISTENT
        $message = new AMQPMessage($messageBody, [
            'content_type' => 'text/plain',
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);

        // 绑定com.register.wsh路由
        $channel->basic_publish($message, $exchange, 'com.register.wsh');

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
        return 'send success';
    }
}
```

消费端C1：

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class TopicC1 extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('topicc1')
            ->setDescription('the topicc1 command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 交换机名
        $exchange = 'topic_logs';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'topic', false, false, false);

        // 声明创建队列
        // 队列名称为空时，回升后曾一个随机名称队列
        [$queue, ,] = $channel->queue_declare('', false, false, true, false);

        // 绑定交换机与队列，并指定路由*.register.*
        $channel->queue_bind($queue, $exchange, '*.register.*');

        $channel->basic_consume($queue, '', false, false, false, false, function ($msg) use ($output) {
            // 模拟耗时
            sleep(3);
            $output->writeln('received:' . $msg->body . PHP_EOL);
            // 确认消息处理完成
            $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
    }
}
```

消费端C2：

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class TopicC2 extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('topicc2')
            ->setDescription('the topicc2 command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 交换机名
        $exchange = 'topic_logs';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'topic', false, false, false);

        // 声明创建队列
        // 队列名称为空时，回升后曾一个随机名称队列
        [$queue, ,] = $channel->queue_declare('', false, false, true, false);

        // 绑定交换机与队列，并指定路由*.*.mail
        $channel->queue_bind($queue, $exchange, '*.*.mail');

        $channel->basic_consume($queue, '', false, false, false, false, function ($msg) use ($output) {
            // 模拟耗时
            sleep(3);
            $output->writeln('received:' . $msg->body . PHP_EOL);
            // 确认消息处理完成
            $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
    }
}
```

控制台分别开启C1、C2消费端

发现C1只消费了*.register.*的消息
C2消费了*.*.mail、email.#二种类型的消息
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021051315170877.png)

# 远程调用（RPC）

**RPC工作流程**

当客户端启动时，它创建一个匿名独占回调队列。

对于一个RPC请求，客户端发送一个具有两个属性的消息： *reply_to* 设置回调队列， *correlation_id* 设置每个请求的唯一值。

请求被发送到*rpc_queue*队列。

RPC服务端一直在等待队列上的请求，当请求出现时，它将执行请求的任务，并使用reply_to字段中的队列将结果返回给客户端。

客户端等待回调队列中的数据。当信息出现时，它检查correlation_id属性。如果它与请求中的值相匹配，则返回对应的应用程序的响应。

**回调队列**

---

客户端发送请求消息，服务器回复响应消息。为了收到响应，我们需要发送一个回调队列地址与请求：

```php
[$queue_name, ,] = $channel->queue_declare("", false, false, true, false);

$msg = new AMQPMessage("message", [
    'reply_to' => $queue_name
]);

$channel->basic_publish($msg, '', 'rpc_queue');
```

> **消息的属性**
> AMQP 0-9-1 协议为消息预定义了一组默认的14个属性。大多数属性很少使用，除了以下几个：
> +*delivery_mode*: 将消息标记为持久性(值为2)或者费持久(1). 你可能记得这个属性，我们在 第二节 教程中使用过。
> +*content_type*: 用于描述MIME类型的编码。例如，对于经常使用的JSON编码，将此属性设置为 “application/json” 是一个很好的做法。
> +*reply_to*: 主要用于命名一个回调队列
> +*correlation_id*: 用于将RPC响应与请求相关联

**correlation_id（关联ID）**

---

在上面提出的方法中，每个RPC请求将创建一个回调队列，这样效率非常低，我们需要为每个客户端创建同一个回调队列。

这样又引发了一个新的问题，在该队列中收到响应后，不能确定响应所属的请求，这就是 correlation_id 派上用场的时候。我们将为每个请求设置一个唯一值(correlation_id)。之后，当我们在回调队列中收到一条消息时，我们将查看此属性，并且基于此，我们能够将响应与请求向匹配。如果我们收到一个未知的 correlation_id 的值，我们就能安全的丢弃该消息 - 它不属于我们的请求。

**总结**

---

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513154202869.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)

**合并代码**

---

RPC服务端

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class RpcServer extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('rpcserver')
            ->setDescription('the rpcserver command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 队列名
        $queue = 'rpc_queue';
        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明创建队列
        $channel->queue_declare($queue, false, false, false, false);

        // 公平调度
        $channel->basic_qos(null, 1, null);

        // 消息消费
        $channel->basic_consume($queue, '', false, false, false, false, function ($msg) use ($output) {
            // 接收到RPC客户端发送的信息
            $output->writeln('received:' . $msg->body . PHP_EOL);

            // 执行方法，回调队列
            $reply = new AMQPMessage('rpc server reply message', [
                'correlation_id' => $msg->get('correlation_id')
            ]);

            $msg->delivery_info['channel']->basic_publish(
                $reply, '', $msg->get('reply_to'));

            // 确认消息处理完成
            $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
    }
}
```

RPC客户端：

```php
<?php
declare (strict_types=1);

namespace app\controller;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

class RpcClient
{
    public function call()
    {
        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();

        // 声明回调队列
        $callback_queue = $channel->queue_declare(
            'callback_queue', false, false, true, false);

        // RPC客户端请求的参数
        $corr_id = uniqid();
        $msg = new AMQPMessage('rpc client send message', [
            'correlation_id' => $corr_id,
            'reply_to' => 'callback_queue'
        ]);

        // 发送RPC请求
        $channel->basic_publish($msg, '', 'rpc_queue');

        // RPC服务端返回的内容
        $response = null;

        // 等待RPC回调
        $channel->basic_consume('callback_queue', '', false, false, false, false, function ($reply) use ($corr_id, &$response) {
            if ($reply->get('correlation_id') == $corr_id) {
                $response = $reply->body;
            }

            // 确认消息处理完成
            $reply->delivery_info['channel']->basic_ack($reply->delivery_info['delivery_tag']);
        });

        while (!$response) {
            $channel->wait();
        }

        // 输出RPC服务端返回的内容
        echo $response;

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
    }
}
```

启动服务端：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513161437987.png)
浏览器访问客户端：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513161526537.png)
返回控制台看服务器情况：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513161538135.png)

> 注意：
> 
> - 如果RPC服务处理太慢，你可以运行另一个服务来扩展。 尝试在新的控制台运行一个rpc_server
> - RPC客户端需要发送和接收一条消息。但是不需要像queue_declare这样的同步调用。

# 延迟队列、死信队列

**延迟队列**

---

延迟队列存储的对象为对应的延时消息，等待指定时间后，消费者才拿到这个消息进行消费。RabbitMQ本身没有直接支持的延迟队列功能，我们可以设置 x-message-ttl，来控制消息的生存时间，如果超时，则消息成为dead letter（死信），进入死信队列，将死信队列指定为普通的消费队列即可实现延迟消费。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210513165943474.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
**死信队列**

---

死信队列中（dead letter）死信的消息来源：

1. 消息被拒绝（basic.reject或basic.nack）并且requeue=false
2. 消息TTL过期（x-message-ttl参数值即为超时时间）
3. 队列达到最大长度（队列满了，无法再添加数据到mq中）

普通队列指定死信队列的相关参数:
x-dead-letter-exchange：用来设置死信后发送的交换机
x-dead-letter-routing-key：用来设置死信的routing key
x-message-ttl：单位毫秒，消息、队列的生命周期。超时后消息会变成dead message

**合并代码**

---

```php
<?php
declare (strict_types=1);

namespace app\controller;

use app\BaseController;
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;
use PhpAmqpLib\Wire\AMQPTable;

class Delay extends BaseController
{
    public function send()
    {
        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();

        // 创建DLX及死信队列
        $channel->exchange_declare('dlx_exchange', 'direct', false, false, false);
        $channel->queue_declare('dlx_queue', false, true, false, false);
        $channel->queue_bind('dlx_queue', 'dlx_exchange', 'dlx_routing_key');

        // 创建延迟队列
        $channel->exchange_declare('delay_exchange', 'direct', false, false, false);
        $args = new AMQPTable();
        // 消息过期方式：设置 queue.normal队列中的消息5s后过期
        $args->set('x-message-ttl', 5000);
        // 设置队列最大长度方式：x-max-length
//        $args->set('x-max-length', 1);
        $args->set('x-dead-letter-exchange', 'dlx_exchange');
        $args->set('x-dead-letter-routing-key', 'dlx_routing_key');
        $channel->queue_declare('delay_queue', false, true, false, false, false, $args);
        $channel->queue_bind('delay_queue', 'delay_exchange', 'delay_routing_key');

        // 模拟发送消息内容
        $messageBody = "该消息将在5s后发送到延迟队列（" . date("h:i:s") . "）";
        // 将我们需要的消息标记为持久化
        $message = new AMQPMessage($messageBody, [
            'content_type' => 'text/plain',
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);

        // 绑定delay_routing_key路由
        $channel->basic_publish($message, 'delay_exchange', 'delay_routing_key');

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();

        return 'send success';
    }


}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021051409485730.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)
5秒之后，消息会过期，然后被进入 dlx_exchange, 进而路由到 dlx_queue 队列中：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210514095005313.png)
消费者代码

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class Delay extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('delay')
            ->setDescription('the delay command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 交换机名
        $exchange = 'dlx_exchange';

        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'direct', false, false, false);
        $channel->queue_declare('dlx_queue', false, true, false, false);
        $channel->queue_bind('dlx_queue', $exchange, 'dlx_routing_key');

        // 消息消费
        $channel->basic_consume('dlx_queue', '', false, false, false, false, function ($msg) use ($output) {
//            $body = iconv('UTF-8', 'GB2312//IGNORE', $msg->body);
            $output->writeln(date('h:i:s') . 'received:' . $msg->body . PHP_EOL);

            // 确认消息处理完成
            $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();


    }
}
```

运行消费者代码之后，消息会从dlx_queue中消费掉：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021051409505622.png)

# 重试队列（可靠性投递，重试超过3次，入库告警）

**消息消费确认实现原理**

当消费者的消息消费异常时，消息进入延迟重试队列，待超时后重新发送到重试队列指定的死信队列，死信队列重新消费信息，如果又出现死信情况，继续进入延时重试队列，依次循环，当重试超过3次后，则入库告警，等待人工处理。

**生产者代码**

```php
<?php
declare (strict_types=1);

namespace app\controller;

use app\BaseController;
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;
use PhpAmqpLib\Wire\AMQPTable;

class Retry extends BaseController
{
    public function send()
    {
        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();

        // 创建DLX及死信队列
        $channel->exchange_declare('dlx_exchange', 'direct', false, false, false);
        $channel->queue_declare('dlx_queue', false, true, false, false);
        $channel->queue_bind('dlx_queue', 'dlx_exchange', 'dlx_routing_key');

        // 创建延迟队列
        $channel->exchange_declare('delay_exchange', 'direct', false, false, false);
        $args = new AMQPTable();
        // 消息过期方式：设置queue.normal队列中的消息5s之后过期
        $args->set('x-message-ttl', 5000);
        // 设置队列最大长度方式： x-max-length
//        $args->set('x-max-length', 1);
        $args->set('x-dead-letter-exchange', 'dlx_exchange');
        $args->set('x-dead-letter-routing-key', 'dlx_routing_key');
        $channel->queue_declare('delay_queue', false, true, false, false, false, $args);
        $channel->queue_bind('delay_queue', 'delay_exchange', 'delay_routing_key');

        // 模拟发送消息内容
        $messageBody = "该消息将在5s后发送到延迟队列（" . date("h:i:s") . "）";
        // 将我们需要的消息标记为持久化
        $message = new AMQPMessage($messageBody, [
            'content_type' => 'text/plain',
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);

        // 设置重试次数，默认0
        $headers = new AMQPTable([
            'retry_nums' => 1
        ]);

        $message->set('application_headers', $headers);

        // 绑定delay.routing_key路由
        $channel->basic_publish($message, 'delay_exchange', 'delay_routing_key');

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
        return 'send success';
    }


}
```

**消费者代码**

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Wire\AMQPTable;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class Retry extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('retry')
            ->setDescription('the retry command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 交换机名
        $exchange = 'dlx_exchange';
        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare($exchange, 'direct', false, false, false);
        $channel->queue_declare('dlx_queue', false, true, false, false);
        $channel->queue_bind('dlx_queue', $exchange, 'dlx_routing_key');

        // 创建重试队列
        $channel->exchange_declare('delay_exchange', 'direct', false, false, false);

        // 消息消费
        $channel->basic_consume('dlx_queue', '', false, false, false, false, function ($msg) use ($output) {
//            $body = iconv("UTF-8", "GB2312//IGNORE", $msg->body);
            $output->writeln(date("h:i:s") . " Received " . $msg->body . PHP_EOL);

            // 确认消息处理完成
            $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);

            $msg_headers = $msg->get('application_headers')->getNativeData();

            // 重试次数超过3次，则入库告警
            if (intval($msg_headers['retry_nums']) > 3) {
//                $body = iconv("UTF-8", "GB2312//IGNORE", "重试次数超过3次，则入库告警");
                $output->writeln(date("h:i:s") . " Error " . "重试次数超过3次，则入库告警" . PHP_EOL);
            } else {
                // 建立连接
                $connection = new AMQPStreamConnection('127.0.0.1', '5672', 'guest', 'guest', '/');
                // 获取信道
                $channel = $connection->channel();
                // 重试次数加1
                $headers = new AMQPTable([
                    "retry_nums" => intval($msg_headers['retry_nums']) + 1
                ]);
                $msg->set('application_headers', $headers);

                // 放回重试队列
                $channel->basic_publish($msg, "delay_exchange", "delay_routing_key");
            }
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
    }
}
```

运行消费者代码之后：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210514103458790.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6YnNrYWs=,size_16,color_FFFFFF,t_70)

# 消费幂等

由于网络闪断、MQ Broker端异常等原因可能导致消费端回送的ask确认消息失败或者异常，导致消费端再次收到实际已经消费的消息

> 解决思路
> 每个消息增加一个消息ID，消费端消费前读取缓存，查看消息ID是否存在，不存在则直接return，存在则消费，消费成功删除缓存数据，实现消费的幂等。

**生产者代码**

```php
<?php
declare (strict_types=1);

namespace app\controller;

use app\BaseController;
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;
use PhpAmqpLib\Wire\AMQPTable;
use think\facade\Cache;

class Idempotent extends BaseController
{
    public function send()
    {
        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', 5672, 'zq', '123456', '/');
        // 获取信道
        $channel = $connection->channel();

        // 创建DLX及死信队列
        $channel->exchange_declare('dlx_exchange', 'direct', false, false, false);
        $channel->queue_declare('dlx_queue', false, true, false, false);
        $channel->queue_bind('dlx_queue', 'dlx_exchange', 'dlx_routing_key');

        // 创建延迟队列
        $channel->exchange_declare('delay_exchange', 'direct', false, false, false);
        $args = new AMQPTable();
        // 消息过期方式：设置queue.normal队列中的消息5s之后过期
        $args->set('x-message-ttl', 5000);
        // 设置队列最大长度方式： x-max-length
//        $args->set('x-max-length', 1);
        $args->set('x-dead-letter-exchange', 'dlx_exchange');
        $args->set('x-dead-letter-routing-key', 'dlx_routing_key');
        $channel->queue_declare('delay_queue', false, true, false, false, false, $args);
        $channel->queue_bind('delay_queue', 'delay_exchange', 'delay_routing_key');

        // 模拟发送消息内容
        $messageBody = "重复消息发送（" . date("h:i:s") . "）";
        // 将我们需要的消息标记为持久化
        $message = new AMQPMessage($messageBody, [
            'content_type' => 'text/plain',
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);

        // 设置消息ID，防止重复消费
        $corr_id = uniqid();
        $headers = new AMQPTable([
            'correlation_id' => $corr_id
        ]);

        $message->set('application_headers', $headers);

        Cache::set($corr_id, $corr_id, 3600);

        // 绑定delay.routingKey路由
        $channel->basic_publish($message, 'delay_exchange', 'delay_routing_key');

        // 模拟发送重复消息
        $channel->basic_publish($message, 'delay_exchange', 'delay_routing_key');

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
        return 'send success';
    }
}
```

**消费者代码**

```php
<?php
declare (strict_types=1);

namespace app\command;

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Wire\AMQPTable;
use think\console\Command;
use think\console\Input;
use think\console\Output;
use think\facade\Cache;

class Idempotent extends Command
{
    protected function configure()
    {
        // 指令配置
        $this->setName('idempotent')
            ->setDescription('the idempotent command');
    }

    protected function execute(Input $input, Output $output)
    {
        // 建立连接
        $connection = new AMQPStreamConnection('127.0.0.1', 5672, 'zq', '123456', '/');
        // 获取信道
        $channel = $connection->channel();
        // 声明交换机
        $channel->exchange_declare('dlx_exchange', 'direct', false, false, false);
        $channel->queue_declare('dlx_queue', false, true, false, false);
        $channel->queue_bind('dlx_queue', 'dlx_exchange', 'dlx_routing_key');

        // 创建重试队列
        $channel->exchange_declare('delay_exchange', 'direct', false, false, false);
        $args = new AMQPTable();
        // 消息过期方式：设置queue.normal队列中的消息5s之后过期
        $args->set('x-message-ttl', 5000);
        // 设置队列最大长度方式： x-max-length
//        $args->set('x-max-length', 1);
        $args->set('x-dead-letter-exchange', 'dlx_exchange');
        $args->set('x-dead-letter-routing-key', 'dlx_routing_key');
        $channel->queue_declare('delay_queue', false, true, false, false, false, $args);
        $channel->queue_bind('delay_queue', 'delay_exchange', 'delay_routing_key');

        // 消息消费
        $channel->basic_consume('dlx_queue', '', false, false, false, false, function ($msg) use ($output) {
            $msg_headers = $msg->get('application_headers')->getNativeData();
            $corr_id = $msg_headers['correlation_id'];

            // 判断是否已经消费过
            if (Cache::get($corr_id) === null) {
//                $body = iconv("UTF-8", "GB2312//IGNORE", "该消息已消费，不再消费");
                $output->writeln(date("h:i:s") . '该消息已消费，不再消费' . PHP_EOL);
                // 确认消息处理完成
                $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
                return;
            }

//            $body = iconv("UTF-8", "GB2312//IGNORE", $msg->body);
            $output->writeln(date("h:i:s") . " Received " . $msg->body . PHP_EOL);

            // 确认消息处理完成
            $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
            Cache::delete($corr_id);
        });

        while (count($channel->callbacks)) {
            $channel->wait();
        }

        // 关闭信道
        $channel->close();
        // 关闭连接
        $connection->close();
    }
}
```

运行消费者代码之后：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210514110537690.png)