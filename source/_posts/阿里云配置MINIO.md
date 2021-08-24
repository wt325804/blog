---
layout:
title: 阿里云配置MINIO
author: 海染了天空白了什么
avatar: https://cdn.jsdelivr.net/gh/wt325804/BlogImg@1.0/123.jpg
date: 2019-05-11 16:20:56
updated:
comments: 哈哈哈
tags:
 - MINIO
 - 随笔
categories:
- other
photos: https://cdn.jsdelivr.net/gh/wt325804/BlogImg@1.0/blog1.jpg
---



# 阿里云-ECS服务器-CENTOS7系统部署MINIO图床

## 1. 下载MINIO的二进制文件

==注: 阿里云ECS网速过慢, 但可以接受== 

```
wget https://dl.minio.io/server/minio/release/linux-amd64/minio
```

```
// 为minio文件赋予750权限
chmod 750 minio
```

## 2. 创建MINIO运行用户

```
// 创建用户组
groupadd -g 2021 minio
useradd -r -u 2021 -g 2021  -c "Minio User" -s /sbin/nologin minio

// 查看相关
id  minio 
>>> uid=2021(minio) gid=2021(minio) groups=2021(minio)
cat /etc/passwd
>>>inio:x:2021:2021:Minio User:/home/minio:/sbin/nologin
```

## 3. 创建MINIO相关目录

```
mkdir /usr/local/minio
mkdir /usr/local/minio/bin
mkdir /usr/local/minio/etc
mkdir /usr/local/minio/data
```

```
// 将下载的minio传入规定位置
cp minio /usr/local/minio/bin
```

## 4. 创建MINIO配置文件

**默认文件配置**

==注: listen_ip位置填写0.0.0.0, 不能填写公网/私网ip== 

```
vim /usr/local/minio/etc/minio.conf
```

```
# minio.conf文件内填写
MINIO_VOLUMES="/usr/local/minio/data"
MINIO_OPTS="-C /usr/local/minio/etc --address listen_ip:9000"
MINIO_ACCESS_KEY="MYMINIO"
MINIO_SECRET_KEY="12345678"
```

**启动文件配置**

```
vim /etc/systemd/system/minio.service
```

```
# minio.service文件内填写
[Unit]
Description=MinIO
Documentation=https://docs.min.io
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/usr/local/minio/bin/minio
[Service]
# User and group
User=minio
Group=minio
EnvironmentFile=/usr/local/minio/etc/minio.conf
ExecStart=/usr/local/minio/bin/minio server $MINIO_OPTS $MINIO_VOLUMES
# Let systemd restart this service always
Restart=always
# Specifies the maximum file descriptor number that can be opened by this process
LimitNOFILE=65536
# Disable timeout logic and wait until process is stopped
TimeoutStopSec=infinity
SendSIGKILL=no
[Install]
WantedBy=multi-user.target
```

## 5. 更改文件、目录属主属组

```
chown -R minio:minio /usr/local/minio
```

## 6. 启动/停止/查询服务

```
// 重载配置文件
systemctl daemon-reload

// 启动/停止/查询服务
systemctl enable minio.service  # 停止服务
systemctl start minio.service  # 启动服务
systemctl status minio.service  # 查询服务运行情况
systemctl restart minio.service  # 重启服务

// 过滤查询
ps aux | grep minio

// 端口查询
ss -tan | grep 9000
```

## 7. 配置阿里云安全组规则

![安全组规则设置](http://www.shichao521.top:9000/image/minio%E5%AE%89%E5%85%A8%E7%BB%84%E9%85%8D%E7%BD%AE.jpg?Content-Disposition=attachment%3B%20filename%3D%22minio%E5%AE%89%E5%85%A8%E7%BB%84%E9%85%8D%E7%BD%AE.jpg%22&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=scsc%2F20210514%2F%2Fs3%2Faws4_request&X-Amz-Date=20210514T141341Z&X-Amz-Expires=432000&X-Amz-SignedHeaders=host&X-Amz-Signature=af75c1eff6cdf78fcc7510eb63e8a95dc4cbf3f78a02d63d7e6ea267618b0b77)

## 8. 日志查看

==注: 服务可能启动失败==

```
// 日志中查看错误
tail -500 /var/log/messages
```

**<font color=red>常见错误: </font>**

* ```
    May 14 20:57:22 Change-myself minio: ERROR Unable to validate passed arguments: host in server address should be this server
    May 14 20:57:22 Change-myself minio: > Please check --address parameter
    May 14 20:57:22 Change-myself minio: HINT:
    May 14 20:57:22 Change-myself minio: --address binds to a specific ADDRESS:PORT, ADDRESS can be an IPv4/IPv6 address or hostname (default port is ':9000')
    
    # 配置文件监听地址错误, 配置为0.0.0.0
    MINIO_OPTS="-C /usr/local/minio/etc --address 0.0.0.0:9000"
    ```

* ```
    May 14 21:01:13 Change-myself minio: ERROR Unable to validate credentials inherited from the shell environment: Invalid credentials
    May 14 21:01:13 Change-myself minio: > Please provide correct credentials
    May 14 21:01:13 Change-myself minio: HINT:
    May 14 21:01:13 Change-myself minio: Access key length should be at least 3, and secret key length at least 8 characters
    
    # 用户名、密码配置错误, 长度问题
    MINIO_ACCESS_KEY="MYMINIO"
    MINIO_SECRET_KEY="12345678"
    ```

## 9. 访问成功

==注: 无域名就使用公网ip:9000==

![安全组规则设置](http://www.shichao521.top:9000/image/%E8%AE%BF%E9%97%AE%E6%88%90%E5%8A%9F.jpg?Content-Disposition=attachment%3B%20filename%3D%22%E8%AE%BF%E9%97%AE%E6%88%90%E5%8A%9F.jpg%22&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=scsc%2F20210514%2F%2Fs3%2Faws4_request&X-Amz-Date=20210514T143359Z&X-Amz-Expires=604800&X-Amz-SignedHeaders=host&X-Amz-Signature=f2381e0caf0d180a60a42d56be007f3820d8beb0fa44304b92e9f27622ecb04a)