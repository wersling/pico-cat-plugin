# PicoCat Claude Code Plugin

与 PicoCat 桌面宠物游戏互动的 Claude Code 插件。

## 功能

- **MCP 工具**：查询气球状态、戳气球获取奖励
- **Hooks**：将 Claude Code 会话事件映射为猫咪动画

## 前置条件

- PicoCat 游戏正在运行，且设置中开启了 Bridge 自动启动

## 安装

```bash
# 从 GitHub marketplace 安装
/plugin marketplace add wersling/pico-cat-plugin
/plugin install pico-cat
```

## 工具

### balloon_status
查询当前游戏中的气球状态。

### balloon_pop
戳破一个漂浮的气球，等待礼物掉落并自动开箱，返回奖励结果。

## Skill

使用 `/cat:balloon` 查看和戳气球。支持 `/loop` 定时轮询：

```
/loop 60s /cat:balloon
```

## 原理

插件通过 MCP Streamable HTTP 协议连接到 PicoCat 游戏的 Bridge 服务器（`http://127.0.0.1:23336/mcp`）。
