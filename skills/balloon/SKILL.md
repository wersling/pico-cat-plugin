---
name: pico-cat:balloon
description: 与猫岛中的气球互动。查看气球状态、戳气球获得金币或装饰奖励。适合用 /loop 定时轮询。
---

# 气球互动

使用 pico_cat MCP 服务器的工具与气球互动。

## 工具说明

### balloon_status
查询当前游戏中的所有气球状态。

返回数组，每个气球包含：
- `id`: 气球唯一标识
- `state`: 当前状态（entering/floating/dropping/ground/opening/done）
- `x`: 水平位置
- `color`: 气球颜色

只有 `state` 为 `floating` 的气球可以戳破。

### balloon_pop
戳破一个漂浮的气球，自动等待礼物掉落并开箱。

参数：
- `balloon_id`（可选）：指定气球 ID，不填则自动选择第一个漂浮的气球

返回奖励信息：
- `type`: "coins"（金币）或 "item"（物品）
- `amount`: 金币数量（type 为 coins 时）
- `itemName`: 物品名称（type 为 item 时）

## 使用流程

1. 调用 `balloon_status` 查看是否有漂浮的气球
2. 如果有，调用 `balloon_pop` 戳破并获取奖励
3. 奖励自动发放到游戏中

## 奖励概率

- 70% 获得 1-21 金币（1% 概率获得 100 金币）
- 30% 获得随机装饰或植物物品

## 循环模式

使用 `/loop` 定时检查并戳气球：

```
/loop 60s /cat:balloon
```

每 60 秒检查一次，有气球就戳掉。
