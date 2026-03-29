> **[English](../../operations/logging.md)**

# 日志

Adora 为实时机器人和 AI 数据流提供结构化日志系统。日志以结构化 JSONL 文件的形式按节点捕获，转发到 coordinator 用于实时流式传输，并可选择作为数据消息通过数据流图路由。

## 应该使用哪种日志方式？

| 我想要... | 方式 | 配置 |
|-----------|------|------|
| **从 Python 记录日志** | 使用 Python 的 `logging` 模块（自动桥接） | 无 -- 只需 `import logging` |
| **从 Rust 记录日志** | 使用 `node.log_info()` / `node.log_error()` 等 | 无 -- 开箱即用 |
| **从 C/C++ 记录日志** | 使用 `adora_log()` / `log_message()` | 无 -- 开箱即用 |
| **过滤嘈杂的节点** | 在 YAML 中设置 `min_log_level` | 每节点 YAML 字段 |
| **在一个地方查看所有日志** | 订阅 `adora/logs` 虚拟输入 | `inputs: logs: adora/logs` |
| **将某个节点的日志作为数据处理** | 在该节点上使用 `send_logs_as` | 每节点 YAML + 连接输出 |
| **轮转日志文件** | 在 YAML 中设置 `max_log_size` | 每节点 YAML 字段 |

### 各语言快速入门

**Python** -- 最简单的方式是使用 Python 内置的 `logging` 模块：

```python
import logging
from adora import Node

node = Node()  # 自动将 Python logging 桥接到 adora

logging.info("传感器已启动")
logging.warning("高温：42C")
print("原始调试输出")            # 捕获为 "stdout" 级别
```

**Rust** -- 使用节点 API 便捷方法：

```rust
let (node, mut events) = AdoraNode::init_from_env()?;
node.log_info("传感器已启动");
node.log_warn("温度过高");
```

**C** -- 使用 `adora_log()` 函数：

```c
adora_log(ctx, "info", 4, "Sensor started", 14);
```

**C++** -- 使用 `log_message()` 函数：

```cpp
log_message(node.send_output, "info", "Sensor started");
```

---

## 功能概览

| 功能 | 范围 | 配置 |
|------|------|------|
| 日志级别过滤 | CLI 显示 | `--log-level`、`ADORA_LOG_LEVEL` |
| 输出格式 | CLI 显示 | `--log-format`、`ADORA_LOG_FORMAT` |
| 每节点级别覆盖 | CLI 显示 | `--log-filter`、`ADORA_LOG_FILTER` |
| 源级过滤 | 每节点 YAML | `min_log_level` |
| 日志聚合 | 数据流输入 | `adora/logs` 虚拟输入 |
| 实时日志流 | `adora logs` | `--follow` |

---

详细内容请参见[英文完整版本](../../operations/logging.md)。
