> **[English](../../development/testing.md)**

# Adora 测试指南

本指南介绍如何在 Adora 工作空间中运行、编写和排查测试。

## 快速入门（5 分钟验证）

运行以下三个命令来验证工作空间是否健康：

```bash
# 1. 格式检查（约 5 秒）
cargo fmt --all -- --check

# 2. 代码检查（首次约 60 秒，之后使用缓存）
cargo clippy --all \
  --exclude adora-node-api-python \
  --exclude adora-operator-api-python \
  --exclude adora-ros2-bridge-python \
  -- -D warnings

# 3. 单元 + 集成测试（首次约 90 秒）
cargo test --all \
  --exclude adora-node-api-python \
  --exclude adora-operator-api-python \
  --exclude adora-ros2-bridge-python
```

提交 PR 前这三项必须全部通过。Python 包因需要 maturin 而被排除。

## 测试层级

| 层级 | 覆盖内容 | 命令 | 速度 |
|------|----------|------|------|
| **格式** | 代码风格 | `cargo fmt --all -- --check` | 约 5 秒 |
| **代码检查** | 警告、正确性 | `cargo clippy --all ...` | 约 60 秒 |
| **单元测试** | 单个函数 | `cargo test --all ...` | 约 90 秒 |
| **CLI** | 命令解析、验证 | `cargo test -p adora-cli` | 约 5 秒 |
| **集成** | 通过环境变量的节点 I/O | `cargo test --test example-tests` | 约 30 秒 |
| **冒烟测试** | 完整 CLI 生命周期 | `cargo test --test example-smoke -- --test-threads=1` | 约 3 分钟 |
| **端到端** | 多数据流场景 | `cargo test --test ws-cli-e2e -- --ignored --test-threads=1` | 约 2 分钟 |
| **容错** | 重启策略、超时 | `cargo test --test fault-tolerance-e2e` | 约 45 秒 |

## 单元测试

单元测试位于使用 `#[cfg(test)]` 模块的代码旁边。运行单个 crate 的测试：

```bash
cargo test -p adora-cli
cargo test -p adora-core
cargo test -p adora-arrow-convert
```

## 集成测试（节点 I/O）

文件：`tests/example-tests.rs`

这些测试使用预录制输入运行编译的节点可执行文件，并将输出与预期基线进行比较。无需 coordinator 或 daemon。

```bash
cargo test --test example-tests
```
