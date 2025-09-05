---
layout:     post
title:      全面解析 Ethscriptions：以太坊版“铭文”协议 3 天铸造 3 万枚的背后故事
date:       2025-09-05
header-img: img/post-bg-desk.jpg
catalog: true
---

> 比特币 Ordinals 在 6 个月内突破 1200 万枚；如今 **Ethscriptions** 横空出世，仅上线数日便完成超 3 万 次 **铸造**。它究竟是短暂炒作，还是足以颠覆 NFT 的底层革命？

---

## 1. Ethscriptions 诞生背景：从 Ordinals 到以太坊铭文

2023 年初，比特币 **铭文** 凶猛出圈，Ordinals 协议一举刷新加密社区对“链上 NFT”的想象。6 月 17 日，Genius.com 联合创始人 **Tom Lehman** 把**类似思路搬到以太坊**，正式发布 **Ethscriptions** 协议。协议名字看似生僻，概念却异常简洁：让任何不超过 96 KB 的文件像贴纸一样被**一次性写入区块**、**永久可读**，并且成本比传统 NFT 合约低得多。

短短 72 小时，“ethscription”关键词冲上 Twitter 热搜，Ethereum Punks 最新系列 1 万枚瞬间被“秒光”。OpenSea 上的 **二级市场成交额**已突破 **200 ETH**，社区热情可见一斑。

👉 [想知道老手如何 1 小时完成 100 次 Ethscriptions 低 Gas 铸造？这里告诉你实用技巧。](https://okxdog.com/)

---

## 2. 工作原理：96 KB 链上“刻录”低成本玩法

### 2.1 交易输入即文件

- **触发条件**：任何以太坊交易的 **输入数据（Calldata）**，若被 UTF-8 解码后为有效 **数据 URI**（如 `data:image/png;base64,…`），**且全网从未出现过相同内容**，即刻被记录为一条 *ethscription*。
- **唯一性**：同区块内出现重复 content 将被丢弃；因此理论上不会出现“双花”铭文。
- **16 进制 or Base64**：开发者可直接把前端生成的 Base64 图像放入 Calldata，省去中心化存储，**零 IPFS、零 AWS**。

### 2.2 运行架构概览

1. 用户构造交易 → 2. 节点解码 Calldata → 3. Ethscriptions Indexer 索引并生成 JSON 元数据 → 4. Explorer（ethscriptions.com）即时展示 → 5. 二级市场挂卖单

通过这种“极简”设计，Ethscriptions 用 1/10 的平均 Gas 让传统 ERC-721、ERC-1155 代币望尘莫及。

---

## 3. 关键场景与生态项目盘点

- **Ethereum Punks**：10,000 枚像素头像，对标 CryptoPunks，上线 4 小时全数铸造完成，地板价最高冲到 0.15 ETH。
- **Ethscriptions Name Service**：ENS 平替，已注册 5,000 + 域名，解析记录在 Calldata 内，免年费。
- **实验性铭文音乐**：Deafbeef 与老 OG Eulerbeats 的灵感再次复刻，直接把 30 秒 8-bit 音频刻进区块。

**去中心化游戏资产**、**链上博客**、**无服务器小程序**……想象空间一圈比一圈大。👉 [抢先测试下一批 Ethscriptions 热门实验，抢占早期红利！](https://okxdog.com/)

---

## 4. 技术优势 VS 隐患：社区两极交锋

### 4.1 优点

- **低成本**：简单转账级别的 Gas，适合小图片、小音频。
- **极高安全性**：与以太坊 PoS 同根同源，理论上与主网永存。
- **原生可组合**：尽管目前只能在网站查看，但未来任何智能合约都能以 **Calldata** 验证并配合 ERC-721 桥接。

### 4.2 缺点

- **96 KB 限制**：高清图、长视频绕道。
- **可修剪性**：若未来 **EIP-4444** 落地，节点可裁剪历史数据，远期可读性存疑。
- **缺乏运行时可编程性**：Ethscriptions 仅存储静态数据，无法像链上 SVG NFT 一样随区块高度动态变化。

Chainleft 在社区长帖中指出：“这不是新技术，只是重新包装 2016 年的 Calldata 艺术。”但 Adam McBride 反击：“**市场共识就是新故事**，有了公平首发与投机，故事才会扩大。”

---

## 5. Ethscriptions 与 Ordinals：谁更去中心化？

| 维度                | Ethscriptions      | Ordinals        |
|---------------------|--------------------|-----------------|
| 上链容量            | ≤ 96 KB            | ≤ 4 MB（隔离见证） |
| 成本                | 0.0003 ETH/笔      | 20 sat/vB * 4 MB 约 0.01 BTC |
| 文件类型            | 任何 MIME          | 任何 MIME       |
| 二次校验            | 索引器统一校验     | 专注 “聪” 溯源    |
| 潜在裁剪风险        | 受 EIP-4444 影响   | Prune 几乎无      |

简言之，**比特币铭文偏向抗审查**，**Ethscriptions 则倚重以太坊生态可组合性**。

---

## 6. 如何铸造你的第一条 Ethscriptions NFT？

1. 将文件转 Base64 字符串（图片建议 ≤ 85 KB）。
2. 构造 `data:image/png;base64,你的编码` URI，长度以 UTF-8 单双字节精确计算 < 96 KB。
3. 使用工具如 ethscriptions.com UI、ethers.js、Foundry 脚本，把 URI 放 Calldata。
4. 发送 *0 转账*给任意地址，确保 gas price ≥ 15 gwei（防卡住）。
5. 数分钟后，网站检索到交易，你的 ETH 地址就拥有独一无二的 **ethscription**。

**小提示**：低 Gas 时段（UTC 02:00-05:00）铸造，费用能比高峰期省一半。

---

## 7. 常见问题 FAQ

**Q1：Ethscriptions 是真正的 NFT 吗？**  
A：目前官方并未发行代币，链上只是 Calldata；从经济博弈角度看，只要能流通、被二级市场认可，它就具备 NFT 的功能属性。

**Q2：96 KB 会不会很快不够？**  
A：社区已提案 **链外压缩 + 哈希存证**：将大图压缩后放置 IPFS，仅在 Calldata 记录压缩前哈希，确保可追溯又不超限。

**Q3：跑节点没历史数据就找不到 ethscription？**  
A：索引器已经快照了全部 Calldata，并开放 JSON API，普通用户无需跑节点即可查询。

**Q4：如何防范重复内容？**  
A：Ethscriptions Indexer 会对所有代码进行双 SHA-256 比对；只要内容相同，索引器只认第一笔交易，其余丢弃。

**Q5：何时引入 ERC-20 代币标准？**  
A：Tom Lehman 透露，当前正与多签组织联名讨论 ERC-20 Indexing 接口，预计第三季度开放测试网。

---

## 8. 结语：回到链上原生创造力的实验周期

从 2016 年早期“把艺术塞进 Calldata”，到如今 **Ethscriptions、Ordinals** 相继出圈，一个明显趋势浮现：开发者欢迎低门槛、低成本的方式，把 **任何数据搬到链上**。**社区共识**而非技术难度，正在成为决定项目走多远的核心动力。下一轮牛市，可能就看这一波“极简铭文”是否能继续书写新故事。

时间，会给出答案。