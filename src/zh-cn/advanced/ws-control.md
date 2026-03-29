> **[English](../../advanced/ws-control.md)**

# WebSocket 控制平面

Adora 的控制平面使用 WebSocket 连接进行 CLI、coordinator 和 daemon 之间的所有通信。单个 Axum 服务器在一个端口上暴露三个路由，取代了之前的多端口 TCP 设计。JSON 文本帧承载 UUID 关联的请求-应答协议，以及用于日志流的即发即忘事件。

## 功能概览

| 功能 | 详情 |
|------|------|
| 路由 | `/api/control`（CLI）、`/api/daemon`（daemon）、`/health` |
| 传输格式 | JSON 文本帧 + 主题数据的二进制帧 |
| 协议 | UUID 关联的请求-应答 + 即发即忘事件 |
| 消息大小限制 | 1 MiB |
| 并发限制 | 256 连接 |
| 服务器框架 | Axum + Tower 中间件 |

## 架构

```
                        单一 Axum 服务器（一个端口）
                       ┌────────────────────────────┐
                       │  /api/control   (CLI)       │
  CLI ──── WS ────────>│  /api/daemon    (Daemon)    │
                       │  /health        (HTTP GET)  │
  Daemon ── WS ───────>│                             │
                       └──────────┬─────────────────┘
                                  │ mpsc::Sender<Event>
                                  v
                            Coordinator
                          (事件循环)
```

## 传输协议

所有消息都是 JSON 文本帧。存在三种消息形式：

### WsRequest（客户端 -> 服务器）

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "method": "control",
  "params": { "List": null }
}
```

### WsResponse（服务器 -> 客户端）

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "result": { "DataflowList": [] }
}
```

### WsEvent（双向）

```json
{
  "event": "log",
  "payload": { "message": "传感器已启动", "level": "info" }
}
```

---

详细内容请参见[英文完整版本](../../advanced/ws-control.md)。
