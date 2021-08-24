---
layout:
title: FastDFS
author: 海染了天空白了什么
avatar: https://cdn.jsdelivr.net/gh/wt325804/BlogImg@1.0/123.jpg
date: 2019-05-11 16:20:56
updated:
comments: 哈哈哈
tags:
 - FastDFS
 - 随笔
categories:
- other
photos: https://cdn.jsdelivr.net/gh/wt325804/BlogImg@1.0/blog1.jpg
---



# python操作FastDFS

## 1 启动FastDFS

```python
# 拉取镜像
docker pull liuqingzheng/fastdfs:v1
# 创建目录 
mkdir /home/tracker
mkdir /home/storage
# 使用docker镜像构建tracker容器（跟踪服务器，起到调度的作用）
docker run -d --network=host --name tracker -v /home/tracker:/var/fdfs liuqingzheng/fastdfs:v1 tracker
#使用docker镜像构建storage容器（存储服务器，提供容量和备份服务）
# 修改成你的ip地址
docker run -d --network=host --name storage -e TRACKER_SERVER=101.133.225.166:22122 -v /home/storage:/var/fdfs -e GROUP_NAME=group1 liuqingzheng/fastdfs:v1 storage 

#此时两个服务都以启动， 进行服务的配置
#进入storage容器， 到storage的配置文件中配置http访问的端口， 配置文件在/etc/fdfs目录下的storage.conf
docker exec -it storage /bin/bash
vi /etc/fdfs/storage.conf
```

## 2 python3 操作FastDFS

### 第一步：安装模块

```
# 第一步：安装模块
pip3 install py3Fdfs
```

### 第二步：在项目目录下新建client.conf

```
connect_timeout=30
network_timeout=60
tracker_server = 101.133.225.166:22122
# tracker服务器的端口
http.tracker_server_port = 8888
```

### 第三步：增加，下载，删除代码如下

```
from fdfs_client.client import get_tracker_conf, Fdfs_client

tracker_conf = get_tracker_conf('./client.conf')
client = Fdfs_client(tracker_conf)

#文件上传
result = client.upload_by_filename('./db.sqlite3')
print(result)
# {'Group name': b'group1', 'Remote file_id': b'group1/M00/00/00/rBMGZWCeGhqAR_vRAAIAABZebgw.sqlite', 'Status': 'Upload successed.', 'Local file name': './db.sqlite3', 'Uploaded size': '128.00KB', 'Storage IP': b'101.133.225.166'}
# 访问地址即可下载：http://101.133.225.166:8888/group1/M00/00/00/rBMGZWCeGhqAR_vRAAIAABZebgw.sqlite


#文件下载
# result = client.download_to_file('./lqz.sqlite', b'group1/M00/00/00/rBMGZWCeGxaAFWqfAAIAABZebgw.sqlite')
# print(result)

# #文件删除
# result = client.delete_file(b'group1/M00/00/00/rBMGZWCeGhqAR_vRAAIAABZebgw.sqlite')
# print(result)
# ('Delete file successed.', b'group1/M00/00/00/rBMGZWCeGhqAR_vRAAIAABZebgw.sqlite', b'101.133.225.166')

# #列出所有的group信息
# result = client.list_all_groups()
# print(result)
```

 

