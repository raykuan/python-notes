## 安装步骤

### 1. celery和django-celery的区别

celery是原生的celery库，而django-celery是对celery进行了封装方便在django中使用    

根据自己的需求选择使用celery和django-celery  

django-celery可以实现类似Linux crontab任务调度的功能


### 2. pip安装celery

```
/env/pip install celery
```


## 配置步骤

### 1. django项目settings.py中关于Celery的配置项
```
# Settings for Celery
# http://docs.celeryproject.org/en/latest/userguide/configuration.html#new-lowercase-settings

CELERY_ENABLE_UTC = False
CELERY_TIMEZONE = 'Asia/Shanghai'
BROKER_URL = 'redis://10.213.32.36:6379/1'   # celery broker 消息中间件的配置(一般使用rabbitmq或redis)
CELERY_RESULT_BACKEND = 'redis://10.213.32.36:6379/2'  # backend是celery执行tasks结果保存的地方(可以是mysql等数据库)
CELERY_TASK_RESULT_EXPIRES = 3600  # CELERY_RESULT_BACKEND中task执行结果的保存时间，默认是86400秒
```

### 2. 在django project配件文件目录下新建celery.py
```
# 从future模块导入absolute_import，防止celery.py模块就不会与Celery库相冲突
from __future__ import absolute_import
import os
from celery import Celery
from django.conf import settings


# set the default Django settings module for the 'celery' program.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'wiseops.settings')

# create celery application
app = Celery('wiseops')

# Using a string here means the worker will not have to pickle the object when using Windows.

# 此处会加载settings文件中关于celery的配置项
app.config_from_object('django.conf:settings')

# 此处会加载项目所有APP中tasks.py函数
app.autodiscover_tasks(lambda: settings.INSTALLED_APPS) 


@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))

```

### 3. 在django app目录下新建tasks.py (当前APP中需要异步执行的函数都写在此文件中)
```
from __future__ import absolute_import
from common.custom.sendmail import SendEmail
from celery import shared_task


@shared_task
def send_reset_pass_email(to, username, code):
    sm = SendEmail()
    to = [to, ]
    subject = '验证邮件 [重置AD域用户密码申请]'
    text = '<h3>%s 验证码有效时间5分钟!</h3>验证码: <span style="color:red;">%s</span>' % (username, code)
    sm.send_mail(to, subject, text)

```

### 4. 在django view中使用tasks.py异步函数
```
from app import tasks

class UserResetADUserPassSendEmailAPIView(APIView):
    """重置AD域用户密码发送验证邮件接口"""
    def get(self, request):
        username = request.query_params.get('username', None)
        pattern = r'.*(?P<special>[~!@#$%^&*_\-+=`|\\(){}\[\]:;"\'<>,.?/]).*'
        match = re.match(pattern, username)
        if match:
            raise exceptions.ValidationError({'username': ['用户名不能包含特殊字符']})
        if username:
            res = get_ad_user_status(username)
            code = res.get('code')
            if code == '200':
                try:
                    code = cache.get('email_vercode_'+username)
                    if not code:
                        code = get_verify_code()
                    # 设置验证码缓存
                    to = res['data'].get('mail')
                    name = res['data'].get('user_displayname')
                    
                    # **************celery异步发送邮件(fun_name.delay调用函数) ************
                    tasks.send_reset_pass_email.delay(to, name, code) 

                    cache.set('email_vercode_' + username, code, timeout=300)
                    response_data = {'recipient': to, 'msg': '邮件已发送', 'result': res['data']}
                    return Response(data=response_data)
                except Exception as e:
                    log.error(e)
                    response_data = {'recipient': username, 'msg': e}
                    return Response(data=response_data)
            else:
                raise exceptions.ValidationError({'username': [res['msg']]})
        else:
            raise exceptions.ValidationError({'username': ['域账号不能为空']})
```


## 启动命令

```
/env/python manage.py celery worker --loglevel=info -E -c 2 &
```
