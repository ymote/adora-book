> **[English](../../operations/cli.md)**

# Adora CLI 参考

Adora（AI-Dora，Dataflow-Oriented Robotic Architecture）是一个 100% Rust 编写的实时机器人与 AI 应用框架。本文档涵盖 `adora` CLI 的使用。

## 快速入门

```bash
# 创建新项目
adora new my-robot --kind dataflow --lang rust

# 本地运行（无需 coordinator/daemon）
adora run dataflow.yml

# 或使用 coordinator/daemon 用于生产环境
adora up
adora start dataflow.yml --attach
# Ctrl-C 停止
adora down
```

## 核心概念

### 数据流

**数据流**是由节点组成的有向图，节点之间通过类型化数据通道连接。节点产生**输出**，其他节点作为**输入**消费。框架处理数据路由、序列化（Apache Arrow）和生命周期管理。

### 执行模式

| 模式 | 命令 | 基础设施 | 用途 |
|------|------|----------|------|
| **本地** | `adora run` | 无 | 开发、测试、单机 |
| **分布式** | `adora up` + `adora start` | Coordinator + Daemon | 生产环境、多机器 |

### 组件角色

```
CLI  -->  Coordinator  -->  Daemon(s)  -->  节点 / 算子
              (控制平面)     (每台机器)       (用户代码)
```

- **CLI**：用户界面。发送命令，显示日志。
- **Coordinator**：跨机器编排数据流生命周期。
- **Daemon**：生成节点进程，管理 IPC，收集指标。
- **节点**：产生和消费 Arrow 数据的独立进程。
- **算子**：在共享运行时内运行的进程内代码（比节点延迟更低）。

### 数据格式

所有数据以 **Apache Arrow** 列式数组的形式在系统中流动。这使得同机节点之间可以进行零拷贝共享内存传输，且零序列化开销。

---

详细的命令参考、环境变量、架构指南和故障排除请参见[英文完整版本](../../operations/cli.md)。
