---
title: "快速使用指南"
date: 2026-08-04T00:00:00+08:00
draft: false
description: "三步快速接入YesAPI中转，完全兼容OpenAI接口"
weight: 2
---

## 1、注册 YesAPI 中转账号

前往 [YesAPI 中转官网](https://yesapi.online) 完成注册，新用户注册即可获得测试额度，方便先跑通流程再决定是否充值。

## 2、获取 API Key

登录后台，进入令牌管理页面，点击「新建令牌」，填写名称后保存，即可获得生成的 Key。

Key 的格式与 OpenAI 官方一致：`sk-xxxx......`

## 3、修改请求地址

调用方式与 OpenAI 官方 SDK 完全一致，只需要把请求的 base URL 替换为：

```
https://yesapi.online
```

并使用在 YesAPI 中转后台生成的令牌作为 API Key，其他代码逻辑无需任何改动。

### 模型支持

本站支持大部分 OpenAI、Claude、DeepSeek 等主流模型，注册后即可在模型价格页查看完整列表并一键复制模型名称。

### 注意事项

不同客户端可能需要填写不同格式的 BASE_URL，部分程序需要加上 `/v1` 后缀，请根据实际情况测试：

- `https://yesapi.online`
- `https://yesapi.online/v1`
- `https://yesapi.online/v1/chat/completions`
