---
title: Celery learning
lang: zh-CN
date: 2018-05-07 11:08:17
categories:
    - python
tags: Celery
---
## celery基本介绍
### 为什么选择celery？
1. 可以后台运行任务。
2. 在请求完成后，执行后续的任务
3. 通过异步执行和重试保证任务的完成
4. 定时任务
5. 。。。等
<!-- more -->
### celery支持的序列化方式有哪些？
包括 JSON、YAML、Pickle和msgpack，在4.0版本之前默认是Pickle，4.0版本之后是JSON。
如果跨语言使用celery，建议不使用Pickle。如果传参是复杂的python对象，使用Pickle。
### celery需要的broker有哪些？
包括RabbitMQ、redis、SQS和Qpid，redis作为broker会由于强制终止或者断电丢失数据，Rabbitmq是推荐使用的。
我们这里使用的是redis。
### 如果需要获取任务的执行结果，该如何配置？
需要配置backend，用于存储result。包括db、memcached、redis和Rabbitmq
我们这里使用的是redis
### 本项目异步任务使用的架构
- celery运行时序图如下所示：
```sequence
生产者->broker(redis):消息
Note right of broker(redis): 2891条消息在队列中等待被消费
broker(redis)->消费者(worker):消息
Note right of 消费者(worker): 2个worker在执行消息
消费者(worker)-->broker(redis):acknowledged
消费者(worker)->redis:result
```
- 本项目异步任务流程图如下所示：
```flow
tag=type:content:>url
s=start:开始e=end:结束o=operation:操作项s-o-e

st=>start: 开始:>http://www.google.com[blank]
e=>end: 结束
request=>inputoutput: 请求
api=>operation: Api接口
db=>operation: 数据库CRUD操作
cn=>condition: 同时开启异步
celery=>operation: 异步任务
cel_res=>condition: 异步执行结果
response=>inputoutput: 响应
msg=>inputoutput: 推送消息给用户
db2=>operation: 数据库更新操作
db3=>operation: 数据库更新操作

st->request->db
db->cn
cn(yes,right)->celery
cn(no)->response
celery(right)->cel_res
cel_res(yes,right)->db2
cel_res(no)->db3->msg
msg(left)->e
response->e
db2->e
```

## 使用示例介绍
### 项目目录结构
```bash
.
├── api
│   ├── compute
│   ├── libs
│   ├── network
│   │   ├── output_fields
│   │   ├── reqparsers
│   │   ├── resources
│   │   └── tests
│   └── tests
├── common
│   ├── compute
│   └── network
├── config
├── db
│   ├── data
│   └── schema
├── deploy
├── docs
├── job
│   ├── compute
│   └── network
├── models
│   ├── compute
│   └── network
└── tasks
    ├── compute
    └── network

```
说明：异步任务使用到的目录就两个，一个是config，另外一个是tasks。config的__init__.py文件记录celery的配置信息，tasks则是定义一个个异步任务模块，也就是定义消费者。那么生产者在哪里呢？生产者就是在api目录中的views中，后面会细讲。
### 安装所需环境
```bash
#切换到neocu-business目录下，新建虚拟环境并安装第三方包
virtualenv .env
source .env/bin/activate
pip install -r requirements.txt 

```

### 关于配置
本项目配置redis作为broker和result backend。CELERY_INCLUDE 用于指明task中所有的worker的路径，新增模块的任务都需要在这里标明下，否则启动celery的worker时候，没有消费者执行任务。其他的配置项目参考[官方文档](http://docs.celeryproject.org/en/latest/userguide/configuration.html)，例如CELERY_RESULT_SERIALIZER和CELERY_TASK_SERIALIZER用于设置序列化方式，CELERY_TASK_ANNOTATIONS设置任务执行速率等。配置如下
```python
NB_REDIS = 'redis://127.0.0.1:6379'
BROKER_URL = NB_REDIS
CELERY_RESULT_BACKEND = NB_REDIS
CELERY_INCLUDE = ['tasks.network.loadbalancers']

```

### celery app
celery的app的在tasks/__init__.py中，celery启动的入口，并且作为纽带连接tasks和api
```python
from celery import Celery
import config

def make_celery():
    celery_obj = Celery(
        'neocu_business',
        backend=config.CELERY_RESULT_BACKEND,
        broker=config.BROKER_URL
    )
    celery_obj.conf.update(
        CELERY_INCLUDE=config.CELERY_INCLUDE,
    )

    return celery_obj
celery = make_celery()
```

### celery tasks
以负载均衡为例，我们在tasks/network目录下面新建loadbalancers.py文件，这里以创建负载均衡的异步任务为例：
```python
from tasks import celery
@celery.task()
def create(token, data):
    user = User(token)
    user.service_type = ServiceType.Neutron
    cache_id = data.pop('cache_id')
    resp = user.requests.post(Paths.lb_path, json=data)
    if resp.success:
        data = resp.json()
        loadbalancer = data.get('loadbalancer', {})
        Loadbalancers.update(cache_id=cache_id, **loadbalancer)
    else:
        with DBSession() as session:
            query = session.query(Loadbalancers).filter(
                Loadbalancers.id == cache_id,
            ).first()
            if query:
                session.delete(query)
                session.commit()
        # todo: push msg
    return resp.status_code
```
因为4.0之后版本的celery序列化方式默认为json，所以传参中我们不能用python对象，上面代码中token和data是基本数据类型。
token是keystone中给登录后用户生成的唯一识别码TOKEN_ID，data是用于异步请求底层api的data。如果请求成功了，那么改写数据库中的记录与底层一致，否则删除创建的数据记录

### celery producer 
同样以负载均衡为例，在api/network/resources/loadbalancers.py中创建负载均衡的api接口如下：
```python
from tasks.network.loadbalancers import create
    
class LoadbalancerListResource(NBResource):

    def post(self):
        args = parser_lb.parse_args(strict=True)
        item_args = parser_lb_item_post.parse_args(req=args)
        data = item_args.copy()
        token = request.headers.get('X_Auth_Token')
        cache_id = 'cache_' + str(uuid.uuid4())
        data.update({
            'provisioning_status': LoadbalancerStatus.PendingCreate,
            'region_id': User(token).region_id,
            'id': cache_id,
        })
        data['provisioning_status'] = LoadbalancerStatus.PendingCreate
        Loadbalancers.add(**data)
        data = {
            'loadbalancer': item_args,
            'cache_id': cache_id,
        }
        create.delay(token, data)
        return cache_id, 201
```
在接收到用户输入，存入数据库中后，调用异步任务`create.delay(token, data)`执行真正的请求，然后直接返回response给用户。

### 启动 celery
在neocu-business目录下运行`celery -A tasks.celery worker -l info`，启动成功如下：
```bash
 -------------- celery@localhost.localdomain v4.1.0 (latentcall)
---- **** ----- 
--- * ***  * -- Linux-3.10.0-693.5.2.el7.x86_64-x86_64-with-centos-7.4.1708-Core 2018-05-31 14:49:31
-- * - **** --- 
- ** ---------- [config]
- ** ---------- .> app:         neocu_business:0x7fb85620f410
- ** ---------- .> transport:   redis://127.0.0.1:6379//
- ** ---------- .> results:     redis://127.0.0.1:6379/
- *** --- * --- .> concurrency: 2 (prefork)
-- ******* ---- .> task events: OFF (enable -E to monitor tasks in this worker)
--- ***** ----- 
 -------------- [queues]
                .> celery           exchange=celery(direct) key=celery
                
[tasks]
  . tasks.network.loadbalancers.add
  . tasks.network.loadbalancers.create
  . tasks.network.loadbalancers.destroy
  . tasks.network.loadbalancers.update

[2018-05-31 14:49:31,883: INFO/MainProcess] Connected to redis://127.0.0.1:6379//
[2018-05-31 14:49:31,890: INFO/MainProcess] mingle: searching for neighbors
[2018-05-31 14:49:32,909: INFO/MainProcess] mingle: all alone
[2018-05-31 14:49:32,918: INFO/MainProcess] celery@localhost.localdomain ready.

```
可以看到启动的celery中有四个类型的tasks等待执行任务，然后利用postman执行api的创建操作，检验异步任务的执行结果。命令中的`-l info`作用：如果task中的代码执行错误会在console中有日志。如下所示
```bash
[2018-05-31 16:31:35,861: INFO/MainProcess] Received task: tasks.network.loadbalancers.update[a5afef6e-1ef0-4dfb-bf82-555adfe50c06]  
[2018-05-31 16:31:40,383: INFO/ForkPoolWorker-1] Task tasks.network.loadbalancers.update[a5afef6e-1ef0-4dfb-bf82-555adfe50c06] succeeded in 0.605822082987s: 200

```

## 基于Celery官方文档画的时序图
理解的Celery消息任务时序图如下：
```sequence
生产者->broker(队列/AMQP/RabbitMQ):消息
Note right of broker(队列/AMQP/RabbitMQ): 2891条消息在队列中等待被消费
broker(队列/AMQP/RabbitMQ)->消费者(worker):消息
Note right of 消费者(worker): 2个worker在执行消息
消费者(worker)-->broker(队列/AMQP/RabbitMQ):acknowledged
消费者(worker)->redis/mysql/rpc:result
```

