# 服务器自动部署


有时项目维护人比较单一，项目不是很复杂，我们可能需要项目自动部署能力，只要把代码推到服务器，服务器自动部署，省去人工操作的步骤。

#### 到对应仓库中修改post-receive文件
比如仓库目录为/data/git/exampe.git

```
$ cd /data/git/example.git
$ vi hooks/post-receive #新建或复制文件，输入以下内容

#!/bin/sh
GIT_WORK_TREE=/data/htdocs/example git checkout -f
```
当客户端push到服务器来时，自动更新某个文件夹。

#### 增加执行权限
```
$ chmod +x hooks/post-receive #添加可执行权限
```

#### 注意给网站目录增加git的写入权限,简单的做法就是
```sql
$ chown -R www:git power
$ chmod g+w power
```


之后每次 push之后，power目录就会自动更新了
