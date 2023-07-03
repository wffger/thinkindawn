---
weight: 7
keywords:
- chatglm chinese english Conversational language model
title: "试用中英双语对话模型chatglm2-6b-int4"
date: 2023-07-03
lastmod: 2023-07-03
draft: false
authors: ["wffger"]
description: ""

tags: ["GLM","python"]
categories: ["AI"]
lightgallery: true
---

<!--more-->
# 本地试用chatglm2-6b-int4
本地机器16GB内存，不足以跑通chatglm2-6b，只能使用chatglm2-6b-int4。


## 步骤
参考：[chatglm2-6b-int4](https://huggingface.co/THUDM/chatglm2-6b-int4)  

1. 安装git lfs
2. 创建环境
3. 下载模型
4. 命令行调用

准备环境：  
```
mkdir u_chatglm2
cd u_chatglm2
pipenv --python 3.11
pipenv shell
pip install protobuf transformers==4.30.2 cpm_kernels torch>=2.0 gradio mdtex2html sentencepiece accelerate
git clone https://huggingface.co/THUDM/chatglm2-6b-int4

```

使用：  
```
from transformers import AutoTokenizer, AutoModel
tokenizer = AutoTokenizer.from_pretrained("chatglm2-6b-int4", trust_remote_code=True)
model = AutoModel.from_pretrained("chatglm2-6b-int4",trust_remote_code=True).float()
model = model.eval()
response, history = model.chat(tokenizer, "你好", history=[])
print(response)
response, history = model.chat(tokenizer, "内脏脂肪多怎么办", history=history)
print(response)
```

## 效果

{{< image src="/images/chatglm2-6b-int4.png" caption="chatglm2-6b-int4 result" >}}
## 评价
回答第二个问题需要二十多分钟。