## 关于Django的ORM之model

#### model常用字段类型
```
1、AutoField(Field)
# int自增列，必须填入参数primary_key=True, 当model中如果没有自增列，则自动会创建一个列名为id的列
# AutoField示例：
class UserInfo(models.Model):
    # 此处设置了AutoField必须指定为primary_key，model就不会创建名为id的列
    nid = models.AutoField(primary_key=True) 
    username = models.CharField(max_length=32)


2、DateTimeField(DateField)
# 日期+时间格式 YYYY-MM-DD HH:MM[:ss[.uuuuuu]][TZ]
# DateTimeField和Django中timezone.now()的类型都为datetime.datetime可以直接比较大小，差值为datetime.timedelta类型
# DateTimeField示例：在Django中经常要对ORM中取出的时间进行比较
# timedelta支持的参数类型: timedelta(days=999999999, hours=23, minutes=59, seconds=59, microseconds=999999)
from django.utils import timezone
from datetime import datetime
class Bills(models.Model):
    biz = models.CharField(max_length=64, default=uuid.uuid4)
    ctime = models.DateTimeField(auto_now_add=True)
obj = Bills.object.get(id=1)
now = timezone.now()
differ = now - obj.ctime  # 获取时间差对象(datetime.timedelta类型)
nextDay = now + timedelta(days=1)  # 增加一天后的时间(datetime.datetime类型)  
nextSecond = now + timedelta(minutes=10, seconds=10)  # 增加十分钟十秒后的时间 (datetime.datetime类型)
```

#### 字段及参数
```
1、null=True
null是针对数据库而言，如果null=True, 表示数据库的该字段可以为空

2、blank=True
blank是针对表单的，如果blank=True，表示你的表单填写该字段的时候可以不填

3、default
default=uuid.uuid4或者default=get_value, 注意后面没有括号，表示在插入一条数据时，调用此函数

4、unique=True
数据库中字段建立唯一索引

5、db_index=True
对数据库中字段建立索引

6、choices
STATUS_CHOICES = ((0, '未开始'),(1, '处理中'))
models.IntegerField(choices=STATUS_CHOICES, default=1)

7、DateTimeField
auto_now_add=True  # 插入一条新数据时，创建时间
auto_now=True  # 更新一条数据时，更新时间
```

#### model外键
```
外键：在某表中创建一行数据时，通过外键指定主表中关联的字段，来表明两个表之间的关系
例如：登录流水表中有一个user字段外键关联user表，user = ForeignKey(User_model), 字段赋值时应是user的实例
```

#### model索引
```

```

#### model一对一
```
一对一：在某表中创建一行数据时，要保证一对一的对应关系，如果有就不创建，没有就创建，get_or_created
例如：user表和token表使用OneToOneField，实现一个user只能有一条唯一的token记录
备注：一对一和外键的关系类似于OneToOneField = ForeignKey(model, unique=True), OneToOneField约束更严格
```

#### model一对多
```
一对多：当一张表中创建一行数据时，有一个单选的下拉框（可以被重复选择）
例如：创建用户信息时候，需要选择一个用户类型【普通用户】【金牌用户】【铂金用户】等
```

#### model多对多
```
多对多：在某表中创建一行数据时，有一个可以多选的下拉框
例如：创建用户信息，需要为用户指定多个爱好
```

