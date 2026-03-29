> **[English](../../operations/debugging.md)**

# 调试与可观测性指南

本指南介绍如何调试、录制、回放和监控 adora 数据流。

---

## 快速调试检查清单

当出现问题时，按以下顺序操作：

```bash
# 1. 运行完整环境诊断
adora doctor --dataflow dataflow.yml

# 2. 查看活跃的数据流
adora list

# 3. 检查问题节点
adora node info -d my-dataflow problem-node

# 4. 检查节点资源使用
adora top

# 5. 流式查看问题节点的日志
adora logs my-dataflow problem-node --follow --level debug

# 6. 节点是否在产生输出？
adora topic echo -d my-dataflow problem-node/output

# 7. 注入测试数据
adora topic pub -d my-dataflow problem-node/input '[1, 2, 3]'

# 8. 是否以预期频率发布？
adora topic hz -d my-dataflow --window 5

# 9. 检查/修改运行时参数
adora param list -d my-dataflow problem-node
adora param set -d my-dataflow problem-node debug_level 2

# 10. 重启行为异常的节点（不停止数据流）
adora node restart -d my-dataflow problem-node

# 11. 查看 coordinator 追踪（无需外部基础设施）
adora trace list
adora trace view <trace-id-prefix>

# 12. 可视化数据流图
adora graph dataflow.yml --open

# 13. 录制用于离线分析
adora record dataflow.yml -o debug-capture.adorec
```

---

## 录制与回放

录制捕获实时数据流消息到文件。回放用录制的数据替代源节点，让您无需硬件即可重现行为。

### 录制数据流

```bash
# 录制所有主题
adora record dataflow.yml

# 指定输出文件
adora record dataflow.yml -o my-capture.adorec
```

### 回放录制

```bash
# 以原始速度回放
adora replay recording.adorec

# 以 2 倍速回放
adora replay recording.adorec --speed 2.0

# 尽可能快地回放
adora replay recording.adorec --speed 0
```

### 选择性回放

仅替换特定源节点，保持其他节点实时运行：

```bash
adora replay recording.adorec --replace sensor
```

---

## 主题检查

主题检查命令需要 `--debug` 标志或 `publish_all_messages_to_zenoh: true`。

### 回显主题数据

```bash
adora topic echo -d my-dataflow camera_node/image
```

### 测量频率

```bash
adora topic hz -d my-dataflow --window 10
```

### 发布测试数据

```bash
adora topic pub -d my-dataflow sensor/threshold '[42]'
```

---

## 资源监控

```bash
# 默认 2 秒刷新
adora top

# JSON 快照用于脚本/CI
adora top --once | jq .
```

显示每个节点的 CPU 使用率、内存（RSS）、节点状态、重启次数、队列深度、网络 TX/RX。

---

## 日志分析

```bash
# 流式查看特定节点的日志
adora logs my-dataflow sensor-node --follow

# 按日志级别过滤
adora logs my-dataflow sensor-node --follow --level debug

# 使用 grep 过滤
adora logs my-dataflow --all-nodes --follow --grep "error"
```

---

## 数据流可视化

```bash
# 生成 HTML 并在浏览器中打开
adora graph dataflow.yml --open

# 生成 Mermaid 图表文本
adora graph dataflow.yml --mermaid
```

---

详细内容请参见[英文完整版本](../../operations/debugging.md)。
