> **[English](../../advanced/ros2-bridge.md)**

# ROS2 桥接

Adora 提供基于 YAML 声明式的 ROS2 桥接，让任何 Adora 节点无需导入 ROS2 库即可与 ROS2 topics、services 和 actions 通信。您只需在数据流 YAML 中使用 `ros2:` 键定义桥接，框架会自动生成一个桥接二进制文件，在 Apache Arrow（Adora 的原生格式）和 ROS2 CDR/DDS 之间转换。

## 功能概览

| 功能 | 配置 | 描述 |
|------|------|------|
| Topic 订阅 | `topic` + `direction: subscribe` | 从 ROS2 接收，转发为 Arrow |
| Topic 发布 | `topic` + `direction: publish` | 接收 Arrow，发布到 ROS2 |
| 多 Topic | `topics` | 单个 ROS2 节点上的多个 topic |
| Service 客户端 | `service` + `role: client` | 发送请求，接收响应 |
| Service 服务端 | `service` + `role: server` | 接收请求，发送响应 |
| Action 客户端 | `action` + `role: client` | 发送目标，接收反馈 + 结果 |
| Action 服务端 | `action` + `role: server` | 接收目标，发送反馈 + 结果 |
| QoS 策略 | `qos` | 可靠性、持久性、历史、活跃性 |

## 架构

```
用户节点 <--(Arrow/SharedMem)--> 桥接二进制 <--(CDR/DDS)--> ROS2
```

## 前提条件

- **ROS2 环境已 source**：`AMENT_PREFIX_PATH` 必须设置
- **消息包已安装**：例如 `turtlesim`、`geometry_msgs`、`example_interfaces`

## Topic 桥接

### 单 Topic（订阅）

```yaml
nodes:
  - id: pose_bridge
    ros2:
      topic: /turtle1/pose
      message_type: turtlesim/Pose
      direction: subscribe
    outputs:
      - pose
```

### 单 Topic（发布）

```yaml
nodes:
  - id: cmd_bridge
    ros2:
      topic: /turtle1/cmd_vel
      message_type: geometry_msgs/Twist
      direction: publish
    inputs:
      cmd_vel: planner/cmd_vel
```

## Service 桥接

```yaml
- id: add_service
  ros2:
    service: /add_two_ints
    service_type: example_interfaces/AddTwoInts
    role: server
  inputs:
    request: client_node/request
  outputs:
    - response
```

## Action 桥接

```yaml
- id: nav_action
  ros2:
    action: /navigate
    action_type: nav2_msgs/NavigateToPose
    role: client
  inputs:
    goal: planner/goal
  outputs:
    - feedback
    - result
```

## QoS 配置

| QoS 字段 | 类型 | 默认值 | 描述 |
|-----------|------|--------|------|
| `reliable` | bool | `false` | 可靠 vs 尽力传输 |
| `durability` | string | `volatile` | `volatile` 或 `transient_local` |
| `keep_last` | integer | `1` | 历史深度 |
| `keep_all` | bool | `false` | 使用 KeepAll 历史 |

---

详细内容请参见[英文完整版本](../../advanced/ros2-bridge.md)。
