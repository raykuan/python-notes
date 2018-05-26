## 代码目录结构

[基于 Django 架构网站代码的目录结构](https://www.cnblogs.com/codemyzen/p/3678787.html)

[【Django】基于 Django 架构网站代码的目录结构](http://www.cnblogs.com/codemyzen/p/3678787.html)
[django 最佳实践：项目布局](http://www.cnblogs.com/holbrook/archive/2012/02/25/2368231.html)

### 模块化目录结构

项目目录结构

```
$ tree .
.
├── $PROJECT_NAME/
│ ├── apps/ # 每个 app 只解决某一个方面的问题 (注1)
│ │ ├── myapp1/
│ │ ├── myapp2/
│ │ └── __init__.py
│ ├── common/ # project 级别的共用代码, 如权限, 异常处理, 模型基类, 分布等等
│ │ ├── __init__.py
│ │ └── permissions.py
│ ├── extra_apps/ # 引用的其他 app
│ │ ├── other_app
│ │ └── __init__.py
│ ├── libs/ # 加载第三方模块，可以避免版本冲突，按照标准的site-packages管理（注2）
│ │ └── python*.* # 指定python版本号
│ │ ├── site-packages/
│ │ └── requirements.pip # pip 的依赖说明文件
│ ├── settings/ # 区分不同条件下的设置, 可做成一个文件
│ │ ├── __init__.py # 默认使用 dev.py 开发环境设置
│ │ ├── common.py # 通用设置
│ │ ├── dev.py # 开发环境设置
│ │ ├── prod.py # 生产环境设置
│ │ └── test.py # 测试环境设置
│ ├── static/ # project 级别的静态文件, 所有共用的部分
│ ├── templates/ # project 级别的模板, 所有共用的部分
│ ├── tests/ # project 级别的测试, 对于每个app, 还要有自己的测试代码
│ ├── utils/ # 工具集, 与业务逻辑无关的工具代码
│ │ ├── __init__.py
│ │ └── util.py
│ ├── __init__.py
│ ├── urls.py # project 级别的 urls 映射, RESTful router 注册
│ └── wsgi.py
├── deploy/ # 部署设置模板, 或使用 docker
├── deploy_tools/ # 自动化部署工具, 如 fabric, 或使用 docker
│ └── fabfile.py # fabric 脚本
├── docs/ # 文档
├── functional_test/ # ui 自动化测试, 功能测试
├── logs/ # 开发环境日志目录
├── media/ # 开发环境上传文件目录
├── requirements # 上线后用于固定库的版本等
│ ├── all.txt # 所有库
│ ├── common.txt # 环境通用库
│ ├── dev.txt # 开发用的库
│ ├── prod.txt # 线上用的库
│ └── test.txt # 测试用的库
├── venv/ # 虚拟环境
├── .env # 环境变量配置文件
├── .coveragerc # 使用 Coverage 分析项目的代码覆盖率
├── .gitattributes # 文件属性, 如 EOL, 是否为二进制等
├── .gitignore # 哪些文件不要上传到 Git
├── CHANGELOG.md # 变更历史
├── CONTRIBUTING.md # 贡献人员
├── ISSUES.md # 问题点, bug 等
├── LICENSE # 授权协议
├── Makefile # 常规管理脚本
├── manage.py # django 入口文件
├── README.md # 自述文件, 安装/配置/运行等等
├── requirements.txt # python 依赖
└── TODO.md # 待开发功能

注1：
加载 app 时使用 $PROJECT_NAME.apps.myapp1
或在settings.py中设置：
sys.path.insert(0, os.path.join(PROJECT_ROOT, $PROJECT_NAME, 'app')
```
