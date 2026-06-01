---
name: pico-cat:bowl
description: 管理猫岛中的猫盆（食盆/水盆/自动喂食器/自动饮水机）。查看猫盆状态、购买消耗品（水、猫粮），并填充空猫盆。当猫盆填充量低于 20% 时自动补充。适合用 /loop 定时轮询。
---

# 猫盆管理

使用 pico_cat MCP 服务器的工具管理猫盆（食盆/水盆）。

## 猫盆类型

### 普通猫盆
- `contentType`: `water` / `cat_food` / `null`
- `fillLevel`: 0-100，每次填充恢复至 100
- 填充阈值：< 20

### 自动喂食器 / 自动饮水机
- `contentType`: `auto_food_feeder` / `auto_water_feeder`
- `fillLevel`: 0-500，每次填充增加 100
- 填充阈值：< 100（即 20%）
- 从空到满最多需要 5 次填充
- 消耗品匹配：`auto_food_feeder` 使用 `cat_food`，`auto_water_feeder` 使用 `water`

## 工具说明

### bowl_status
查询场景中所有猫盆的状态。

返回数组，每个猫盆包含：
- `id`: 猫盆唯一标识
- `contentType`: 内容物类型（water/cat_food/auto_food_feeder/auto_water_feeder/null）
- `fillLevel`: 普通猫盆 0-100，自动喂食器/饮水机 0-500
- `x`, `y`: 场景位置

`contentType` 为 `null` 表示空猫盆，可以倒入任意类型的消耗品。

### consumable_list
查询所有可购买的消耗品及价格。

返回数组，每个消耗品包含：
- `id`: 消耗品 ID（water/cat_food）
- `name`: 名称
- `price`: 单价
- `feederContentType`: 倒入猫盆后的内容物类型
- `fillAmount`: 每次填充量

### coin_status
查询当前金币数量。

返回：
- `coins`: 当前金币数

### buy_consumable
购买消耗品到背包。

参数：
- `itemId`（必填）：消耗品 ID
- `count`（可选，默认 1）：购买数量

返回：
- `success`: 是否成功
- `message`: 结果描述
- `cost`: 总花费
- `remainingCoins`: 剩余金币

购买前会自动检查余额，不足时会返回错误。

### fill_bowl
给猫盆添加食物或水。

参数：
- `bowlId`（必填）：猫盆 ID

规则：
- 仅当猫盆填充量低于 20% 时才能填充
- 自动从背包中消耗 1 个匹配的消耗品
- 空猫盆（contentType=null）可接受任意类型消耗品
- 已装过食物的猫盆只能使用相同类型的消耗品
- 填充后填充量恢复为 100%

返回：
- `success`: 是否成功
- `message`: 结果描述
- `bowlId`: 猫盆 ID
- `contentType`: 填充后的内容物类型
- `fillLevel`: 填充后的填充量

## 使用流程

### 手动维护

1. 调用 `bowl_status` 查看所有猫盆状态
2. 找到 `fillLevel < 20` 的猫盆
3. 调用 `fill_bowl` 尝试填充
4. 如果返回"背包中没有匹配的消耗品"错误：
   a. 调用 `consumable_list` 查找匹配的消耗品
   b. 调用 `coin_status` 确认金币是否足够
   c. 调用 `buy_consumable` 购买需要的消耗品
   d. 重新调用 `fill_bowl`

### 匹配规则

消耗品与猫盆的匹配关系：

| 消耗品 | feederContentType | 可填充的猫盆 |
|--------|-------------------|-------------|
| 水 (water) | water | contentType=water / auto_water_feeder / null |
| 猫粮 (cat_food) | cat_food | contentType=cat_food / auto_food_feeder / null |

### 自动喂食器/饮水机的填充策略

自动喂食器和自动饮水机容量为 500，每次填充增加 100：
1. 检查 `fillLevel < 100` 时触发填充
2. 计算需要的填充次数：`Math.ceil((500 - fillLevel) / 100)`
3. 逐次调用 `fill_bowl`，每次消耗 1 个对应消耗品
4. 如果中途消耗品不足，按购买流程补充后继续

## 循环模式

使用 `/loop` 定时检查并维护猫盆：

```
/loop 10m /cat:bowl
```

每 10 分钟检查一次，自动购买消耗品并填充空猫盆。


## 异常情况指引
 - 如遇到无法连接MCP，引导用户回游戏-设置-AI工具中查看 Bridge 服务是否启动，如果还是不行，建议重启游戏再试。
