
## 关于Django的setting.py中静态文件相关的配置

#### template配置
```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, "templates")],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

# 从Django1.8开始废弃了TEMPLATES_DIRS，之后所有和template相关的配置项都放在TEMPLATES中
'DIRS': [os.path.join(BASE_DIR, "templates")], 指定template的目录

```


#### static配置
```
STATIC_URL = '/static/'
STATICFILES_DIRS = (os.path.join(BASE_DIR, "static"),)
STATIC_ROOT = os.path.join(BASE_DIR, "project/static")

# STATIC_URL是静态文件的URL前缀(在URL和template中显示的)
URL：https://github.com/raykuan/static/customstyle.css
template:{% load static %}或<link rel="stylesheet" href="/static/css/customstyle.css" %}">

# STATICFILES_DIRS是自定义的静态文件目录
默认情况下Django只会去每个APP的static目录找静态文件，如果想在别的地方设置静态文件目录就需要添加在此处

# STATIC_ROOT是静态文件的总目录
执行manage.py collectstatic会把所有APP下static目录和STATICFILES_DIRS自定义目录的静态文件都拷贝到STATIC_ROOT目录中
```


#### media配置
```
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, "project/media")

# MEDIA_URL是静态文件的URL前缀类似STATIC_URL
URL：https://github.com/raykuan/media/log20170909.log

# MEDIA_ROOT是media文件的存放目录
如MEDIA_ROOT=/data/media，那么File=models.FileField(upload_to="images/")，上传的文件就会被保存到/data/media/images/目录下
```
