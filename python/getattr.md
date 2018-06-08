#  __getattribute__ 与 __getattr__


#### 例1 
```
class Count():
    def __init__(self,mymin,mymax):
        self.mymin=mymin
        self.mymax=mymax

obj1 = Count(1,10)
print(obj1.mymin)
print(obj1.mymax)
print(obj1.mycurrent)

输出：
1
10
AttributeError: 'Count' object has no attribute 'mycurrent'
```

#### 例2
```
class Count:
    def __init__(self,mymin,mymax):
        self.mymin=mymin
        self.mymax=mymax    

    def __getattr__(self, item):
        self.__dict__[item]=0
        return 0

obj1 = Count(1,10)
print(obj1.mymin)
print(obj1.mymax)
print(obj1.mycurrent1)

输出：
1
10
0
```

#### 例3
```
class Count:

    def __init__(self,mymin,mymax):
        self.mymin=mymin
        self.mymax=mymax
        self.current=None

    def __getattribute__(self, item):
        if item.startswith('cur'):
            raise AttributeError('not found attribute {}'.format(item))
        return object.__getattribute__(self,item) 
        # or you can use ---return super().__getattribute__(item)

obj1 = Count(1,10)
print(obj1.mymin)
print(obj1.mymax)
print(obj1.current)

输出：
1
10
AttributeError: not found attribute current
```

#### 例4
```
class Count(object):

    def __init__(self,mymin,mymax):
        self.mymin=mymin
        self.mymax=mymax
        self.current=None

    def __getattr__(self, item):
            self.__dict__[item]=0
            return 0

    def __getattribute__(self, item):
        if item.startswith('cur'):
            raise AttributeError('not found attribute {}'.format(item))
        return object.__getattribute__(self,item)
        # or you can use ---return super().__getattribute__(item)
        # note this class subclass object

obj1 = Count(1,10)
print(obj1.mymin)
print(obj1.mymax)
print(obj1.current)

输出：
1
10
None
```

### 总结
* __getattribute__ 在访问实例任何的属性时都会被调用
* __getattr__ 只会在没有查找到相应实例属性时被调用
* __getattribute__ 与 __getattr__ 在一个类中同时被实现时，__getattribute__ 会优先被调用，如果raises AttributeError则异常会被忽略，且__getattr__不会被调用
