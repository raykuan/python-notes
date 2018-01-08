## Git常用操作汇总

#### 合并分支
```
推荐使用--no-ff参数后，会执行正常合并，在Master分支上生成一个新节点
git merge --no-ff dev_branch

```

#### 打标签
```
1、打tag标签(添加tag应该在commit之后push之前)
git tag -a v1.0 -m “commit version 1.0”

2、push到远程仓库(打完tag之后，去push即可)
git push origin –tags (push所有tag到远程仓库)

3、删除tag便签
git tag -d v1.0

4、查看tag标签
git tag
```

#### 版本回退
```
1、git log查看版本号(git log --pretty=oneline一行显示)

2、git reset --hard 版本号(前8位)

```
