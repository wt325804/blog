---
layout:
title: Minio操作
author: 海染了天空白了什么
avatar: https://cdn.jsdelivr.net/gh/wt325804/BlogImg@1.0/123.jpg
date: 2019-05-11 16:20:56
updated:
comments: 哈哈哈
tags:
 - Minio
 - 随笔
categories:
- other
photos: https://cdn.jsdelivr.net/gh/wt325804/BlogImg@1.0/blog1.jpg
---





# Minio从下载安装到使用到上传下载到图片显示



## Minio下载安装(docker版)



```python
# 下载安装
docker pull minio/minio

# 运行minio
'''
这里一定要开启两个端口映射，并且指定--console-address ":9000"为9000，因为minio会有个动态随机的端口用于访问客户端，如果只是想跑API接口，可以映射一个即可在python中操作。
'''
docker run -p 9000:9000 -p 9001:9001 --name minio -d --restart=always -e "MINIO_ACCESS_KEY=admin" -e "MINIO_SECRET_KEY=admin123456" -v /home/data:/data -v /home/config:/root/.minio minio/minio server /data --console-address ":9000" --address ":9001" 
                

```



## Minio上传与下载

```python
# python环境安装
pip install minio

# 创建Minio对象
minioClient = Minio('存储桶名',
                    access_key='MINIO_ACCESS_KEY',
                    secret_key='MINIO_SECRET_KEY',
                    secure=True)
# 若是http请求，secure为False

# 上传 fput_object(bucket_name, object_name, file_path, content_type='application/octet-stream', metadata=None)
fput_object('image','任意名称','文件路径')


# 下载 fget_object(bucket_name, object_name, file_path, request_headers=None)
fget_object('image','lbt1.jpg','路径')

```

## Minio获取url并应用在前端页面

```python
# presigned_get_object(bucket_name, object_name, expiry=timedelta(days=7))
from datetime import timedelta

print(minioClient.presigned_get_object('image', 'lbt1.jpg', expires=过期时间))


```



