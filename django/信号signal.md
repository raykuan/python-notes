### django内置的signal
```
Model signals
    pre_init                    #django的modal执行其构造方法前，自动触发
    post_init                   #django的modal执行其构造方法后，自动触发
    pre_save                    #django的modal对象保存前，自动触发
    post_save                   #django的modal对象保存后，自动触发
    pre_delete                  #django的modal对象删除前，自动触发
    post_delete                 #django的modal对象删除后，自动触发
    m2m_changed                 #django的modal中使用m2m字段操作第三张表（add,remove,clear）前后，自动触发
    class_prepared              #程序启动时，检测已注册的app中modal类，对于每一个类，自动触发
Management signals
    pre_migrate                 #执行migrate命令前，自动触发
    post_migrate                #执行migrate命令后，自动触发
Request/response signals
    request_started             #请求到来前，自动触发
    request_finished            #请求结束后，自动触发
    got_request_exception       #请求异常后，自动触发
Test signals
    setting_changed             #使用test测试修改配置文件时，自动触发
    template_rendered           #使用test测试渲染模板时，自动触发
Database Wrappers
    connection_created          #创建数据库连接时，自动触发
```
    
1、 对于Django内置的信号，仅需注册指定信号，当程序执行相应操作时，自动触发注册函数:
```
    from django.core.signals import request_finished
    from django.core.signals import request_started
    from django.core.signals import got_request_exception
    from django.db.models.signals import class_prepared
    from django.db.models.signals import pre_init, post_init
    from django.db.models.signals import pre_save, post_save
    from django.db.models.signals import pre_delete, post_delete
    from django.db.models.signals import m2m_changed
    from django.db.models.signals import pre_migrate, post_migrate
    from django.test.signals import setting_changed
    from django.test.signals import template_rendered
    from django.db.backends.signals import connection_created


    def callback(sender, **kwargs):
        print("xxoo_callback")
        print(sender,kwargs)

    xxoo.connect(callback)
    # xxoo指上述导入的内容
```
2、 dispatch方式注册:
```
from django.core.signals import request_finished
from django.dispatch import receiver

@receiver(request_finished)
def my_callback(sender, **kwargs):
    print("Request finished!")
```

### 自定义信号

1、定义信号
```
import django.dispatch
pizza_done = django.dispatch.Signal(providing_args=["toppings", "size"])
```

2、注册信号
```
def callback(sender, **kwargs):
    print("callback")
    print(sender,kwargs)
 
pizza_done.connect(callback)
```

3、触发信号
```
from 路径 import pizza_done
 
pizza_done.send(sender='seven',toppings=123, size=456)
```
