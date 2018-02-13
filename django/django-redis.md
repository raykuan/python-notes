## django-redis

#### 安装
```
创建redis用户
useradd redis

在redis用户下
tar -xvf redis-3.2.0.tar.gz
cd redis-3.2.0
make

在root用户下进入编译好的目录
make install

pip install django-redis
```

#### 启动
```
在任何用户下都可以使用自己的配置文件启动自己的redis只要保证端口不冲突即可
redis-server /path/my_custom_redis.conf
后台启动redis：nohup redis-server /path/6379.conf &

配置文件参见目录下的redis.conf文件
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
