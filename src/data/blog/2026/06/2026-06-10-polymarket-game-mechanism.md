---
title: "Polymarket 背后的游戏机制"
description: "用简单方式解释二元预测市场如何运作、为什么每份赢家份额兑换 1 美元，以及 Polymarket 和赢家通吃资金池有什么不同。"
pubDatetime: 2026-06-10T12:53:33Z
tags: [polymarket, prediction-markets, trading, market-mechanism]
---

## 核心想法

理解 Polymarket 最简单的方法，是把它看成一个交易“事件结果”的市场。

一个市场会提出类似这样的问题：

```text
Will event X happen by date Y?
```

在最简单的二元市场里，只有两个结果：

```text
Yes / A: the event happens
No / B: the event does not happen
```

人们买的不是传统股票，而是某个结果的份额。如果你买的结果是正确的，每一份赢家份额可以兑换 1 美元。如果结果错了，那份份额就变成 0 美元。

这条规则解释了这个游戏的大部分机制：

```text
Winning share  -> $1
Losing share   -> $0
```

所以，如果你用 0.40 美元买入一份 Yes，最后 Yes 赢了，你会收到 1 美元。你的利润是 0.60 美元。如果 Yes 输了，你就损失买入时支付的 0.40 美元。

## 最简单的资金池模型

在看 Polymarket 本身之前，可以先想象一个更简单的游戏。

市场里有 A 和 B 两边：

```text
Money on A = $10
Money on B = $30
Total pool = $40
```

如果 A 赢，选择 A 的人瓜分整个 40 美元资金池。如果 B 赢，选择 B 的人瓜分整个 40 美元资金池。

在这个简化模型里，A 占全部资金的 25%：

```text
A share of pool = 10 / (10 + 30) = 0.25
```

所以 A 的粗略隐含价格是 0.25 美元，B 的粗略隐含价格是 0.75 美元。

如果你在这个简化资金池里向 A 投入 1 美元，那么 A 赢时你的预期返还是：

```text
$1 / $0.25 = 4 shares
4 winning shares x $1 = $4 returned
```

也就是说，如果 A 赢，你的 1 美元会变成 4 美元，总利润是 3 美元。

这是一个很有用的直觉：当某个结果价格是 0.25 美元时，如果它赢了，就相当于 4 倍总返还。

## 基本公式

对二元结果份额来说，基础返还倍数是：

```text
gross payout multiple = 1 / share price
```

例子：

| 份额价格 | 1 美元可以买到的份额 | 判断正确时返还 | 判断正确时利润 |
|---:|---:|---:|---:|
| $0.20 | 5.00 | $5.00 | $4.00 |
| $0.25 | 4.00 | $4.00 | $3.00 |
| $0.40 | 2.50 | $2.50 | $1.50 |
| $0.50 | 2.00 | $2.00 | $1.00 |
| $0.80 | 1.25 | $1.25 | $0.25 |

份额越便宜，如果它赢了，返还倍数就越高。但它便宜也有原因：市场当前认为这个结果发生的可能性更低。

## 为什么价格看起来像概率

因为每份赢家份额会支付 1 美元，所以份额价格可以被理解成一种隐含概率。

如果 Yes 的价格是 0.65 美元，市场大致在表达：

```text
Yes has about a 65% chance
No has about a 35% chance
```

这不代表市场一定是对的。它只表示交易者目前愿意在这个价格附近买卖。

好的交易者不只是问：“这件事会发生吗？”更重要的问题是：

```text
Is the real probability higher or lower than the current market price?
```

如果你认为某件事有 60% 的概率发生，而 Yes 份额只卖 0.40 美元，你可能会认为 Yes 被低估了。

如果你认为某件事只有 30% 的概率发生，而 Yes 份额卖 0.60 美元，你可能会认为 Yes 被高估了。

## 关键区别：Polymarket 不只是一个资金池

简单资金池模型会说：

```text
price of A = money on A / total money
```

这是一个有帮助的教学模型，但它并不是 Polymarket 定价的真实方式。

Polymarket 使用订单簿。价格来自买方和卖方互相挂单、成交的过程。如果有人愿意以 0.38 美元买 Yes，另一个人愿意以 0.40 美元卖 Yes，那么可见市场价格就会围绕这些订单形成。

所以，当前价格不是根据历史上投向 A 和 B 的总资金机械计算出来的。它来自当前最好的买价和卖价：

```text
Bid = highest price someone is willing to pay
Ask = lowest price someone is willing to sell for
Spread = ask - bid
```

例子：

```text
Best bid for Yes = $0.34
Best ask for Yes = $0.40
```

如果你立刻买入，通常要支付卖价，也就是 0.40 美元。

如果你立刻卖出，通常只能拿到买价，也就是 0.34 美元。

页面显示的概率可能看起来像中间价，大约 0.37 美元，但你的真实成交价格取决于订单簿。

## Yes 和 No 份额如何获得抵押支持

在底层机制里，一组完整的 Yes/No 份额由 1 美元抵押品支持。

概念上可以这样理解：

```text
$1 collateral -> 1 Yes share + 1 No share
```

在正常的二元市场中，最终应该只有一边获胜。

如果 Yes 赢：

```text
Yes share -> $1
No share  -> $0
```

如果 No 赢：

```text
Yes share -> $0
No share  -> $1
```

这就是为什么在忽略买卖价差、费用和市场摩擦的情况下，Yes 和 No 的价格通常会加起来接近 1 美元。

如果 Yes 大约是 0.70 美元，No 通常应该大约是 0.30 美元。如果两边价格偏离这个关系太远，交易者可能会通过套利把差距拉回来。

## 一个具体例子

假设某个市场的问题是：

```text
Will A happen?
```

当前订单簿允许你以 0.25 美元买入 A。

你花 10 美元：

```text
$10 / $0.25 = 40 A shares
```

如果 A 赢：

```text
40 shares x $1 = $40 returned
Profit = $40 - $10 = $30
```

如果 A 输：

```text
40 shares x $0 = $0 returned
Loss = $10
```

所以风险是不对称的：

```text
Maximum loss = the amount you paid
Maximum payout = number of shares x $1
```

在单纯做多 Yes 或 No 的情况下，你最多亏掉买入成本，不会亏超过你支付的金额。但这也意味着你可能损失全部本金。

## 交易者真正玩的是哪一层

这个游戏不只是“猜结果”。更准确地说，它是在“寻找错误价格”。

假设 Yes 的交易价格是 0.25 美元。

如果你真实估计 Yes 的概率就是 25%，那这个价格基本公平。在很多类似交易中，扣除成本和买卖价差之前，你没有明显优势。

如果你真实估计 Yes 的概率是 40%，那么 0.25 美元可能很有吸引力。

如果你真实估计 Yes 的概率只有 10%，那么 0.25 美元可能就太贵了。

优势来自你的概率估计和市场价格之间的差距：

```text
edge = your estimated probability - market price
```

对 Yes 份额来说：

```text
positive edge  -> your probability is higher than price
negative edge  -> your probability is lower than price
```

当然，真正困难的地方在于：你是否真的比市场估得更准。

## 为什么可以在结算前退出

你不一定要等到事件最终结算。

如果你以 0.40 美元买入 Yes，之后市场价格涨到 0.70 美元，你可以在最终结果出来前卖出：

```text
Buy at $0.40
Sell at $0.70
Profit = $0.30 per share
```

这让 Polymarket 同时像预测游戏，也像交易游戏。交易者可以从事件结束前的价格变化中获利，即使他们从来没有持有到最终结算。

但能不能退出取决于流动性。页面显示的价格不保证你能以那个价格卖出大仓位。你需要另一边有买家。

## 心智模型

可以这样记住 Polymarket：

```text
1. Each market has outcome shares.
2. Each winning share pays $1.
3. Each losing share pays $0.
4. The share price behaves like probability.
5. Your payout multiple is 1 / price.
6. Real prices come from an order book, not a simple pool formula.
7. Profit comes from buying probabilities that are too cheap or selling probabilities that are too expensive.
```

资金池模型仍然有用。如果 A 有 10 美元，B 有 30 美元，很自然会觉得 A 的价格大约是 25%。但在 Polymarket 里，真正的交易发生在另一个交易者此刻愿意接受的价格上。

这就是这个机制的核心：预测变成可交易资产，概率变成价格，而最终真相会把赢家代币变成 1 美元。

## 来源

- [Polymarket 101](https://docs.polymarket.com/polymarket-101)
- [Polymarket Prices & Orderbook](https://docs.polymarket.com/concepts/prices-orderbook)
- [Polymarket Positions & Tokens](https://docs.polymarket.com/concepts/positions-tokens)
- [Polymarket Resolution](https://docs.polymarket.com/concepts/resolution)
