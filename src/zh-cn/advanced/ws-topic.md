> **[English](../../advanced/ws-topic.md)**

# WebSocket 主题数据通道

主题数据通道扩展了 WebSocket 控制平面，将实时数据流消息从 coordinator 代理到 CLI 客户端。CLI 命令如 `topic echo`、`topic hz` 和 `topic info` 通过现有的 WebSocket 连接以二进制帧接收消息数据，而无需直接 Zenoh 网络访问。

## 动机

| 场景 | 之前（Zenoh 直连） | 之后（WS 代理） |
|------|-------------------|-----------------|
| CLI 与 daemon 同机 | 可用 | 可用 |
| CLI 远程，Zenoh 可达 | 可用 | 可用 |
| CLI 远程，无 Zenoh 访问 | 不可用 | 可用 |
| 基于浏览器的 Web UI | 不可能 | 可能 |
| 嵌入式目标，无本地磁盘 | 无法本地录制 | `--proxy` 流式传输到 CLI |

## 架构

```
CLI  ──── WS（二进制帧）────>  Coordinator  ──── Zenoh 订阅 ────>  Daemon
                              (Zenoh 代理)                       (调试发布)
```

coordinator 充当 Zenoh 代理：

1. CLI 通过现有文本帧 WS 协议发送 `TopicSubscribe` 请求
2. coordinator 验证数据流并打开 Zenoh 订阅者
3. coordinator 将每个 Zenoh 样本作为二进制 WS 帧转发回 CLI
4. CLI 通过订阅 UUID 将二进制帧分派到相应的消费者

## 二进制数据帧

握手后，coordinator 推送二进制 WS 帧。每个帧有一个固定大小的头部：

```
[16 字节订阅 UUID][bincode 有效载荷]
```

---

详细内容请参见[英文完整版本](../../advanced/ws-topic.md)。
