## django-redis

#### 安装
```
yum install redis-server
pip install django-redis
```

#### 配置
```
CACHES = {
  'default': {
    'BACKEND': 'redis_cache.cache.RedisCache',
    'LOCATION': 'redis://127.0.0.1:6379',
    'OPTIONS': {
      'CLIENT_CLASS': 'redis_cache.client.DefaultClient',
    },
  },
}
```

#### 使用
```
from django.conf import settings
from django.core.cache import cache


#read cache user id
def read_from_cache(self, user_name):
    key = 'user_id_of_'+user_name
    value = cache.get(key)
    if value == None:
        data = None
    else:
        data = json.loads(value)
    return data


#write cache user id
def write_to_cache(self, user_name):
    key = 'user_id_of_'+user_name
    cache.set(key, json.dumps(user_name), settings.NEVER_REDIS_TIMEOUT)
```
