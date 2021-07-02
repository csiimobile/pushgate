# 推送通知网关

## 支持平台

- [苹果APNS](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html)
- [安卓FCM](https://firebase.google.com/)
- [华为HMS](https://developer.huawei.com/consumer/cn/hms/)

## 功能特性

- 支持境外安卓 [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging) 。
- 支持苹果iOS/Mac [Apple Push Notification Service](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html) 。
- 支持华为 [HMS Push Service](https://developer.huawei.com/consumer/en/hms/huawei-pushkit) ，向安卓和鸿蒙设备发送通知。
- 支持集中配置管理。
- 支持通过命令行发送单条推送通知。
- 支持通过RESTful API发送推送通知。
- 支持HTTP/2 和 HTTP/1.1 协议。
- 支持消息队列及多路并发。
- 支持发送状态统计。

### 发送安卓通知

使用如下命令发送单条安卓通知：

```bash
pushgate -android -m "your message" -k "API Key" -t "Device token"
```

发送通知到主题：

```bash
pushgate --android --topic "/topics/foo-bar" \
  -m "This is a Firebase Cloud Messaging Topic Message" \
  -k your_api_key
```

- `-m`: 通知消息正文
- `-k`: [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging) 密钥（api key）
- `-t`: 设备令牌
- `--title`: 通知消息标题
- `--topic`: 发送消息到主题，此模式下无需指定设备令牌
- `--proxy`: 设置代理服务器

### 发送华为（HMS）推送通知

使用如下命令发送单条华为通知：

```bash
pushgate -huawei -title "Notification Title" -m "your message" -hk "API Key" -hid "App ID" -t "Device token"
```

发送通知到主题：

```bash
pushgate --huawei --topic "foo-bar" \
  -title "Notification Title" \
  -m "This is a Huawei Mobile Services Topic Message" \
  -hk "API Key" \
  -hid "App ID"
```

- `-m`: 通知消息正文
- `-hk`: [Huawei Mobile Services](https://developer.huawei.com/consumer/en/doc/development/HMS-Guides/Preparations) 密钥（api secret）
- `-t`: 设备令牌
- `--title`: 通知消息标题
- `--topic`: 发送消息到主题，此模式下无需指定设备令牌
- `--proxy`: 设置代理服务器

### 发送苹果（iOS/Mac）通知

使用如下命令发送单条苹果通知：

```bash
pushgate -ios -m "your message" -i "your certificate path" \
  -t "device token" --topic "apns topic"
```

- `-m`: 通知消息正文
- `-i`: 证书路径 (`pem` 或 `p12` 文件)
- `-t`: 设备令牌
- `--title`: 通知消息标题
- `--topic`: 通知主题
- `--password`: 证书密码

默认发送到开发环境。 添加 `-production` 参数可以发送到生产环境。

```bash
pushgate -ios -m "your message" -i "your certificate path" \
  -t "device token" \
  -production
```

## 启动网关服务

请确认配置文件 [config.yml](config.yml) 存在。 默认端口为 `8088`。

```bash
# 加载默认配置文件，同路径下的config.yml
pushgate
# 加载指定的配置文件
pushgate -c path/config.yml
```

## 网关API

推送通知网关支持如下API：

- **GET**  `/api/stat/app` 获取推送通知的成功、失败计数信息
- **POST** `/api/push` 发送推送通知


### GET /api/stat/app

获取推送通知的成功和失败计数。

```json
{
  "version": "v1.0.0",
  "queue_max": 8192,
  "queue_usage": 0,
  "total_count": 77,
  "ios": {
    "push_success": 19,
    "push_error": 38
  },
  "android": {
    "push_success": 10,
    "push_error": 10
  },
  "huawei": {
    "push_success": 3,
    "push_error": 1
  }
}
```

### POST /api/push

通过API发送通知。

发送苹果推送通知, `platform` 设置为 `1`:

```json
{
  "notifications": [
    {
      "tokens": ["token_a", "token_b"],
      "platform": 1,
      "message": "Hello World iOS!"
    }
  ]
}
```

发送境外安卓通知，`platform` 设置为 `2`:

```json
{
  "notifications": [
    {
      "tokens": ["token_a", "token_b"],
      "platform": 2,
      "message": "Hello World Android!"
    }
  ]
}
```

发送华为通知消息, `platform` 设置为 `3`:

```json
{
  "notifications": [
    {
      "tokens": ["token_a", "token_b"],
      "platform": 3,
      "title": "Hello with HMS",
      "message": "Hello World Huawei!"
    }
  ]
}
```

同时发送多条多平台通知：

```json
{
  "notifications": [
    {
      "tokens": ["token_a", "token_b"],
      "platform": 1,
      "message": "Hello World iOS!"
    },
    {
      "tokens": ["token_a", "token_b"],
      "platform": 2,
      "message": "Hello World Android!"
    },
    {
      "tokens": ["token_a", "token_b"],
      "platform": 3,
      "message": "Hello World Huawei!",
      "title": "Hello with HMS"
    },
    .....
  ]
}
```

