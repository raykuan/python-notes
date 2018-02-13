
## celery配置实例

摘抄至http://blog.csdn.net/u011350541/article/details/72885550
```
#!/usr/bin/env python  
import random  
from kombu import serialization  
from kombu import Exchange, Queue  
import ansibleService  
  
serialization.registry._decoders.pop("application/x-python-serialize")  
  
broker_url = ansibleService.getConfig('/etc/ansible/rabbitmq.cfg', 'rabbit', 'broker_url')  
celeryMq = ansibleService.getConfig('/etc/ansible/rabbitmq.cfg', 'celerymq', 'celerymq')  
  
SECRET_KEY='top-secrity'  
CELERY_BROKER_URL = broker_url  
CELERY_RESULT_BACKEND = broker_url  
CELERY_TASK_RESULT_EXPIRES = 1200  
CELERYD_PREFETCH_MULTIPLIER = 4  
CELERYD_CONCURRENCY = 1  
CELERYD_MAX_TASKS_PER_CHILD = 1  
CELERY_TIMEZONE = 'CST'  
CELERY_TASK_SERIALIZER='json'  
CELERY_ACCEPT_CONTENT=['json']  
CELERY_RESULT_SERIALIZER='json'  
CELERY_QUEUES = (  
    Queue(celeryMq, Exchange(celeryMq), routing_key=celeryMq),  
)  
CELERY_IGNORE_RESULT = True  
CELERY_SEND_EVENTS = False  
CELERY_EVENT_QUEUE_EXPIRES = 60
```
rmq作为消息队列, 并发worker数25, 每个worker最多执行一个任务就销毁(执行完进程销毁重建，释放内存)

```
# -*- coding:utf-8 -*-                                                                                                                                                    
from datetime import timedelta  
from settings import REDIS_HOST, REDIS_PORT, REDIS_PASSWORD, REDIS_DB_NUM  
  
  
# 某个程序中出现的队列，在broker中不存在，则立刻创建它  
CELERY_CREATE_MISSING_QUEUES = True  
  
CELERY_IMPORTS = ("async_task.tasks", "async_task.notify")  
  
# 使用redis 作为任务队列  
BROKER_URL = 'redis://:' + REDIS_PASSWORD + '@' + REDIS_HOST + ':' + str(REDIS_PORT) + '/' + str(REDIS_DB_NUM)  
  
#CELERY_RESULT_BACKEND = 'redis://:' + REDIS_PASSWORD + '@' + REDIS_HOST + ':' + str(REDIS_PORT) + '/10'  
  
CELERYD_CONCURRENCY = 20  # 并发worker数  
  
CELERY_TIMEZONE = 'Asia/Shanghai'  
  
CELERYD_FORCE_EXECV = True    # 非常重要,有些情况下可以防止死锁  
  
CELERYD_PREFETCH_MULTIPLIER = 1  
  
CELERYD_MAX_TASKS_PER_CHILD = 100    # 每个worker最多执行万100个任务就会被销毁，可防止内存泄露  
# CELERYD_TASK_TIME_LIMIT = 60       # 单个任务的运行时间不超过此值，否则会被SIGKILL 信号杀死   
# BROKER_TRANSPORT_OPTIONS = {'visibility_timeout': 90}  

# 任务发出后，经过一段时间还未收到acknowledge , 就将任务重新交给其他worker执行  
CELERY_DISABLE_RATE_LIMITS = True     
  
# 定时任务  
CELERYBEAT_SCHEDULE = {  
    'msg_notify': {  
        'task': 'async_task.notify.msg_notify',  
        'schedule': timedelta(seconds=10),  
        #'args': (redis_db),  
        'options' : {'queue':'my_period_task'}  
    },  
    'report_result': {  
        'task': 'async_task.tasks.report_result',  
        'schedule': timedelta(seconds=10),  
      #'args': (redis_db),  
        'options' : {'queue':'my_period_task'}  
    },  
    #'report_retry': {  
    #    'task': 'async_task.tasks.report_retry',  
    #    'schedule': timedelta(seconds=60),  
    #    'options' : {'queue':'my_period_task'}  
    #},  
  
}  
################################################  
# 启动worker的命令  
# *** 定时器 ***  
# nohup celery beat -s /var/log/boas/celerybeat-schedule  --logfile=/var/log/boas/celerybeat.log  -l info &  
# *** worker ***  
# nohup celery worker -f /var/log/boas/boas_celery.log -l INFO &  
################################################
```

注意：  

CELERYD_TASK_TIME_LIMIT 设置的过小，会导致task还没有执行完，worker就被杀死  

BROKER_TRANSPORT_OPTIONS 设置的过小，task有可能被多次反复执行  
