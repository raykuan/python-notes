### django内置的signal
```
Model signals
    pre_init                    #django的model执行其构造方法前，自动触发
    post_init                   #django的model执行其构造方法后，自动触发
    pre_save                    #django的model对象保存前，自动触发
    post_save                   #django的model对象保存后，自动触发
    pre_delete                  #django的model对象删除前，自动触发
    post_delete                 #django的model对象删除后，自动触发
    m2m_changed                 #django的model中使用m2m字段操作第三张表（add,remove,clear）前后，自动触发
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

### 示例
```
class SmsCode(models.Model):
    SMS_TYPE_CHOINCES = ((1, '注册'), (2, '忘记密码'),)
    SEND_STATUS_CHOINCES = ((1, '发送成功'),(2, '发送失败'),)

    sms_type = models.IntegerField('验证码类型', choices=SMS_TYPE_CHOINCES)
    mobile = models.CharField('手机号', max_length=11)
    device_sn = models.CharField('设备序列号', max_length=64)
    sms_code = models.CharField('验证码', max_length=6, default=get_sms_code)
    send_status = models.IntegerField('发送状态', choices=SEND_STATUS_CHOINCES, default=1)
    used = models.BooleanField('是否被使用', default=False)
    create_time = models.DateTimeField('创建时间', auto_now_add=True)
    expire_time = models.DateTimeField('过期时间', null=True)

    class Meta:
        verbose_name = '手机验证码'
        verbose_name_plural = verbose_name
        db_table = "wiseu_smscode"

    def save(self, *args, **kwargs):
        self.create_time = timezone.now()
        self.expire_time = self.create_time + datetime.timedelta(0, 300)
        super(SmsCode, self).save(*args, **kwargs)


def send_sms_code(sender, instance, *args, **kwargs):
    # 如果当前实例_disable_post_save的属性是True就不执行post_save
    # 防止后面的instance.save()执行时陷入循环调用
    if getattr(instance, '_disable_post_save', False) is True:
        return
    res = smsapi.send_sms_ali(instance.mobile, instance.sms_code)
    if res != 1:
        log.error("%s短信发送失败：%s" % (instance.mobile, res))
        print("%s短信发送失败：%s" % (instance.mobile, res))
        instance.send_status = 2
        # 修改短信发送状态之后把当前实例_disable_post_save = True
        setattr(instance, '_disable_post_save', True)
        instance.save()

post_save.connect(send_sms_code, sender=SmsCode)


在view中如果对调用实例的save方法，同样可以使用上面的setattr属性防止重复执行
def sms_validate(self, request, sms_type):
       sms_serializer = SmsRetrieveSerializer(data=request.data)
       sms_serializer.is_valid(raise_exception=True)
       sms_code = SmsCode.objects.filter(
           mobile=request.data.get('mobile'), sms_code=request.data.get('sms_code'), sms_type=sms_type).last()
       # import ipdb; ipdb.set_trace()
       if not sms_code:
           raise exceptions.ValidationError({'mobile': ['手机号或验证码错误']})
       sms_code.used = True
       # 设置当前实例_disable_post_save属性为True防止save的时间触发post_save
       sms_code._disable_post_save = True
       sms_code.save()
```
