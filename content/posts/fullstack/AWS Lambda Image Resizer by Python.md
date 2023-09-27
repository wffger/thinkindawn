---
weight: 7
keywords:
- aws lambda python
title: "AWS Lambda Image Resizer by Python"
date: 2023-09-27
lastmod: 2023-09-27
draft: false
authors: ["wffger"]
description: ""

tags: ["python","aws lambda"]
categories: ["AWS"]
lightgallery: true
---

<!--more-->
# 步骤
## 创建角色
创建角色Lambda-S3-Role，选择Lambda作为可信实体，授权AmazonS3FullAccess和AWSLambdaBasicExecutionRole。
## 创建桶
创建两个桶：img-wffger和img-wffger-resized。<br />第一个桶中创建文件夹images，用作Lambda S3触发器的前缀。<br />我们往images上传jpg文件，将会触发Lambda 函数执行python操作，创建新的图片并上传到第二个桶。<br />第二个桶的名称必须为第一个桶名称加上resized后缀。
## 创建环境依赖层
aws自带的layer没有提供Pillow模块，需要我们自己创建一个包含依赖的layer。
### 本地创建zip包
```python
py -m pip install Pillow -t python/
zip -r pillow.zip python
```
### Lambda创建层
[创建层](https://ap-southeast-1.console.aws.amazon.com/lambda/home?region=ap-southeast-1#/create/layer)，上传zip，命名为python311-pillow
## 创建Lambda函数
选择执行角色为Lambda-S3-Role<br />添加层，python311-pillow
### 编辑代码
修改代码后，记得点击Deploy。
```python
import boto3
import os
from PIL import Image

s3 = boto3.client('s3')

def resize_image(image_path, resized_path):
    with Image.open(image_path) as image:
        image.thumbnail(tuple(x / 2 for x in image.size))
        image.save(resized_path)

def lambda_handler(event, context):
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        tempkey = key.replace('/', '')
        download_path = '/tmp/{}{}'.format(os.path.splitext(tempkey)[0], os.path.splitext(tempkey)[1])
        upload_path = '/tmp/resized-{}'.format(tempkey)

        s3.download_file(bucket, key, download_path)
        resize_image(download_path, upload_path)
        s3.upload_file(upload_path, '{}-resized'.format(bucket), key)
    
```
### 编辑触发器

1. 添加事件类型为“All object create events”
2. 编辑Prefix为“images”
3. 编辑Suffix为“.jpg"

{{< image src="/images/ladmbda-s3-trigger.png" caption="s3 trigger" >}}

<br />触发器创建完毕后，相关事件通知可以在img-wffger的属性页签下找到。

### 使用虚拟事件测试
修改下面的JSON，共四处修改，其中key为真实存在的文件。<br />Records.awsRegion<br />Records.s3.bucket.name<br />Records.s3.bucket.arn<br />Records.s3.object.key
```json
{
  "Records": [
    {
      "eventVersion": "2.0",
      "eventSource": "aws:s3",
      "awsRegion": "ap-southeast-1",
      "eventTime": "1970-01-01T00:00:00.000Z",
      "eventName": "ObjectCreated:Put",
      "userIdentity": {
        "principalId": "EXAMPLE"
      },
      "requestParameters": {
        "sourceIPAddress": "127.0.0.1"
      },
      "responseElements": {
        "x-amz-request-id": "EXAMPLE123456789",
        "x-amz-id-2": "EXAMPLE123/5678abcdefghijklambdaisawesome/mnopqrstuvwxyzABCDEFGH"
      },
      "s3": {
        "s3SchemaVersion": "1.0",
        "configurationId": "testConfigRule",
        "bucket": {
          "name": "img-wffger",
          "ownerIdentity": {
            "principalId": "EXAMPLE"
          },
          "arn": "arn:aws:s3:::img-wffger"
        },
        "object": {
          "key": "images/field-6574455_1920.jpg",
          "size": 1024,
          "eTag": "0123456789abcdef0123456789abcdef",
          "sequencer": "0A1B2C3D4E5F678901"
        }
      }
    }
  ]
}
```

## 查看结果
第一个桶

{{< image src="/images/bucket-source.png" caption="bucket source" >}}

第二个桶

{{< image src="/images/bucket-resized.png" caption="bucket resized" >}}


---

# 错误说明
> Unable to import module 'lambda_function': libjpeg.so.62
> 出现这个报错说明layer有误


> Cannot have overlapping prefixes () or suffixes (.jpg) in two rules for the same event type.
> 不能有重叠的规则。必须移除先前的规则。


---

# 参考
[https://repost.aws/knowledge-center/lambda-import-module-error-python](https://repost.aws/knowledge-center/lambda-import-module-error-python)<br />[https://docs.aws.amazon.com/zh_cn/lambda/latest/dg/with-s3-example.html](https://docs.aws.amazon.com/zh_cn/lambda/latest/dg/with-s3-example.html)
