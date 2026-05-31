本文档维护后端暴露的自定义HTTP API，前端按需接入。

## GET /adb/{deviceId}/status

获取当前adb服务连接状态。

Request 无

**成功**

```json
{
    "connected": true,
    "width": integer | null,
    "height": integer | null,
    "currentPackage": string | null,
    "activity": string | null
}
```

**失败**

```json
{
    "connected": false
}
```

## GET /system/{deviceId}/status

获取当前系统API服务连接状态。

Request 无

**成功**

```json
{
    "connected": true,
    "path": string,
    "remoteAddress": string,
}
```

**失败**

```json
{
    "connected": false
}
```

## GET /network/{deviceId}/status

查询网络状态。

Request 无

**成功**

```json
{
    "networkConnected": boolean,
    "mode": "cloud" | "local",
    "localServerRunning": boolean,
    "localBaseUrl": string | null,
    "localModelName": string | null,
    "localModelPath": string | null,
    "lastError": string,
}
```

## POST /network/{deviceId}/status

更新网络状态。

Request

```json
{
    "connected": boolean,
}
```

Response

成功同`GET /network/{deviceId}/status`，失败很多，忽略就行。