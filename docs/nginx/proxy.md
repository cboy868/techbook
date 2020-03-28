# 反向代理配置

## 反向代理
### 上游配置

机器1 内网地址：10.12.12.1 外网地址：12.10.10.1
```
server {
   listen   10.12.12.1:8880; #监听的端口前加上固定地址
}
```

机器2 内网地址：10.12.12.2 外网地址：12.10.10.2
```
server {
   listen   10.12.12.2:8880; #监听的端口前加上固定地址
}
```

机器3 内网地址：10.12.12.3 外网地址：12.10.10.3
```
server {
   listen   10.12.12.3:8880; #监听的端口前加上固定地址
}
```

### 反向代理配置

```
upstream local {
    server 12.10.10.1:8880;
    server 12.10.10.2:8880;
    server 12.10.10.3:8880;
}
server {
    listen 8880;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://local;
    }
}
```


## 反向代理缓存
针对不同用户页面相同、且不变的页面进行缓存，减轻上游服务压力

### 缓存部分
```
http {
    #整体缓存设置
    proxy_cache_path /tmp/nginxcache levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m use_temp_path=off;
}
```

### 使用缓存

```
server {
    server_name ...;
    listen ...;
    location / {
        proxy_cache my_cache;
        proxy_cache_key $host$uri$is_args$args;#缓存受这些参数影响
        proxy_cache_valid 200  304 302 1d;#这些响应不返回
        proxy_pass ...;
    }
}

```