---
title: Celery learning
lang: zh-CN
date: 2018-05-07 11:08:17
tags: Celery
---

理解的Celery消息任务时序图如下：
```sequence
生产者->broker(队列/AMQP/RabbitMQ):消息
Note right of broker(队列/AMQP/RabbitMQ): 2891条消息在队列中等待被消费
broker(队列/AMQP/RabbitMQ)->消费者(worker):消息
Note right of 消费者(worker): 2个worker在执行消息
消费者(worker)-->broker(队列/AMQP/RabbitMQ):acknowledged
消费者(worker)->redis/mysql/rpc:result

```
<!-- more -->
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

