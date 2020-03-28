# docker安装Gogs 

#### 创建目录
```
$ mkdir /data/docker/gogs
```

#### 编写docker-compose文件
```
$ cd /data/docker/gogs
$ vim docker-compose.yml
```
添加以下内容
net-mysql 是否需要 再确定,应该不需要
```
version: "3"
services:
  gogs:
    image: gogs/gogs
    container_name: host_gogs
    hostname: host_gogs
    ports:
      - "10022:22"
      - "10080:3000"
    volumes:
      - /data/gogs:/data
    networks:
     - net-gogs
     - net-mysql
networks:
  net-gogs:
  net-mysql:
```
#### 启动
```
docker-compose up -d
```

#### nginx返向代理配置
```
upstream gogs {
 server 内网地址:10080; #注:配置docker服务器时，这里的内网地址不可以是127.0.0.1
}

server{
    listen 80;
    server_name gogs.abc.com;
    error_log  /var/log/dnmp/gogs.error.log  warn;
    access_log  /var/log/dnmp/gogs.access.log  main;

    location / {
        proxy_pass   http://gogs;
    }
}
```

#### 重启nginx
#### 远端访问 gogs.abc.com

至此gogs安装完成 

#### gogs配置

```
APP_NAME = Gogs
RUN_USER = git
RUN_MODE = prod

[database]
DB_TYPE  = mysql
HOST     = 内网ip:3306
NAME     = gogs
USER     = root
PASSWD   = 123456
SSL_MODE = disable
PATH     = data/gogs.db

[repository]
ROOT = /data/git/gogs-repositories

[server]
DOMAIN           = http://gogs.abc.com
HTTP_PORT        = 3000
ROOT_URL         = http://gogs.abc.com:80/
DISABLE_SSH      = false
SSH_PORT         = 10022
START_SSH_SERVER = false
OFFLINE_MODE     = false

[mailer]
ENABLED = false

[service]
REGISTER_EMAIL_CONFIRM = false
ENABLE_NOTIFY_MAIL     = false
DISABLE_REGISTRATION   = false
ENABLE_CAPTCHA         = true
REQUIRE_SIGNIN_VIEW    = false

[picture]
DISABLE_GRAVATAR        = false
ENABLE_FEDERATED_AVATAR = false

[session]
PROVIDER = file

[log]
MODE      = file
LEVEL     = Info
ROOT_PATH = /app/gogs/log

[security]
INSTALL_LOCK = true
SECRET_KEY   = PelRoiGlDCKhyPG
```


#### 配置完成后，切勿忘记在这里生成 '.ssh/authorized_keys' 文件,否则使用ssh连接时会报以下错误:
http://gogs.abc.com/admin

```
Cloning into 'lichen'...
The authenticity of host '[gogs.iutree.com]:10022 ([47.93.53.205]:10022)' can't be established.
ECDSA key fingerprint is SHA256:L+9csagelv44D3GDT4EeoRULedvLFqQfDeJPCQQHI/4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[gogs.iutree.com]:10022,[47.93.53.205]:10022' (ECDSA) to the list of known hosts.
git@gogs.iutree.com: Permission denied (publickey,keyboard-interactive).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```


### 错误处理

502

```
2019/06/10 09:53:00 [error] 6#6: *1 recv() failed (104: Connection reset by peer) while reading response header from upstream, client: 1.119.196.210, server: gogs.abc.com, request: "GET / HTTP/1.1", upstream: "http://ip:10080/", host: "gogs.iutree.com"
2019/06/10 09:53:00 [error] 6#6: *1 recv() failed (104: Connection reset by peer) while reading response header from upstream, client: 1.119.196.210, server: gogs.abc.com, request: "GET /favicon.ico HTTP/1.1", upstream: "http://ip:10080/favicon.ico", host:
 "gogs.abc.com", referrer: "http://gogs.abc.com/"
```

好久没有找到答案，后来把app.ini删除，重新生成了一遍，结果ok了，应该是安装完成后，修改此文件，ip地址修改的有些问题，本安装是通过docker安装，所使用的mysql库也是docker环境，所以会有些问题，如果真机直接安装，会少了很多问题。



























