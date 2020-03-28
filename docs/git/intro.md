# Git常用命令示例

#### 初始化

```
touch README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/cboy868/test.git
git push -u origin master
```

#### 添加远程仓库

```
git remote add origin https://github.com/cboy868/test.git
git push -u origin master
```

#### 强制下拉，会覆盖未提交代码，注意安全

```
git fetch --all 
git reset --hard origin/master
git pull
```



