
### 基本配置
```
**django_auth_example/settings.py

INSTALLED_APPS = [
    # 其它应用列表...
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'users', # 注册新建的应用 users
]
# django.contrib.contenttypes 是auth模块的用户权限处理部分依赖的应用

MIDDLEWARE = [
    # 其它中间列表...
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
]
# SessionMiddleware 用户处理用户会话
# AuthenticationMiddleware 绑定一个User对象到请求中（具体将在后面介绍）
```

### 拓展User模型
```
Django 用户认证系统提供了一个内置的User对象，用于记录用户的用户名，密码等个人信息。
对于 Django 内置的 User 模型， 仅包含以下一些主要的属性：
username，即用户名
password，密码
email，邮箱
first_name，名
last_name，姓
对于一些网站来说，用户可能还包含有昵称、头像、个性签名等等其它属性，因此仅仅使用Django内置的User模型是不够。
好在 Django 用户系统遵循可拓展的设计原则，我们可以方便地拓展 User 模型。

1、继承AbstractUser拓展用户模型
这是推荐做法，查看User的源码就知道，User也是继承自AbstractUser抽象基类，仅仅就是继承了AbstractUser，没有做任何的拓展。
**users/models.py
from django.db import models
from django.contrib.auth.models import AbstractUser
class User(AbstractUser):
    nickname = models.CharField(max_length=50, blank=True)
    class Meta(AbstractUser.Meta):
        pass
        
注意：一定要继承 AbstractUser，而不是继承 auth.User。
尽管 auth.User 继承自 AbstractUser 且并没有对其进行任何额外拓展，但 AbstractUser 是一个抽象类，而 auth.User 不是。
如果你继承了 auth.User 类，这会变成多表继承，在目前的情况下这种继承方式是不被推荐的。
关于 Django 的抽象模型类和多表继承，请查阅 Django的官方文档 模型继承。
此外，AbstractUser 类又继承自AbstractBaseUser，前者在后者的基础上拓展了一套用户权限（Permission）系统。
因此如非特殊需要，尽量不要从AbstractBaseUser拓展，否则你需要做更多的额外工作。
为了让 Django 用户认证系统使用我们自定义的用户模型，必须在settings.py里通过AUTH_USER_MODEL指定自定义用户模型所在的位置：
**django_auth_example/settings.py
# 其它设置...
AUTH_USER_MODEL = 'users.User'


2、使用Profile模式拓展用户模型
如果想为一个已使用了Django内置User模型的项目拓展用户模型，上述继承AbstractUser的拓展方式会变得有点麻烦。
Django没有提供一套自动化方式将内置User迁移到自定义的User，因为Django已经为内置的User模型生成了数据库迁移文件和数据库表。
如果非要这么做的话，需要手工修改迁移文件和数据库表，并且移动数据库中相关的用户数据。
所以我们采用另一种不改动数据库表的方式来拓展用户模型，具体来说我们在创建一个模型（通常命名为Profile）来记录用户相关的数据，
然后使用一对一的方式将这个Profile模型和User关联起来，就好像每个用户都关联着一张记录个人资料的表一样。代码如下：
**models.py
from django.contrib.auth.models import User

class Profile(models.Model):
    nickname = models.CharField(max_length=50, blank=True)
    user = models.OneToOneField(User)
这种方式和 AbstractUser 的区别是，继承 AbstractUser 的用户模型只有一张数据库表。
而 Profile 这种模式有两张表，一张是User模型对应的表，一张是Profile模型对应的表，两张表通过一对一的关系关联。
可见，当要查询某个用户的Profile时，需要执行额外的跨表查询操作，所以这种方式比起直接继承 AbstractUser 效率更低一点。
因此对于新项目来说，优先推荐使用继承 AbstractUser 的方式来拓展用户模型。
PS：如果你使用了Profile模式，希望在创建User对象的同时也创建与之关联的Profile对象，可以使用Django的Signal实现这个需求。
```

### 自定义认证后台
```
Django auth 应用默认支持用户名（username）进行登录。
但是在实践中，网站可能还需要邮箱、手机号、身份证号等进行登录，这就需要我们自己写一个认证后台，
用于验证用户输入的用户信息是否正确，从而对拥有正确凭据的用户进行登录认证。

1、Django 验证用户合法性的方式
Django 对用户登录的验证工作均在一个被称作认证后台（Authentication Backend）的类中进行。
这个类是一个普通的 Python 类，它有一个authenticate方法，接收登录用户提供的凭据（如用户名或者邮箱以及密码）作为参数，
并根据这些凭据判断用户是否合法（即是否是已注册用户，密码是否正确等）。
下面是 Django 内置的认证后台的部分源代码，从代码中可以清晰地看到其工作方式：

django.contrib.auth.backends

class ModelBackend(object):
    """
    Authenticates against settings.AUTH_USER_MODEL.
    """
    
    def authenticate(self, request, username=None, password=None, **kwargs):
        if username is None:
            username = kwargs.get(UserModel.USERNAME_FIELD)
        try:
            user = UserModel._default_manager.get_by_natural_key(username)
        except UserModel.DoesNotExist:
            # Run the default password hasher once to reduce the timing
            # difference between an existing and a non-existing user (#20760).
            UserModel().set_password(password)
        else:
            if user.check_password(password) and self.user_can_authenticate(user):
                return user
这段代码根据用户传入的username和password，验证该username对应的用户是否存在以及密码是否正确，是则返回该 user 对象。
可以定义多个认证后台，Django 内部会逐一调用这些后台的authenticate方法来验证用户提供登录凭据的合法性，
一旦通过某个后台的验证，表明用户提供的凭据合法，从而允许登录该用户。

2、Email Backend
在本示例项目中，用户注册时需要填写邮箱，。
因为Django auth应用内置只支持用户名和密码的认证方式，所以是无法使用Email进行登录的,为了实现邮箱登录需要编写一个认证后台。
这个后台的作用便是验证用户提供的凭据（这里是邮箱以及密码）是合法的，完全仿照内置的 ModelBackend 代码即可。
首先在 users 应用下新建一个 backends.py 文件，然后写入如下代码：
users/backends.py
from .models import User

class EmailBackend(object):
    def authenticate(self, request, **credentials):
        # 要注意登录表单中用户输入的用户名或者邮箱的 field 名均为 username
        email = credentials.get('email', credentials.get('username'))
        try:
            user = User.objects.get(email=email)
        except User.DoesNotExist:
            pass
        else:
            if user.check_password(credentials["password"]):
                return user

    def get_user(self, user_id):
        """
        该方法是必须的
        """
        try:
            return User.objects.get(pk=user_id)
        except User.DoesNotExist:
            return None
根据用户提供的Email和密码，检查该emai对应的用户是否存在，如果存在则检查密码是否正确，如果密码也没有问题，则返回该user对象。

3、配置Backend
接下来就要告诉 Django，需要使用哪些 Backends 对用户的凭据信息进行验证，这需要在 settings.py 中设置：
settings.py

AUTHENTICATION_BACKENDS = (
    'django.contrib.auth.backends.ModelBackend',
    'users.backends.EmailBackend',
)
第一个Backend是Django 内置的 Backend，当用户提供的是用户名和正确的密码时该Backend会通过验证；
第二个 Backend 是刚刚自定义的 Backend，当用户提供的是 Email 和正确的密码时该 Backend 会通过验证。

4、备注
因为django内置的User model定义时email默认可以重复，考虑到难以修改 model 的定义，且不是一个好方法。
我们可以通过复写表单的验证方法（例如 clean_email）来验证email的唯一性，具体请参考 django 关于 form 数据验证的文档。

email = credentials.get('email', credentials.get('username'))，
请参考python dict对象的get方法。具体来说，dict.get('key')，如果key不存在则返回None。
也可以设置key不存在时的返回值，dict.get('key'，value)，当key不存在时，会返回value。
```
