## Python之安装virtualenv虚拟环境

#### Linux环境：
```
1 先装好python环境，同时安装py2和py3两个版本，默认版本是Linux自带的py2
[root@host ~]# which python
/usr/bin/python
[root@host ~]# which python2
/usr/bin/python2
[root@host ~]# which python3
/usr/local/bin/python3

2 使用yum安装python-virtualenv，系统自带rpm包
[root@host ~]# yum install python-virtualenv
或者
[root@host ~]# pip install virtualenv

3 创建python虚拟环境
[root@host ~]# mkdir env
[root@host ~]# cd env
[root@host env]# virtualenv pyenv01
[root@host env]# cd pyenv01/

4 进入python虚拟环境
[root@host pyenv01]# source bin/activate
(pyenv01)[root@host pyenv01]# python --version
Python 2.7.5
(pyenv01)[root@host pyenv01]# pip list
pip (1.4.1)
setuptools (0.9.8)
wsgiref (0.1.2)

5 退出python虚拟环境
(pyenv01)[root@host pyenv01]# deactivate
[root@host pyenv01]# 

6 指定创建python3版本的虚拟环境
[root@host ~]# virtualenv -p /usr/local/bin/python3 py3env
```

#### Windows环境：
```
1 先装好python环境，也可以同时安装py2和py3两个版本
2 pip install virtualenv
3 virtualenv pyenv01(默认在当前目录下创建pyenv01)
4 cd pyenv01/Scripts/
5 执行activate.bat(cmd下)，或source activate(git cmd下)进入虚拟环境
6 执行deactivate退出虚拟环境
```

