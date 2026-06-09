---
title: "Polymarket 新手入门：从 USDC 到 pUSD 的完整流程"
description: "用简体中文解释 Polymarket 的基本玩法、Polygon 入金、pUSD、下单、结算和提现流程。"
pubDatetime: 2026-06-09T08:53:48Z
tags: [polymarket, prediction-markets, polygon, usdc]
---

## 先说结论

Polymarket 是一个预测市场。你不是在买传统股票，而是在交易某个事件的结果，例如“某件事会不会在指定日期前发生”。市场通常分成 Yes 和 No 两边，每一边的价格在 0 到 1 美元之间。

如果你买 Yes，最后事件真的发生，这个 Yes 份额通常会结算为 1 美元；如果事件没有发生，就结算为 0。No 份额也是同理，只是方向相反。

从资金流程看，可以简单理解成：

```text
USDC / USDC.e on Polygon
  -> 存入 Polymarket
  -> 转成 pUSD
  -> 用 pUSD 买 Yes / No
  -> 赢了得到更多 pUSD，输了损失 pUSD
  -> 退出时把 pUSD 提现回 USDC 或其他支持的资产
```

截至 2026-06-09，Polymarket 国际站的加密资产流程主要围绕 Polygon、USDC/USDC.e 和 pUSD 展开。不同国家和地区的可用性、入金方式、最小金额和支持资产可能变化，真正操作前一定要以 Polymarket 页面显示为准。

这不是投资建议，也不是法律意见。预测市场有亏损风险，而且不同地区的监管状态不同。

## Polymarket 到底在交易什么

Polymarket 上每个市场都会问一个明确的问题。常见形式是：

```text
Will X happen by Y date?
```

也就是：“X 这件事会在 Y 日期前发生吗？”

你可以买：

| 方向 | 含义 | 赢了通常得到 |
|---|---|---|
| Yes | 你认为事件会发生 | 每份 1 美元 |
| No | 你认为事件不会发生 | 每份 1 美元 |

价格可以粗略看成市场共识概率。比如 Yes 价格是 0.63 美元，代表市场大概认为这件事有 63% 的概率发生。但这只是交易价格，不是真理，也不保证最后结果。

## 技术上它是怎么工作的

Polymarket 不是一个纯中心化投注网站。它的结构更像一个混合系统：

| 模块 | 简化理解 |
|---|---|
| Polygon | 主要链上网络 |
| 智能合约 | 管理抵押、份额和结算 |
| pUSD | Polymarket 内部使用的抵押资产 |
| CTF outcome tokens | Yes / No 结果份额，技术上是 ERC-1155 代币 |
| CLOB 订单簿 | 撮合订单的交易系统 |
| Oracle / resolution | 判断市场最终结果的机制 |

关键点是：订单撮合并不是每一步都完全在链上完成。Polymarket 使用中心化限价订单簿，也就是 CLOB，做快速撮合；成交之后再通过 Polygon 上的智能合约结算。

这带来的好处是速度更快、成本更低；代价是你仍然依赖 Polymarket 的前端、订单簿、规则解释、市场结算机制，以及 Polygon 网络本身。

## 准备工作

如果只是想小额测试一次流程，通常需要准备：

1. 一个可以使用 Polymarket 的账户。
2. 一点 USDC 或 USDC.e，最好在 Polygon 网络上。
3. 确认自己所在地区可以使用对应产品。
4. 看清楚 Polymarket 页面里显示的最低入金金额。
5. 只用可以承受损失的小额资金。

如果你人在美国，要特别区分 `polymarket.com` 国际站和 `polymarket.us`。官方文档把 Polymarket US 描述为面向美国用户、使用美元的受监管产品；它和国际站的加密资产流程不是一回事。

## 入金：USDC 怎么变成 pUSD

普通用户不要自己随便找一个地址转账。正确流程应该从 Polymarket 页面开始：

1. 打开 Polymarket。
2. 点击 Deposit。
3. 选择 Transfer Crypto。
4. 选择你要用的 token 和 network，例如 Polygon 上的 USDC 或 USDC.e。
5. 复制页面显示的专属入金地址。
6. 从钱包或交易所把资金发到这个地址。
7. 等待确认后刷新页面，余额会显示在 Polymarket 账户里。

Polymarket 现在使用 pUSD 作为交易抵押资产。所以你存入的 USDC 或 USDC.e 会被转换或包装成 pUSD，之后下单时用的是 pUSD 余额。

不要把资金直接发到所谓的 profile address，也不要凭记忆使用旧地址。每次操作前都应该从 Deposit 页面复制当前显示的地址。

## 最小金额：1 USDC 够不够

不建议用 1 USDC 测试。

Polymarket 文档显示，Polygon 入金有最低金额，当前大约是 2 美元级别。低于最低金额的入金可能不会立刻处理，可能要等累计金额达到门槛。

更现实的问题是，下单也可能有最小订单量。不同市场的订单簿会带有自己的 `min_order_size`。所以即使你成功入金 2 USDC，也可能发现某些市场的最小下单金额比你的余额更高。

如果只是为了完整测试一次流程，我会用这样的金额：

```text
5 USDC：可以测试，但空间比较小
10 USDC：更适合新手跑通入金、下单和提现流程
```

10 USDC 不代表更安全，只是更不容易卡在最低入金、最低订单量和小额余额这些细节上。

## 如何看一个市场

进入一个市场前，先看四件事：

| 项目 | 为什么重要 |
|---|---|
| 问题描述 | 决定最后到底按什么标准结算 |
| 截止时间 | 有些事件会在特定时间点后停止交易或结算 |
| 规则说明 | 细节可能决定 Yes 和 No 的边界 |
| 订单簿深度 | 决定你能不能以接近显示价格成交 |

不要只看一个大大的百分比就下单。显示价格通常只是市场价格的简化表达。真实成交要看买一价、卖一价和订单簿深度。

举个例子：

```text
Yes 最佳买价：0.34
Yes 最佳卖价：0.40
页面显示价格：约 0.37
```

如果你立刻买入，可能要按 0.40 成交；如果你立刻卖出，可能只能按 0.34 成交。中间的差就是 spread。流动性差的市场，spread 可能很大。

## 下单：Market order 和 Limit order

新手最常见的是两种操作：

| 类型 | 含义 | 适合场景 |
|---|---|---|
| Market order | 立刻按当前可成交价格买入或卖出 | 想马上成交 |
| Limit order | 只在你指定价格或更好价格成交 | 想控制价格 |

Polymarket 的底层订单本质上更接近限价订单。所谓 market order，通常可以理解成一个愿意立刻吃掉现有订单簿的限价单。

小额测试时，不要急着点最大金额。更稳的方式是：

1. 选一个流动性比较好的市场。
2. 仔细读规则。
3. 看 Yes 和 No 的买卖价差。
4. 用很小金额买入一边。
5. 到 Portfolio 里确认仓位。
6. 试着挂一个小额卖出单，理解退出流程。

## 赢了、输了、提前卖出分别会怎样

买入之后你持有的是某个结果方向的份额。

| 情况 | 结果 |
|---|---|
| 你买的方向最终获胜 | 每份通常可兑换 1 pUSD |
| 你买的方向最终失败 | 每份通常价值归零 |
| 市场还没结算前卖出 | 按当时订单簿价格成交，可能赚也可能亏 |

比如你用 4 pUSD 买入 10 份 Yes，成交均价是 0.40。如果最后 Yes 赢了，这 10 份会变成约 10 pUSD。如果 Yes 输了，这 10 份变成 0。

如果中途市场价格从 0.40 涨到 0.60，你也可以在结算前卖出，提前锁定收益。但能否成交、成交多少、成交价格如何，都取决于订单簿。

## 提现：pUSD 怎么回到 USDC

当你想退出时，流程通常是：

1. 到 Portfolio 或 Withdraw 页面。
2. 输入接收地址。
3. 选择目标网络和目标 token。
4. 确认金额。
5. 提交提现。
6. 等待资金到达外部钱包。

技术上，pUSD 可以通过 Polymarket 的 Collateral Offramp 解包回 USDC.e，也可以通过提现桥接流程换成支持的目标资产。官方文档还提到，某些提现路径会经过 Uniswap v3 池把资产换成 native USDC；如果池子流动性不足，大额提现可能需要拆分或等待。

普通用户不用手动操作合约，直接用页面里的 Withdraw 流程即可。真正要注意的是：

- 选择正确网络。
- 填写你自己控制的接收地址。
- 不要把 pUSD 直接打到不支持 pUSD 的交易所。
- 大额提现前先小额测试。

## 小额测试流程

如果目标只是学习流程，可以这样做：

```text
1. 准备 10 USDC on Polygon
2. 在 Polymarket Deposit 页面选择 Polygon + USDC
3. 复制页面显示的入金地址
4. 发送 10 USDC
5. 等余额显示为 pUSD
6. 找一个流动性好的市场
7. 用很小金额买入 Yes 或 No
8. 到 Portfolio 检查仓位
9. 尝试卖出一小部分或等待结算
10. 用 Withdraw 测试提回外部钱包
```

第一次测试的目标不是赚钱，而是理解每一步会发生什么：入金多久到账、余额如何显示、下单是否有最小金额、成交价和页面价格是否一致、提现时支持哪些 token 和网络。

## 常见风险

Polymarket 的流程看起来简单，但风险并不少。

| 风险 | 说明 |
|---|---|
| 亏损风险 | 判断错方向时，仓位可能归零 |
| 流动性风险 | 想买或想卖时可能没有好价格 |
| 规则风险 | 市场问题和结算规则的细节会影响结果 |
| Oracle 风险 | 结果争议依赖平台和 oracle/resolution 机制 |
| 链上风险 | Polygon、智能合约、桥和钱包操作都有技术风险 |
| 地址错误 | 转错链或转错地址可能很难找回 |
| 地区限制 | 可访问不等于当地法律允许 |
| pUSD 接收风险 | 外部交易所未必支持直接接收 pUSD |

尤其要避免用 VPN 绕过地理限制。平台条款、风控和当地法律都可能让这件事变得很麻烦。

## 新手检查清单

下单前可以用这份清单快速确认：

- [ ] 我确认自己所在地区可以使用对应 Polymarket 产品。
- [ ] 我知道这是预测市场，可能亏掉本金。
- [ ] 我使用的是 Deposit 页面显示的地址。
- [ ] 我选择了正确的网络，例如 Polygon。
- [ ] 我知道 USDC / USDC.e 入金后会变成 pUSD。
- [ ] 我读过市场问题和结算规则。
- [ ] 我看过订单簿，不只是看页面百分比。
- [ ] 我理解赢了通常每份得 1 美元，输了通常归零。
- [ ] 我知道提现时要选择外部钱包支持的 token 和网络。
- [ ] 我先用小额测试，而不是直接转大额资金。

## 一句话总结

Polymarket 的新手流程可以理解为：把 Polygon 上的 USDC 或 USDC.e 存进去，系统转成 pUSD；用 pUSD 买 Yes 或 No；市场结算后赢的一边变成更多 pUSD，输的一边归零；最后再通过提现流程把 pUSD 换回 USDC 或其他支持资产。

如果只是想学习流程，10 USDC 比 1 USDC 或 2 USDC 更适合测试，因为它能绕开很多最低金额带来的小麻烦。

## 参考资料

- [Polymarket Deposit](https://docs.polymarket.com/trading/bridge/deposit)
- [Polymarket Supported Assets](https://docs.polymarket.com/trading/bridge/supported-assets)
- [Polymarket USD](https://docs.polymarket.com/concepts/pusd)
- [Polymarket Withdraw](https://docs.polymarket.com/trading/bridge/withdraw)
- [Polymarket Prices & Orderbook](https://docs.polymarket.com/concepts/prices-orderbook)
- [Polymarket Contracts](https://docs.polymarket.com/resources/contracts)
