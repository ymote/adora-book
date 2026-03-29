> **[English](../../operations/performance.md)**

# 性能

Adora 通过零拷贝共享内存 IPC、Apache Arrow 列式格式和 100% Rust 内核，实现了比 ROS2 Python 低 10-17 倍的延迟。

## 架构优势

| 层 | Adora | ROS2 (rclpy) |
|----|-------|---------------|
| 运行时 | Rust 异步（tokio） | Python + C++ 中间件 |
| IPC（>4KB） | Zenoh SHM 零拷贝 | DDS 序列化 + 拷贝 |
| IPC（<4KB） | TCP + bincode | DDS 序列化 + 拷贝 |
| 数据格式 | Apache Arrow（零序列化） | CDR 序列化 |
| 线程 | 无锁通道（flume） | GIL 绑定的回调 |
| 扇出 | Arc 包装（每接收者 O(1)） | 每接收者拷贝 |

## 基准测试套件

### 内部基准测试（`examples/benchmark/`）

测量 Adora 在 10 种有效载荷大小（0B 到 4MB）下的延迟和吞吐量。

```bash
cd examples/benchmark
./compare.sh          # Rust vs Python 发送者对比
```

### ROS2 对比（`examples/ros2-comparison/`）

使用两个框架上相同的 Python 工作负载进行同等对比。

```bash
cd examples/ros2-comparison
./run_comparison.sh   # 需要 ROS2 Humble+
```

## 性能调优

### 队列大小

默认队列大小为 10。对于高吞吐量输出，增加它：

```yaml
inputs:
  data:
    source: producer/output
    queue_size: 1000
```

### 背压

对于不能丢失消息的输入，使用 `backpressure` 策略：

```yaml
inputs:
  commands:
    source: controller/cmd
    queue_size: 100
    queue_policy: backpressure
```

### CPU 亲和性

将节点固定到特定 CPU 核心（仅 Linux）：

```yaml
- id: controller
  path: ./controller
  cpu_affinity: [2, 3]
```

### 实时调优

```bash
sudo adora daemon --rt
```

---

详细内容请参见[英文完整版本](../../operations/performance.md)。
