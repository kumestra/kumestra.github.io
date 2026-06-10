---
title: "The Game Mechanism Behind Polymarket"
description: "A simple explanation of how binary prediction markets work, why shares pay $1, and how Polymarket differs from a winner-takes-all pool."
pubDatetime: 2026-06-10T12:53:33Z
tags: [polymarket, prediction-markets, trading, market-mechanism]
---

## The Core Idea

Polymarket is easiest to understand as a market for event outcomes.

A market asks a question such as:

```text
Will event X happen by date Y?
```

For a simple binary market, there are only two outcomes:

```text
Yes / A: the event happens
No / B: the event does not happen
```

People do not buy a traditional stock. They buy outcome shares. If the outcome is correct, each winning share can be redeemed for $1. If the outcome is wrong, that share becomes worth $0.

That one rule explains most of the game:

```text
Winning share  -> $1
Losing share   -> $0
```

So if you buy one Yes share for $0.40 and Yes wins, you receive $1. Your profit is $0.60. If Yes loses, you lose the $0.40 you paid.

## The Simplest Pool Model

Before looking at Polymarket itself, imagine a simpler game.

There are two sides, A and B:

```text
Money on A = $10
Money on B = $30
Total pool = $40
```

If A wins, the people who chose A split the whole $40 pool. If B wins, the people who chose B split the whole $40 pool.

In that simplified model, A has 25% of the money:

```text
A share of pool = 10 / (10 + 30) = 0.25
```

So the rough implied price of A is $0.25, and the rough implied price of B is $0.75.

If you put $1 on A in this simplified pool, your expected payout if A wins is:

```text
$1 / $0.25 = 4 shares
4 winning shares x $1 = $4 returned
```

That means your $1 comes back as $4 if A wins, giving you $3 profit.

This is a useful intuition: when an outcome is priced at $0.25, it behaves like a 4x gross payout if it wins.

## The Formula

For a binary outcome share, the basic payout multiple is:

```text
gross payout multiple = 1 / share price
```

Examples:

| Share price | Shares bought with $1 | Return if correct | Profit if correct |
|---:|---:|---:|---:|
| $0.20 | 5.00 | $5.00 | $4.00 |
| $0.25 | 4.00 | $4.00 | $3.00 |
| $0.40 | 2.50 | $2.50 | $1.50 |
| $0.50 | 2.00 | $2.00 | $1.00 |
| $0.80 | 1.25 | $1.25 | $0.25 |

The cheaper the share, the bigger the payout if it wins. But it is cheap for a reason: the market currently thinks that outcome is less likely.

## Why Price Looks Like Probability

Because a winning share pays $1, a share price can be read as an implied probability.

If Yes costs $0.65, the market is roughly saying:

```text
Yes has about a 65% chance
No has about a 35% chance
```

This does not mean the market is correct. It only means traders are currently willing to buy and sell around that price.

A good trader is not asking only, "Will this happen?" A good trader is asking:

```text
Is the real probability higher or lower than the current market price?
```

If you think an event has a 60% chance, and the Yes share costs $0.40, you may think Yes is underpriced.

If you think an event has a 30% chance, and the Yes share costs $0.60, you may think Yes is overpriced.

## The Important Difference: Polymarket Is Not Just a Pool

The simple pool model says:

```text
price of A = money on A / total money
```

That is a helpful teaching model, but it is not exactly how Polymarket sets prices.

Polymarket uses an order book. Prices are formed by buyers and sellers placing orders against each other. If someone wants to buy Yes at $0.38 and someone else wants to sell Yes at $0.40, the visible market sits around those orders.

So the current price is not mechanically calculated from the total money ever placed on A and B. It comes from the best available bids and asks:

```text
Bid = highest price someone is willing to pay
Ask = lowest price someone is willing to sell for
Spread = ask - bid
```

Example:

```text
Best bid for Yes = $0.34
Best ask for Yes = $0.40
```

If you buy immediately, you usually pay the ask: $0.40.

If you sell immediately, you usually receive the bid: $0.34.

The displayed probability might look like the middle, around $0.37, but your real execution price depends on the order book.

## How Yes and No Shares Are Backed

Under the hood, a complete Yes/No pair is backed by $1 of collateral.

Conceptually:

```text
$1 collateral -> 1 Yes share + 1 No share
```

Only one side should win in a normal binary market.

If Yes wins:

```text
Yes share -> $1
No share  -> $0
```

If No wins:

```text
Yes share -> $0
No share  -> $1
```

This is why Yes and No prices usually add up to about $1, ignoring spread, fees, and market frictions.

If Yes is around $0.70, No should be around $0.30. If they drift too far from that relationship, traders may arbitrage the difference.

## A Concrete Example

Suppose a market asks:

```text
Will A happen?
```

The order book currently lets you buy A at $0.25.

You spend $10:

```text
$10 / $0.25 = 40 A shares
```

If A wins:

```text
40 shares x $1 = $40 returned
Profit = $40 - $10 = $30
```

If A loses:

```text
40 shares x $0 = $0 returned
Loss = $10
```

So the risk is asymmetric:

```text
Maximum loss = the amount you paid
Maximum payout = number of shares x $1
```

You cannot lose more than your purchase cost on a simple long Yes or No position, but you can lose the entire amount.

## What Traders Are Really Playing

The game is not only "guess the outcome." It is "find a wrong price."

Imagine Yes is trading at $0.25.

If your true estimate is that Yes has a 25% chance, then the price is fair. Over many similar bets, there is no edge before costs and spread.

If your true estimate is that Yes has a 40% chance, then $0.25 may be attractive.

If your true estimate is that Yes has only a 10% chance, then $0.25 may be expensive.

The edge comes from the gap between your probability estimate and the market price:

```text
edge = your estimated probability - market price
```

For a Yes share:

```text
positive edge  -> your probability is higher than price
negative edge  -> your probability is lower than price
```

Of course, the hard part is knowing whether your estimate is better than the market's.

## Why People Can Exit Before Resolution

You do not always need to wait until the event resolves.

If you buy Yes at $0.40 and later the market price rises to $0.70, you can sell before the final outcome:

```text
Buy at $0.40
Sell at $0.70
Profit = $0.30 per share
```

This turns Polymarket into both a prediction game and a trading game. Traders can profit from price movement before the event ends, even if they never hold the share until final settlement.

But exiting depends on liquidity. A displayed price does not guarantee you can sell a large position at that price. You need buyers on the other side.

## The Mental Model

A clean mental model for Polymarket is:

```text
1. Each market has outcome shares.
2. Each winning share pays $1.
3. Each losing share pays $0.
4. The share price behaves like probability.
5. Your payout multiple is 1 / price.
6. Real prices come from an order book, not a simple pool formula.
7. Profit comes from buying probabilities that are too cheap or selling probabilities that are too expensive.
```

The pool model is still useful. If A has $10 and B has $30, it is natural to think A is priced around 25%. But in Polymarket, the real trade happens at the price another trader is willing to accept right now.

That is the heart of the mechanism: prediction becomes a tradable asset, probability becomes a price, and the final truth turns the winning token into $1.

## Sources

- [Polymarket 101](https://docs.polymarket.com/polymarket-101)
- [Polymarket Prices & Orderbook](https://docs.polymarket.com/concepts/prices-orderbook)
- [Polymarket Positions & Tokens](https://docs.polymarket.com/concepts/positions-tokens)
- [Polymarket Resolution](https://docs.polymarket.com/concepts/resolution)
