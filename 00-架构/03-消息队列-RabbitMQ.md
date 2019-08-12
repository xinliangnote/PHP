## RabbitMQ

#### 简介

RabbitMQ 是一个开源的消息代理和队列服务器，用来通过普通协议在完全不同的应用之间共享数据，RabbitMQ 是使用 Erlang 语言编写的，是基于 AMQP 协议的。

`开源`、`性能优秀`、`稳定`

集群模式丰富，表达式配置，HA模式，镜像队列模型。

保证数据不丢失的前提做到高可靠行、可用性。

AMQP：Advanced Message Queuing Protocol 高级消息队列协议。

#### 流程图

![](https://github.com/xinliangnote/PHP/blob/master/images/rabbitMQ.jpg)

#### 核心概念

- Producer：消息生产者。

- Broker：接受客户端的连接，实现 AMQP 实体服务。
- Connection：连接，应用程序与 Broker 的网络连接。
- Channel：网络信道，几乎所有的操作都在 Channel 中进行，Channel 是进行消息读写的通道。客户端可以建立多个 Channel ，每个 Channel 代表一个会话任务。
- Message：消息，服务器和应用程序之间传送的数据，由 Properties 和 Body 总成。Properties 可以对消息进行修饰，比如消息的优先级、延迟等高级特性；Body 就是消息体的内容。
- Virtual Host：虚拟地址，用于进行逻辑隔离，最上层的消息路由。一个 Virtual Host 里面可以有若干个 Exchange 和 Queue，同一个 Virtual Host 里面不能有相同名称的 Exchange 和 Queue。
- Exchange：交换机，接受消息，根据路由键转发消息到绑定的队列。
- Binding：Exchange 和 Queue 之间的虚拟链接，binding中可以包含 routing key。
- Routing key：一个路由规则，虚拟机可用它来确定如何路由如何路由一个特定消息。
- Queue：也称为 Message Queue，消息队列，保存消息并将它们转发给消费者。
- Consumer：消息消费者。

#### 保障 100% 消息投递成功设计方案

- 消息入库，对消息状态进行打标记（发出一个标记，接收一个标记）
- 消费者实现幂等性。

#### PHP

- AMQP 扩展：http://pecl.php.net/package/amqp
- Kafka PHP Client：https://github.com/weiboad/kafka-php