---
name: pico-cat:wooden-fish
description: AI 代你敲木鱼积功德。检测场景中的木鱼后连续敲击 N 下，每敲一下消耗一次 AI token、功德 +1。适合配合 /loop 周期性敲一轮。
---

# 敲木鱼

使用 pico_cat MCP 服务器的工具敲木鱼积攒功德。每调用一次 `woodfish_knock` 即敲一下、消耗一次 AI token、功德 +1。

## 工具说明

### woodfish_status
查询场景中是否存在木鱼及当前功德。

返回：
- `hasWoodfish`: 场景中是否有木鱼
- `count`: 场景中木鱼数量
- `woodfish`: 木鱼数组，每个包含：
  - `id`: 木鱼唯一标识
  - `x`, `y`: 场景位置
- `meritCount`: 当前累计功德数

`hasWoodfish` 为 `false` 说明场景里没有木鱼，需要先去商店「用品」页购买并放置木鱼（物品名「功德+1」，价格 80 金币）。

### woodfish_knock
敲一下场景中的木鱼，功德 +1。

参数：
- `woodfish_id`（可选）：指定木鱼 ID，不填则敲场景中第一个木鱼

返回：
- `success`: 是否成功
- `message`: 结果描述
- `meritCount`: 敲击后的最新功德数

如果场景中没有木鱼会返回错误。

## 使用流程

1. 调用 `woodfish_status` 检测场景中是否有木鱼、查看当前功德
2. 如果 `hasWoodfish` 为 `true`，连续调用 `woodfish_knock` 共 N 次：
   - 每次调用即敲一下，功德 +1
   - 保持连续敲击，每次调用之间自然间隔约 2 秒（每次都是一次 MCP 往返）
   - 敲到目标次数 N 后停止
3. 敲完后再调一次 `woodfish_status` 汇报累计功德

如果场景没有木鱼，提醒用户去商店购买并放置木鱼后再试。

## 敲击次数 N

- 用户调用时可指定次数，如 `/pico-cat:wooden-fish 9` 敲 9 下
- 未指定时默认敲 **9** 下
- 可传 `108`（功德圆满），但会持续消耗较多 token

每敲一下都会消耗一次 AI token，次数越多 token 消耗越大。

## 循环模式

配合 `/loop` 周期性敲一小轮：

```
/loop 5m /pico-cat:wooden-fish 9
```

每 5 分钟让 AI 敲 9 下，持续积攒功德。

## 异常情况指引
 - 如遇到无法连接MCP，引导用户回游戏-设置-AI工具中查看 Bridge 服务是否启动，如果还是不行，建议重启游戏再试。
