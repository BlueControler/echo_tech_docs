# Connection

- URL: `ws://host:port/system/{deviceId}`
- 建链方向：Agent模块（下称Server）主动建立并监听，系统工具模块（下称Client）进行连接
- 消息格式：jsonl，每行一条 JSON
- `deviceId`: 设备的唯一标识符，暂由APP端决定

# Envelope

- 本协议的所有请求和响应成对列举，消息格式如下：

```json
{
  "type": "request" | "response",
  "message": "string",
  "data": "any",
  "requestId": "integer?"
}
```

说明：
- requestId 请求和响应关联同一个ID，Server第一次请求时使用1，之后递增；ping/pong心跳消息不需要携带 requestId，也不占用 requestId 序列。
- 支持并发请求，一次请求不需要等待响应就可以发送下一次请求。

# Client 请求

**ping**

手机端心跳。

```json
{
  "type": "request",
  "message": "ping",
  "data": null
}
```

Server 响应：

**pong**

```json
{
  "type": "response",
  "message": "pong",
  "data": null
}
```

# Server 请求

下文的响应仅列出成功的情况，失败结果统一如下：

```json
{
  "type": "response",
  "message": "string", // 与请求的 message 相同
  "data": {
    "error": "string"
  },
  "requestId": "integer"
}
```

## listApps

列出所有已安装的应用。

type 可选值说明：
- all：列出所有应用
- third：列出第三方应用
- system：列出系统应用

**Request**

```json
{
  "type": "request",
  "message": "listApps",
  "data": {
    "type": "all" | "third" | "system"
  },
  "requestId": "integer"
}
```

**Response**

```json
{
  "type": "response",
  "message": "listApps",
  // 包含应用包名(key)和应用名称(value)的键值对
  "data": "map[string, string]",
  "requestId": "integer"
}
```

示例：

```json
{
  "type": "response",
  "message": "listApps",
  "data": {
    "com.tencent.mm": "微信",
    "com.alipay.mobile": "支付宝"
  },
  "requestId": 1
}
```

## 日程

对日程的增删改查。数据模型如下：

**Event**

节选一部分，参见[CalendarContract.EventsColumns | Android Developers](https://spot.pcc.edu/~mgoodman/developer.android.com/reference/android/provider/CalendarContract.EventsColumns.html)

```json
{
  "_id": "integer", // 日程ID，唯一标识一个日程，由系统分配，创建时不需要指定
  "title": "string",
  "description": "string",
  "eventLocation": "string",
  "dtstart": "timestamp", // Unix时间戳，单位毫秒
  "dtend": "timestamp",
  "allDay": "boolean",
  "eventTimezone": "string", // 时区ID，例如"Asia/Shanghai"
  "duration": "string", // 日程的时长，格式为 RFC2445，例如 PT1H (1小时)
  "rrule": "string", // 日程的重复规则，例如 FREQ=DAILY;INTERVAL=1 (每天重复)
  "availability": "busy" | "free" | "tentative", // 日程的状态，分别表示忙、空闲和待定
  "status": "confirmed" | "tentative" | "cancelled" // 日程的确认状态，分别表示已确认、待定和已取消
}
```

### 创建一个日程

Request：

```json
{
  "type": "request",
  "message": "createEvent",
  "data": {
    "event": "Event",
  },
  "requestId": "integer"
}
```

Response：

```json
{
  "type": "response",
  "message": "createEvent",
  "data": {
    "id": "integer" // 日程ID，唯一标识一个日程
  },
  "requestId": "integer"
}
```

### 列出某个时间段内的日程

查询所有开始时间或结束时间在指定时间段内的日程。

Request：

```json
{
  "type": "request",
  "message": "listEvents",
  "data": {
    "start": "timestamp", // 起始时间，Unix时间戳，单位毫秒
    "end": "timestamp" // 结束时间，Unix时间戳，单位毫秒
  },
  "requestId": "integer"
}
```

Response：

```json
{
  "type": "response",
  "message": "listEvents",
  "data": "array<Event>",
  "requestId": "integer"
}
```

### 更新一个日程

更新一个已存在的日程，必须指定日程ID，如果需要删除日程，仅需更新Status为cancelled。

Request：

```json
{
  "type": "request",
  "message": "updateEvent",
  "data": {
    "event": "Event",
  },
  "requestId": "integer"
}
```

Response：

```json
{
  "type": "response",
  "message": "updateEvent",
  "data": null,
  "requestId": "integer"
}
```

## 提醒

**Reminder**

参见[CalendarContract.RemindersColumns | Android Developers](https://spot.pcc.edu/~mgoodman/developer.android.com/reference/android/provider/CalendarContract.RemindersColumns.html)

```json
{
  "minutes": "integer", // 提前提醒的分钟数，例如 10 表示提前10分钟提醒
  "method": "alert" | "alarm" // 提醒方式，分别表示弹出通知和响铃，其他方式暂不支持
}
```

### 列出一个日程的所有提醒

Request：

```json
{
  "type": "request",
  "message": "listReminders",
  "data": {
    "eventId": "integer" // 日程ID，唯一标识一个日程
  },
  "requestId": "integer"
}
```

Response：

```json
{
  "type": "response",
  "message": "listReminders",
  "data": "array<Reminder>",
  "requestId": "integer"
}
```

### 给一个日程修改提醒

传入哪些提醒，就设置成哪些提醒，之前的提醒会被覆盖掉。如果传入空列表，就删除所有提醒。

Request：

```json
{
  "type": "request",
  "message": "updateReminders",
  "data": {
    "eventId": "integer", // 日程ID，唯一标识一个日程
    "reminders": "array<Reminder>" // 提醒列表，一个日程可以有多个提醒
  },
  "requestId": "integer"
}
```

Response：

```json
{
  "type": "response",
  "message": "updateReminders",
  "data": null,
  "requestId": "integer"
}
```

## 位置信息（定位）

Request：

```json
{
  "type": "request",
  "message": "getLocation",
  "data": null,
  "requestId": "integer"
}
```

Response：

```json
{
  "type": "response",
  "message": "getLocation",
  "data": {
    "latitude": "double", // 纬度
    "longitude": "double", // 经度
    "accuracy": "float", // 定位精度，单位米
    "timestamp": "timestamp" // 定位时间，Unix时间戳，单位毫秒
  },
  "requestId": "integer"
}
```

## 传感器

待定。

**Android 传感器数据获取**

Android 通过 **`SensorManager`** 和 **`SensorEventListener`** 来获取传感器数据，支持加速度计、陀螺仪、磁力计、光线传感器等多种类型。

**支持的主要传感器类型**

Android 平台支持三大类传感器：

| 类别           | 传感器类型                 | 说明                                                  | 常见用途         |
| -------------- | -------------------------- | ----------------------------------------------------- | ---------------- |
| **运动传感器** | `TYPE_ACCELEROMETER`       | 加速度计，测量三个轴向上的加速力（含重力），单位 m/s² | 检测摇晃、倾斜   |
|                | `TYPE_GYROSCOPE`           | 陀螺仪，测量旋转速率，单位 rad/s                      | 检测旋转、转动   |
|                | `TYPE_GRAVITY`             | 重力传感器（软件合成）                                | 检测重力方向     |
|                | `TYPE_LINEAR_ACCELERATION` | 线性加速度（不含重力）                                | 监测单轴加速度   |
|                | `TYPE_ROTATION_VECTOR`     | 旋转矢量传感器                                        | 检测屏幕方向     |
| **环境传感器** | `TYPE_LIGHT`               | 光线传感器，单位 lx                                   | 自动调节屏幕亮度 |
|                | `TYPE_PRESSURE`            | 气压计，单位 hPa/mbar                                 | 监测气压变化     |
|                | `TYPE_AMBIENT_TEMPERATURE` | 环境温度，单位 °C                                     | 监测气温         |
|                | `TYPE_PROXIMITY`           | 距离传感器，单位 cm                                   | 通话时息屏       |
| **位置传感器** | `TYPE_MAGNETIC_FIELD`      | 磁力计（电子罗盘），单位 μT                           | 指南针           |

> **注意**：很少有设备拥有所有类型的传感器。使用前应检查传感器是否存在。
