# 前言

RabbitMQ 是实现了高级消息队列协议（AMQP）的开源消息代理软件（亦称面向消息的中间件）。RabbitMQ服务器是用 `Erlang` 语言编写的，而群集和故障转
移是构建在开放电信平台框架上的。所有主要的编程语言均有与代理接口通讯的客户端库。

因此，想要安装 RabbitMQ 首先需要安装 `Erlang` 环境。笔者安装后发现 `RabbitMQ` 安装起来比较简单，不过在安装 `Erlang` 环境时会遇到各种坑。

所有，笔者将两者的安装分开介绍。

+ [Erlang 环境配置](./erlang-install.md)
+ [安装 RabbitMQ](./rabbitmq-install.md)