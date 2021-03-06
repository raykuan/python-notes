## 安装及启动celery

```
安装celery
/env/pip install celery

启动celery
/env/python manage.py celery worker --loglevel=info -E -c 2 &
```

## 在django中使用celery

### 1. 在django settings.py中添加celery配置项
```
# 更多的配置项参见如下官方链接
# http://docs.celeryproject.org/en/latest/userguide/configuration.html#new-lowercase-settings

# Settings for Celery
CELERY_ENABLE_UTC = False
CELERY_TIMEZONE = 'Asia/Shanghai'
BROKER_URL = 'redis://10.213.32.36:6379/1'             # 指定redis作为消息代理broker(一般使用RabbitMQ和redis)
CELERY_RESULT_BACKEND = 'redis://10.213.32.36:6379/2'  # 把任务结果存在redis(也支持mysql等关系数据库)
CELERY_TASK_RESULT_EXPIRES = 60 * 60 * 24              # 任务过期时间，默认是86400
```

### 2. 在django project配置文件目录下新建celery.py
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

### 3. 在django project配置文件目录下的__init__.py添加如下内容
```
from __future__ import absolute_import

# This will make sure the app is always imported when
# Django starts so that shared_task will use this app.

from .celery import app as celery_app

__all__ = ['celery_app']

```

### 4. 在django app目录下新建tasks.py (当前APP中需要异步执行的函数都写在此文件中)
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

### 5. 在django view中使用tasks.py异步函数
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