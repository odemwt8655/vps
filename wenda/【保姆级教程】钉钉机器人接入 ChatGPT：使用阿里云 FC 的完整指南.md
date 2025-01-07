
# 【保姆级教程】钉钉机器人接入 ChatGPT：使用阿里云 FC 的完整指南

## 前言

近年来，人工智能的热度持续高涨，越来越多的开发者开始尝试将 AI 应用到实际场景中。本文将为大家详细介绍如何通过阿里云 FC 将钉钉机器人接入 ChatGPT。本教程基于之前的阿里云对接 OpenAI API 经验，适合有一定开发基础的朋友操作。

---

## 前提条件

在开始之前，确保以下条件已满足：

1. **阿里云账号**
   - 本教程将使用阿里云的 FC（函数计算）和 OSS（对象存储服务）。
2. **OpenAI账号**
   - 需要绑定银行卡（无需开通会员）。
   - 如果缺少虚拟信用卡，可通过 [WildCard虚拟卡](https://bit.ly/bewildcard) 注册，完成绑定。
3. **钉钉账号**
   - 用于创建钉钉机器人应用。

---

## 准备工作

### GitHub 项目文件

1. 前往 [GitHub项目地址](https://github.com/cyoahs/Dingtalk-ChatGPT-Connector) 下载以下两份脚本：
   - [Dingtalk_Conversation.py](https://github.com/cyoahs/Dingtalk-ChatGPT-Connector/blob/main/Dingtalk_Conversation.py)
   - [Dingtalk_ChatGPT_Reply.py](https://github.com/cyoahs/Dingtalk-ChatGPT-Connector/blob/main/Dingtalk_ChatGPT_Reply.py)

2. **修改脚本逻辑**：
   - **第一份脚本**：删除多余的验签代码，以避免阿里云 FC 执行时环境变量错误导致的报错。
   - **第二份脚本**：调整 `HISTORY_LENGTH` 变量，删除无用的代码逻辑，确保机器人可以无记忆运行。

3. 将修改后的两份代码分别保存至独立文件夹，并压缩成 ZIP 文件，便于后续导入阿里云 FC。

---

## 使用阿里云搭建后端

### 1. 创建 OSS 存储

1. 登录 [阿里云 OSS 控制台](https://oss.console.aliyun.com/)。
2. 创建一个 OSS Bucket，用于存储 ChatGPT 的临时文件，收费几乎可忽略。

---

### 2. 配置阿里云 FC

#### 创建服务

1. 登录 [阿里云 FC 控制台](https://fcnext.console.aliyun.com/)。
2. 创建服务，推荐选择美国地区（例如弗吉尼亚）以避免 API 调用失败。
3. 服务名称建议命名为 `ChatGTP_Services`（如果更改名称，请同步修改脚本中的环境变量）。
4. **绑定日志功能**，方便后续调试。

#### 挂载 OSS

1. 将前面创建的 OSS Bucket 挂载到服务上，挂载路径建议为 `/mnt/oss`。
2. 确保服务已绑定角色权限（如 `AliyunFcDefaultRole`）。

---

### 3. 创建函数

#### 函数一：Dingtalk_Conversation

1. 创建新函数，名称为 `Dingtalk_Conversation`。
2. 上传修改后的 `Dingtalk_Conversation.py` 脚本 ZIP 文件。
3. 配置环境变量（示例配置如下）：
   ```json
   {
       "CHATGPT_FUNCTION": "Dingtalk_ChatGPT_Reply",
       "DINGTALK_APP_SECRET": "你的钉钉应用appSecret",
       "ENDPOINT": "你的阿里云函数计算Endpoint地址",
       "SERVICE_NAME": "ChatGTP_Services",
       "VERBOSE": "25"
   }
   ```
   - **Endpoint 地址**：根据地区选择，例如 `&lt;主账号ID&gt;.us-east-1-internal.fc.aliyuncs.com`。
   - **Service Name**：如果服务名称不同，请同步更改。

4. 设置触发器为默认配置。

#### 函数二：Dingtalk_ChatGPT_Reply

1. 创建新函数，名称为 `Dingtalk_ChatGPT_Reply`。
2. 上传修改后的 `Dingtalk_ChatGPT_Reply.py` 脚本 ZIP 文件。
3. 配置环境变量（示例配置如下）：
   ```json
   {
       "CHATGPT_API_KEY": "你的ChatGPT API Key",
       "HISTORY_LENGTH": "5",
       "OSS_MOUNT_POINT": "/mnt/oss",
       "TIMEOUT": "55",
       "VERBOSE": "25",
       "ENDPOINT": "https://api.openai.com",
       "USER_API_KEY": ""
   }
   ```
   - 若使用三方接口：修改 `ENDPOINT` 和 `USER_API_KEY`。

4. 提交并完成函数配置。

---

## 钉钉机器人配置

1. 登录钉钉，创建一个新的组织。
2. 在组织中创建机器人应用，填写以下内容：
   - **触发器地址**：复制阿里云 FC 中 `Dingtalk_Conversation` 函数的公网访问地址。
3. 发布应用，直接应用到钉钉群聊中，即可使用。

---

## 使用效果展示

完成配置后，在钉钉群中触发机器人即可实现与 ChatGPT 的交互。例如输入问题后，机器人将根据配置返回 GPT 的回答。

---

## 总结

通过阿里云 FC 与钉钉机器人的结合，可以实现高效、稳定的 ChatGPT 接入服务。这种架构适用于企业内部的自动化应用，也为个人开发者提供了强大的功能扩展支持。

如需虚拟卡支持，推荐使用 [WildCard虚拟信用卡](https://bit.ly/bewildcard)，轻松解决支付问题。

立即开始搭建你的智能钉钉助手吧！
