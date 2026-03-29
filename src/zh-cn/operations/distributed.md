> **[English](../../operations/distributed.md)**

# 分布式部署指南

Adora 支持跨多台机器部署数据流，适用于多机器人车队、边缘 AI 管道和分布式机器人系统。

## 概述

Adora 的分布式架构有三层：

```
CLI  -->  Coordinator  -->  Daemon(s)  -->  节点 / 算子
              (一个)         (每台机器)       (用户代码)
```

- **CLI** 向 coordinator 发送控制命令（构建、启动、停止）。
- **Coordinator** 编排 daemon，解析节点放置，管理数据流生命周期。
- **Daemon** 在每台机器上运行，生成和监管节点进程。
- **节点** 通过共享内存（同机）或 Zenoh 发布/订阅（跨机器）通信。

## 快速入门

1. 创建 `cluster.yml`：

```yaml
coordinator:
  addr: 10.0.0.1
machines:
  - id: robot
    host: 10.0.0.2
    user: ubuntu
  - id: gpu-server
    host: 10.0.0.3
    user: ubuntu
```

2. 启动集群：

```bash
adora cluster up cluster.yml
```

3. 启动数据流：

```bash
adora start dataflow.yml --name my-app --attach
```

4. 检查集群健康：

```bash
adora cluster status
```

5. 关闭：

```bash
adora cluster down
```

## 节点调度

### 基于机器的调度

```yaml
- id: camera-driver
  _unstable_deploy:
    machine: robot-arm
```

### 基于标签的调度

```yaml
- id: ml-inference
  _unstable_deploy:
    labels:
      gpu: "true"
      arch: "arm64"
```

## 二进制分发

三种策略：
- **Local** -- 每个 daemon 从源码构建（默认）
- **Scp** -- CLI 通过 SSH/SCP 推送构建的二进制文件
- **Http** -- daemon 从 coordinator 的 `/api/artifacts` 端点拉取

---

详细内容请参见[英文完整版本](../../operations/distributed.md)。
