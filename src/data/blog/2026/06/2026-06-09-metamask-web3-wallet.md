---
title: "MetaMask 如何让网站和钱包通信"
description: "用 OAuth 类比解释 MetaMask、window.ethereum、钱包地址、签名登录和交易授权之间的关系。"
pubDatetime: 2026-06-09T05:17:50Z
tags: [metamask, ethereum, web3, wallet]
---

## 先说结论

MetaMask 可以理解成浏览器里的一个 Web3 钱包中间层。普通网站本来不能直接访问你的钱包、私钥或浏览器扩展内部状态，但 MetaMask 会在网页环境里注入一个受控的 JavaScript provider，也就是常见的 `window.ethereum`。

网站可以调用这个 provider，请求连接账号、读取当前网络、发起签名、建议交易。MetaMask 再决定哪些请求可以直接返回，哪些请求必须弹出确认窗口让用户批准。

最重要的安全边界是：网站可以拿到你的公开钱包地址，也可以拿到你批准后的签名或交易结果，但不能拿到私钥。

## `window.ethereum` 是什么

`window.ethereum` 不是浏览器标准 API。Chrome、Firefox 或 Safari 默认都不会提供它。它通常由 MetaMask、Rabby、Coinbase Wallet 这类钱包扩展注入到网页里。

网站代码可以这样检测它：

```js
if (window.ethereum) {
  const accounts = await window.ethereum.request({
    method: "eth_requestAccounts",
  });

  console.log(accounts);
}
```

这里的 `request()` 是网站和钱包沟通的主要入口。常见请求包括：

```js
ethereum.request({ method: "eth_requestAccounts" });
ethereum.request({ method: "eth_chainId" });
ethereum.request({ method: "personal_sign", params: [message, address] });
ethereum.request({ method: "eth_signTypedData_v4", params: [address, typedData] });
ethereum.request({ method: "eth_sendTransaction", params: [transaction] });
```

网站看起来是在调用一个普通 JavaScript 对象，但这个对象背后连接的是 MetaMask 扩展。

## 加载网页时发生了什么

当用户打开一个网页时，MetaMask 的扩展脚本会运行。它通常不会去改网页 HTML，而是把一个 provider 注入到页面的 JavaScript 环境里。

可以把流程简化成这样：

```text
浏览器加载网页
  -> MetaMask 内容脚本运行
  -> 注入 provider
  -> 页面获得 window.ethereum
  -> 网站 JS 调用 window.ethereum.request(...)
  -> 请求被转发给 MetaMask 扩展
  -> MetaMask 检查权限或弹出确认
  -> 结果返回给网站 JS
```

真实实现里还会经过浏览器扩展的隔离环境、`window.postMessage`、`chrome.runtime` 消息等桥接机制。普通网页 JS 不能直接调用扩展的内部 API，所以 `window.ethereum` 本质上是一个受控消息网关。

## 网站能拿到什么

连接 MetaMask 后，网站最常拿到的是钱包地址：

```text
0x1234...
```

严格说，这不是原始公钥，而是从公钥派生出来的 Ethereum 地址。在日常 Web3 登录和交易场景中，它就是你的公开账号标识。

MetaMask 可能返回给网站的信息包括：

| 类型 | 例子 | 是否敏感 |
|---|---|---|
| 钱包地址 | `0xabc...` | 公开信息，但会关联你的链上活动 |
| 当前网络 | Ethereum、Polygon、Arbitrum 等 | 通常不敏感 |
| 签名结果 | `0xSignature...` | 需要认真看签名内容 |
| 交易哈希 | `0xTransactionHash...` | 公开链上记录 |
| 错误信息 | 用户拒绝、网络错误等 | 通常不敏感 |

MetaMask 不会把私钥返回给网站。无论网站 JavaScript 怎么写，它都不能通过 `window.ethereum` 直接读取你的私钥。

## 签名如何证明你拥有地址

很多 Web3 网站会用“地址 + 签名”来登录用户。

流程通常是：

```text
网站请求连接钱包
  -> MetaMask 返回地址
  -> 网站生成一段登录消息
  -> 网站请求用户签名
  -> MetaMask 弹出确认
  -> 用户批准
  -> MetaMask 用私钥签名
  -> 网站拿到签名
  -> 网站验证签名是否匹配该地址
```

网站最终看到的是：

```text
地址 + 原始消息 + 签名
```

它可以用这些信息验证：这段签名确实来自拥有该地址私钥的人。网站不需要知道私钥本身。

一个比较好的登录消息应该包含随机 nonce：

```text
Login to example.com

Address: 0xabc...
Nonce: 934829
Issued At: 2026-06-09T05:17:50Z
```

nonce 的作用是防止重放攻击。如果没有 nonce，别人可能复用你过去签过的一段消息。现实中很多网站会使用 SIWE，也就是 Sign-In with Ethereum，来规范这种登录消息格式。

## 为什么它有点像 OAuth

MetaMask 连接网站的过程很像 Web2 里的 OAuth 授权。

相似点是：

| OAuth | MetaMask / Web3 |
|---|---|
| 网站请求访问权限 | 网站请求连接钱包 |
| 用户看到授权页面 | 用户看到 MetaMask 连接页面 |
| 网站获得用户身份信息 | 网站获得钱包地址 |
| 权限可以撤销 | 网站连接可以断开 |
| 权限有 scope | 钱包有账号和网络权限 |

所以“MetaMask 是 Web3 世界里的 OAuth”是一个不错的第一层理解。

但它们最关键的差异在于：OAuth 往往会发给网站一个 bearer token。谁拿到这个 token，谁就可以在有效期内调用对应 API。

MetaMask 通常不会给网站一个可以代替你花钱或签名的长期 token。网站想签名、想发交易、想授权代币额度，都要再次请求钱包。MetaMask 会弹出确认界面，用户批准后才执行。

可以粗略地说：

```text
OAuth:
“给你一个 token，你拿它去访问我的数据。”

MetaMask:
“给你我的公开地址。以后要签名或发交易，再来问我的钱包。”
```

## 连接权限意味着什么

MetaMask 的站点连接页面通常会显示类似这些权限：

```text
See your accounts and suggest transactions
Use your enabled networks
```

第一项表示网站可以看到已连接的钱包地址，也可以发起交易请求。这里的 “suggest transactions” 不代表网站能静默转走资产，而是它可以准备一笔交易并请求 MetaMask 弹窗确认。

第二项表示网站可以使用你为该站点启用的网络，比如 Ethereum、Polygon、Base、Arbitrum 等。网站可能会读取当前 chain id，也可能请求切换网络。

连接本身通常不是最高风险动作。真正需要仔细看的，是后续 MetaMask 弹出的签名、授权和交易确认。

## 哪些确认最需要小心

### 签名消息

签名可以用于登录，也可能用于授权某些链下行为。签名前要看清楚：

- 域名是不是你正在访问的网站
- 消息是否包含合理的用途
- 是否包含 nonce 和时间
- 有没有看不懂的大段结构化数据

### Token approve

`approve` 是高风险操作之一。它可能不是马上转账，而是授权某个合约以后可以花你的 token。

尤其要注意：

- 授权对象是不是可信合约
- 授权额度是不是无限额度
- 授权 token 是不是你以为的那个 token

### 交易请求

交易请求可能会转账、买卖、挂单、领取、交换资产，或者调用复杂合约。确认时要看：

- 目标合约地址
- 支付金额
- 资产类型
- 网络是否正确
- gas 费用是否异常

## 一个实用心智模型

可以把 MetaMask 分成三层来看：

```text
公开身份层:
  钱包地址、当前网络

证明控制权层:
  签名消息，证明你控制该地址对应的私钥

资产操作层:
  发交易、授权 token、调用合约
```

风险也大致按这三层递增：

| 层级 | 常见动作 | 风险 |
|---|---|---|
| 公开身份 | 连接钱包、暴露地址 | 低到中，主要是隐私关联 |
| 证明控制权 | 登录签名、结构化签名 | 中，需要确认消息内容 |
| 资产操作 | approve、swap、send transaction | 高，需要逐项检查 |

## 总结

MetaMask 并不是把你的钱包直接交给网站。它是在网页和钱包之间放了一个受控桥梁：网站通过 `window.ethereum` 发请求，MetaMask 检查权限、弹出确认、执行签名或交易，再把结果返回给网站。

网站通常能拿到的是公开地址、网络信息、用户批准后的签名或交易结果。它拿不到私钥，也不能在没有用户确认的情况下直接签名或发交易。

把它类比成 OAuth 有助于理解授权体验；但从安全模型看，MetaMask 更像是“每次敏感动作都要经过本地钱包批准”的签名设备。真正保护资产的关键，不只是是否连接网站，而是每一次签名、授权和交易确认时看清楚自己正在批准什么。
