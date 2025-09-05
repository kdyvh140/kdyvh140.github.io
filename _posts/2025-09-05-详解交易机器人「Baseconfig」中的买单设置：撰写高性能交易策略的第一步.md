---
layout:     post
title:      详解交易机器人「Baseconfig」中的买单设置：撰写高性能交易策略的第一步
date:       2025-09-05
header-img: img/post-bg-desk.jpg
catalog: true
---

无论你是刚入门还是已深耕量化交易多年，合理配置 **买入设置** 都是稳赚中立、减少回撤的关键环节。本文将系统拆解 **Baseconfig 中所有 Buy settings** 的作用逻辑，并配合实际案例、常见问题 FAQ，帮助你在 **加密货币量化交易** 这条路上少走弯路。

## 买单触发方式：四大渠道并行不悖

在平台上，机器人可以为 **现货交易** 建立买单的来源有四种：

1. 手动下单：在仪表盘点击“立即买入”。  
2. 策略引擎：由你在策略构建器中设置的 **技术指标** 触发。  
3. TradingView 报警：利用第三方指标向机器人推送信号。  
4. 信号订阅：通过 Marketplace 购买的付费或免费 **交易信号**。

> 关键见解：上述四种买入方式 **可以同时开启**。你可以让策略在行情震荡时做小止损的 **现货套利**，同时在 Signals 捕获到特别强势币时一键加仓；配合 DCA 或 TSB（下文详解），灵活度极高。

## 进阶买入功能：三大神器组合使用

| 功能名 | 适用币种 | 核心思路 | 风险提示 |
|---|---|---|---|
| Trailing Stop-Buy（TSB） | 新建仓位 | 币价先向下“踩点”，当开始反弹时追买，摊薄成本 | 只能应用于策略或手动买入，**不支持 Signals** |
| Shorting 空买回 | 已有仓位的卖出后动作 | 卖出后预留 Quote 货币，择机更低价买回，实现“低买高卖”二次入场 | 与 DCA **互斥** |
| Dollar-Cost Averaging（DCA） | 已浮亏仓位 | 追加买入来 **摊低风险阈值**，降低回本所需涨幅 | 增加总仓位，行情继续跌会放大亏损 |

提醒大家：DCA 与 Shorting **不能同时启用**，需在低风险预案里依据资金韧性二选一。

---

## 基础买回设置全览

### 订单类型（Order type）

- **市价单**（Market）：即刻成交，滑点风险大，适合急抢热点。  
- **限价单**（Limit）：挂指定价，不易成交，但可避免高波动冲击。

### 最大挂单时长（Max open time buy）

防止资金长期被锁，默认 5~10 分钟即可。

### 高/低比挂单（Percentage higher/lower bid）

- 想杀价可把 bid 设在现价 -0.5% ～ -1.5%，但不建议设 **过于激进**（>15%未成交风险高）。  
- 在放量上涨时，也可挂 +0.3% 抢单人成交。

### 仓位管理与冷却

| 标签 | 示例取值 | 目的 |
|---|---|---|
| Max open positions | 20 | 最多同时持有 20 个币 |
| Max % open positions per currency | 4% | 单币最多 4 个仓位 |
| Only 1 open buy order per currency | On | 同币种最多一张买单 |
| Enable cooldown | Off/On | 成交后 **冷却期** 避免连买5分钟 |
| Cooldown when |  Buy / Sell / Both | 依据需求选择 |
| Auto merge positions | Off | 多仓合并为单仓，方便统计盈利 |

---

### 入门必读实例

假设你总仓 10,000 USDT，按 **百分比法** 设定：

- Percentage buy amount：单仓固定购买总仓的 **2%（200 USDT）**
- Minimum amount per order：20 USDT（防止低于交易所最小下单）
- Maximum amount allocated：8,000 USDT（留足 2,000 USDT 机动）

这样即可在行情暴跌时，既有子弹 DCA，又不至梭哈。

---

## 币种 & 资金分配

### Quote currency 与 Allowed currencies

- Quote：首推 USDT/BUSD 等 **稳定资产**，波动可控易于核算收益。  
- Allowed：DIY 选币时，可以选取 **BTC、ETH、BNB、SOL** 等主流高流通标，降低深度风险。

### 常见问题速答专区

**Q1：为什么我设置好参数，机器人仍不买？**  
- 先检查是否 **Cooldown** 中  
- 检查 Quote 余额、最大仓位限制  
- Only buy when positive pairs 开启后，看近 24h 涨幅是否满足阈值  

**Q2：加仓后不想合并仓位怎么办？**  
关闭 Auto merge positions；保持“单币种多仓位”即可独立止盈止损。  

**Q3：百分比买入与固定金额冲突时市场如何执行？**  
系统自动取“较大值”原则：若 Percentage 算得 50 USDT，但 Minimum per order 设定 100 USDT，最终按 100 USDT 下单。

---

## 策略选取与高级开关

- **Strategy 选项卡**：可直接复用 **策略商店** 的现成模型，或用 Strategy Builder 自研 MA+MACD 指标魔方。  
- **Number of targets to buy**：一次 **TOP N** 入场，例如“发现5币信号，只买前3个”，仓位管理更聚焦。  
- **Signals Only**：关闭所有策略，纯跟信号交易，适合“订阅大神号”用户。

> **体验高阶玩法**  
👉 [点此了解无代码也能打造的智能多策略组合](https://okxdog.com/)

---

## 实战 Tips：三步走完一套高性能 Buy settings

1. **初选资产池**：在 Allowed currencies 里只留 8-12 个 **高流动性现货**，避免冷门币被插针。  
2. **拿出 70% 资金** 做核心仓位，保留 30% 用于 DCA 追弱水码。  
3. **监控** Only buy when positive pairs（2h）+ Cooldown（10 min）组合，能让机器人在高位拐顶时 **及时刹车**，降低全仓回撤。

---

### Trailing Stop-Buy：上下震荡里的“抄底”利器

- 启用后，**触发百分比** 通常设 0.8%–2%，根据币种波动性调整。  
- 适合趋势线刚突破但不想追高，等回测确认。

---

## 结语：用细节雕刻利润曲线

一套优秀的 **买入设置** 既是风险阀也是加速器：合理比例的 DCA、严格的 Cooldown、策略与信号的交错入场，都可能决定你是持续盈利还是 **高买低卖**。现在回到交易面板，把今天学到的参数重新排列组合，下一次 **K 线翻绿** 时，你的机器人已经做好准备。

👉 [立刻体验一站式买入参数模板，让机器人把握下一个黄金买点](https://okxdog.com/)