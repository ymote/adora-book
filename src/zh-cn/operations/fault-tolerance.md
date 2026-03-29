> **[English](../../operations/fault-tolerance.md)**

# 容错机制

Adora 为机器人和 AI 数据流提供内置容错功能。节点可以在故障时自动重启，检测上游连接超时，在输入不可用时优雅降级，coordinator 可以将状态持久化到磁盘以在崩溃和重启后恢复。

## 功能概览

| 功能 | 范围 | 配置 |
|------|------|------|
| 重启策略 | 每节点 | `restart_policy`、`max_restarts`、`restart_delay` 等 |
| 健康监控 | 每节点 | `health_check_timeout`、`health_check_interval`（数据流级） |
| 输入超时 | 每输入 | `input_timeout` |
| 断路器 | 自动 | 由 `input_timeout` 触发，自动恢复 |
| NodeRestarted 事件 | 下游节点 | 上游重启时自动发送 |
| InputTracker API | Rust 节点 | `adora_node_api::InputTracker` |
| Coordinator 状态持久化 | Coordinator | `--store redb` |

---

## 重启策略

### 配置

```yaml
nodes:
  - id: my-node
    path: ./target/debug/my-node
    restart_policy: on-failure  # never | on-failure | always
    max_restarts: 5             # 0 = 无限（默认：0）
    restart_delay: 1.0          # 初始延迟（秒）
    max_restart_delay: 30.0     # 指数退避上限
    restart_window: 300.0       # 此秒数后重置计数器
```

### 策略类型

**`never`**（默认）-- 节点不会重启。故障正常传播。

**`on-failure`** -- 仅在节点以非零退出码退出时重启。正常退出（代码 0）不会重启。

**`always`** -- 在任何退出时重启，除非：
- 数据流被用户停止
- 所有输入关闭且节点以非零代码退出

### 退避

设置 `restart_delay` 后，daemon 在重启前等待。延迟每次尝试翻倍（指数退避），受 `max_restart_delay` 限制。

---

## 健康监控

```yaml
health_check_interval: 2.0  # 秒（默认：5.0，数据流级）
nodes:
  - id: my-node
    health_check_timeout: 30.0  # 秒（每节点）
    restart_policy: on-failure
```

daemon 在配置的间隔运行健康检查扫描。如果节点在超时时间内未与 daemon 通信，该进程将被终止并评估重启策略。

---

## 输入超时与断路器

```yaml
inputs:
  sensor_data:
    source: camera-node/frames
    input_timeout: 5.0  # 秒
```

### 工作原理

1. 如果在超时期间没有数据到达，输入被标记为"断开"
2. daemon 向节点发送 `InputClosed { id }` 事件
3. 当上游恢复并发送新数据时，断路器自动恢复
4. 节点收到 `InputRecovered { id }` 事件

---

## InputTracker API（Rust）

```rust
use adora_node_api::{AdoraNode, Event, InputTracker};

let (mut node, mut events) = AdoraNode::init_from_env()?;
let mut tracker = InputTracker::new();

while let Some(event) = events.recv() {
    tracker.process_event(&event);
    match event {
        Event::Input { id, data, .. } => { /* 正常处理 */ }
        Event::InputClosed { id } => {
            // 使用缓存数据作为回退
            if let Some(stale_data) = tracker.last_value(&id) {
                // 使用过期数据
            }
        }
        Event::Stop(_) => break,
        _ => {}
    }
}
```

---

## Coordinator 状态持久化

```bash
# 使用默认路径（~/.adora/coordinator.redb）
adora coordinator --store redb

# 使用自定义路径
adora coordinator --store redb:/path/to/coordinator.redb
```

启用后，coordinator 在每次状态转换时将数据流记录持久化到磁盘。重启时，它会恢复之前运行的状态并将过期的数据流标记为失败。

---

## 最佳实践

- **从 `on-failure` 开始**。仅对预期退出并重启的节点使用 `always`。
- **设置 `max_restarts`**。无限重启可能掩盖 bug。
- **使用 `restart_window`**。防止永久重启循环。
- **调优 `restart_delay`**。从 0.5-1.0 秒开始。
- **慷慨设置 `health_check_timeout`**。至少是节点最长预期处理时间的 2-3 倍。
- **为关键路径使用 `InputTracker`**。当节点必须在降级输入下继续运行时。
- **生产部署使用 `--store redb`**。确保 coordinator 在崩溃后保留数据流历史。

---

详细内容和完整用例请参见[英文完整版本](../../operations/fault-tolerance.md)。
