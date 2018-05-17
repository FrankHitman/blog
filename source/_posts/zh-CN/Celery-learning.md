---
title: Celery learning
lang: zh-CN
date: 2018-05-07 11:08:17
tags: Celery
---
- 为什么选择celery？
1. 可以后台运行任务。
2. 在请求完成后，执行后续的任务
3. 通过异步执行和重试保证任务的完成
4. 定时任务
5. 。。。等
<!-- more -->
- celery支持的序列化方式有哪些？
包括 JSON、YAML、Pickle和msgpack，在4.0版本之前默认是Pickle，4.0版本之后是JSON。
如果跨语言使用celery，建议不使用Pickle。如果传参是复杂的python对象，使用Pickle。
- celery需要的broker有哪些？
包括RabbitMQ、redis、SQS和Qpid，redis作为broker会由于强制终止或者断电丢失数据，Rabbitmq是推荐使用的。
我们这里使用的是redis。
- 如何需要获取任务的执行结果，该如何配置？
需要配置backend，用于存储result。包括db、memcached、redis和Rabbitmq
我们这里使用的是redis
- 设计图如下所示
```sequence
生产者->broker(redis):消息
Note right of broker(redis): 2891条消息在队列中等待被消费
broker(redis)->消费者(worker):消息
Note right of 消费者(worker): 2个worker在执行消息
消费者(worker)-->broker(redis):acknowledged
消费者(worker)->redis:result

```
理解的Celery消息任务时序图如下：
```sequence
生产者->broker(队列/AMQP/RabbitMQ):消息
Note right of broker(队列/AMQP/RabbitMQ): 2891条消息在队列中等待被消费
broker(队列/AMQP/RabbitMQ)->消费者(worker):消息
Note right of 消费者(worker): 2个worker在执行消息
消费者(worker)-->broker(队列/AMQP/RabbitMQ):acknowledged
消费者(worker)->redis/mysql/rpc:result

```


有点尴尬，删掉流程图会导致上图不显示：
```flow
tag=type:content:url
s=start:开始e=end:结束o=operation:操作项s-o-e

st=>start: Start
e=>end
op=>operation: My Operation
cond=>condition: Yes or No?

st->op->cond
cond(yes)->e
cond(no)->op
```

