# Telegram Bot 发送图片问题排查与解决方案

> Troubleshooting Guide for Telegram Bot Image Sending Issues

## 问题描述 | Problem Description

在使用 Telegram Bot API 发送图片时，可能会遇到以下错误：

- `Bad Request: wrong file_id specified`
- 图片无法正常显示
- `sendPhoto` 请求返回失败

## 常见原因 | Common Causes

1. **file_id 过期或无效** - Telegram 的 file_id 有时效性，不同 Bot 之间的 file_id 不通用
2. **URL 不可访问** - 图片 URL 必须是公网可访问的
3. **文件格式不支持** - Telegram 对图片格式和大小有限制
4. **网络问题** - 服务器无法访问 Telegram API

## 解决方案 | Solutions

### 方案一：使用 URL 发送（推荐）

最可靠的方式是使用公网可访问的图片 URL：

```python
import requests

TOKEN = "your_bot_token"
CHAT_ID = "target_chat_id"

# 使用 URL 发送图片
response = requests.post(
    f"https://api.telegram.org/bot{TOKEN}/sendPhoto",
    data={
        "chat_id": CHAT_ID,
        "photo": "https://example.com/image.jpg"
    }
)
print(response.json())
```

### 方案二：上传本地文件

使用 `multipart/form-data` 上传本地文件：

```python
import requests

TOKEN = "your_bot_token"
CHAT_ID = "target_chat_id"

# 上传本地文件
with open("image.jpg", "rb") as photo:
    response = requests.post(
        f"https://api.telegram.org/bot{TOKEN}/sendPhoto",
        data={"chat_id": CHAT_ID},
        files={"photo": photo}
    )
print(response.json())
```

### 方案三：使用 file_id（需谨慎）

如果必须使用 file_id，请确保：

1. file_id 来自同一个 Bot
2. file_id 是最近获取的
3. 文件仍然存在于 Telegram 服务器

```python
import requests

TOKEN = "your_bot_token"
CHAT_ID = "target_chat_id"
FILE_ID = "AgACAgIAAxkB..."  # 从之前的消息中获取

response = requests.post(
    f"https://api.telegram.org/bot{TOKEN}/sendPhoto",
    data={
        "chat_id": CHAT_ID,
        "photo": FILE_ID
    }
)
print(response.json())
```

## 调试技巧 | Debugging Tips

### 1. 检查 API 响应

```python
response = requests.post(...)
print(f"Status: {response.status_code}")
print(f"Response: {response.json()}")
```

### 2. 验证图片 URL 可访问性

```bash
curl -I "https://example.com/image.jpg"
```

### 3. 检查文件大小

Telegram 限制：
- 图片：最大 10 MB
- 文档：最大 50 MB

## 最佳实践 | Best Practices

1. **优先使用 URL 方式** - 最稳定可靠
2. **本地文件用 multipart 上传** - 不要尝试 base64 编码
3. **缓存 file_id** - 如果需要重复发送同一图片，保存首次发送后返回的 file_id
4. **错误处理** - 始终检查 API 响应，处理可能的失败情况
5. **使用代理** - 如果在中国大陆，可能需要配置代理访问 Telegram API

## 相关链接 | References

- [Telegram Bot API - sendPhoto](https://core.telegram.org/bots/api#sendphoto)
- [Telegram Bot API - File](https://core.telegram.org/bots/api#file)

---

如果这篇指南对你有帮助，欢迎 Star ⭐

If this guide helps you, please give it a Star ⭐
