## Connection

- URL: `ws://host:port/adb/{deviceId}`
- 建链方向：Agent模块（下称Server）主动建立并监听，ADB工具模块（下称Client）进行连接
- 消息格式：`jsonl`，每行一条 JSON
- `deviceId`: 设备的唯一标识符，暂由APP端决定

## Envelope

```json
{
  "type": "request" | "response",
  "message": "string",
  "data": "any",
  "requestId": "integer?"
}
```

说明：

- `requestId` 请求和响应关联同一个ID，Client connect请求固定为1，后续均为Server请求；Server第一次请求时使用2，之后递增；ping/pong心跳消息不需要携带 `requestId`，也不占用 `requestId` 序列。
- 支持并发请求，一次请求不需要等待响应就可以发送下一次请求。

## Client 请求 Server

### connect

手机端连接建立后第一条消息。

```json
{
  "type": "request",
  "message": "connect",
  "requestId": 1,
  "data": {
    "width": 1080,
    "height": 2400,
    "screenshot": "base64-or-null",
    "ui": "<hierarchy>...</hierarchy>",
    "currentPackage": "com.android.settings",
    "activity": "MainActivity"
  }
}
```

### ping

手机端心跳。

```json
{
  "type": "request",
  "message": "ping",
  "data": null
}
```

## Server 响应 Client

### pong

```json
{
  "type": "response",
  "message": "pong",
  "data": null
}
```

## Server 请求 Client

### observe

观察当前屏幕状态，获取屏幕信息。

```json
{
  "type": "request",
  "message": "observe",
  "requestId": "integer",
  "data": null
}
```

### launch

启动一个应用包，参数为包名。

```json
{
  "type": "request",
  "message": "launch",
  "requestId": "integer",
  "data": {
    "package": "com.android.settings"
  }
}
```

### tap

点击屏幕坐标 (x, y)。

```json
{
  "type": "request",
  "message": "tap",
  "requestId": "integer",
  "data": {
    "x": 300,
    "y": 500
  }
}
```

### type

输入文本，发送到当前焦点所在的输入框。

```json
{
  "type": "request",
  "message": "type",
  "requestId": "integer",
  "data": {
    "text": "hello"
  }
}
```

### swipe

从坐标 (startX, startY) 滑动到坐标 (endX, endY)。

```json
{
  "type": "request",
  "message": "swipe",
  "requestId": "integer",
  "data": {
    "startX": 500,
    "startY": 1600,
    "endX": 500,
    "endY": 500
  }
}
```

### longPress / doubleTap

长按和双击。

```json
{
  "type": "request",
  "message": "longPress",
  "requestId": "integer",
  "data": {
    "x": 300,
    "y": 500
  }
}
```

```json
{
  "type": "request",
  "message": "doubleTap",
  "requestId": "integer",
  "data": {
    "x": 300,
    "y": 500
  }
}
```

### keyevent

发送按键事件，支持多种操作。

参见 [Android KeyEvent](https://developer.android.com/reference/android/view/KeyEvent) 文档。[ADB指令详解覆盖设备应用日志与性能测试-开发者社区-阿里云](https://developer.aliyun.com/article/1611457)

```json
{
  "type": "request",
  "message": "keyevent",
  "requestId": "integer",
  "data": {
    "keyevent": "integer"
  }
}
```

### interact

让用户进行交互操作，例如完成验证码验证等。由llm生成提示语，指导用户完成操作后再继续后续流程。

该工具会导致Agent阻塞，直到用户完成交互操作并返回结果。

```json
{
  "type": "request",
  "message": "interact",
  "requestId": "integer",
  "data": {
    "message": "请用户完成验证码验证。"
  }
}
```

## 手机端 响应

所有请求成功时，手机端都应返回 `actionResult`，失败时返回 `error`。

### actionResult

```json
{
  "type": "response",
  "message": "actionResult",
  "requestId": "integer",
  "data": {
    "screenshot": "base64-or-null",
    "ui": "<hierarchy>...</hierarchy>",
    "currentPackage": "com.android.settings",
    "activity": "MainActivity"
  }
}
```

### error

```json
{
  "type": "response",
  "message": "error",
  "requestId": "integer",
  "data": {
    "message": "参数错误或执行失败原因",
    "screenshot": "base64-or-null",
    "ui": "<hierarchy>...</hierarchy>",
    "currentPackage": "com.android.settings",
    "activity": "MainActivity"
  }
}
```