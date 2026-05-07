---
title: JWT, opaque API keys, and why you rarely see JWTs in the wild
description: A walkthrough of JWT structure, signature generation, and why developer-facing API keys (sk-*, ghp_*) are almost always opaque tokens instead.
pubDatetime: 2026-05-07T09:18:58Z
tags: [jwt, auth, tokens, api-keys, security, oauth]
---

## What a JWT actually is

A JWT (JSON Web Token) is just **a string** — an ASCII string with three base64url-encoded segments joined by dots:

```
xxxxx.yyyyy.zzzzz
```

Because it's URL-safe ASCII, it fits anywhere a string fits: an HTTP header (`Authorization: Bearer <jwt>`), a cookie, a query parameter, localStorage. The "JSON" in the name refers to what's *inside* — the token you transmit is one flat string.

## The three parts

```mermaid
flowchart LR
    H[Header JSON] -->|base64url| A[xxxxx]
    P[Payload JSON] -->|base64url| B[yyyyy]
    A & B --> S["HMAC/RSA(xxxxx.yyyyy, key)"]
    S -->|base64url| C[zzzzz]
    A & B & C --> JWT["xxxxx.yyyyy.zzzzz"]
```

| Part | Source | Encoded as |
|---|---|---|
| Header | JSON object | base64url of JSON text |
| Payload | JSON object | base64url of JSON text |
| Signature | Raw bytes from HMAC/RSA | base64url of binary |

Note the asymmetry: the first two parts are base64url of **JSON strings**; the third is base64url of **raw binary**. It's not JSON, not text — just the cryptographic output bytes.

Also worth noting: it's **base64url**, not regular base64. Same idea, but uses `-` and `_` instead of `+` and `/`, and drops `=` padding — safe to put in URLs and headers without escaping.

## What goes in the header

The header is metadata about the token itself. No user data.

```json
{
  "alg": "HS256",
  "typ": "JWT",
  "kid": "key-id-2"
}
```

- `alg` — signing algorithm
- `typ` — token type
- `kid` — key ID (used when you rotate keys)

## What goes in the payload

The payload is the **claims** — where user info lives.

```json
{
  "sub": "user_12345",
  "name": "Alice",
  "roles": ["admin", "editor"],
  "iat": 1700000000,
  "exp": 1700003600
}
```

Two kinds of claims mixed together:

**Standard (registered) claims** — short names from the spec:

- `sub` — subject (usually the user ID)
- `iss` — issuer
- `aud` — audience
- `iat` — issued at
- `exp` — expiration
- `nbf` — not valid before
- `jti` — unique token ID

**Custom claims** — anything else (`email`, `roles`, `tenant_id`, …).

⚠️ The payload is **signed, not encrypted**. Anyone can base64-decode it and read every claim. So:

- ✅ User ID, roles, expiration — fine
- ❌ Passwords, API keys, anything sensitive — never

If you need confidentiality, use **JWE** (JSON Web Encryption) or keep sensitive data on the server and reference it by ID.

## How the signature is generated

The signature is computed over the **already-encoded** header and payload — not the original JSON.

```
signingInput = base64url(header) + "." + base64url(payload)
signature    = base64url(HMAC_SHA256(signingInput, secret))
JWT          = signingInput + "." + signature
```

### Two algorithm families

- **HMAC** (`HS256`, `HS384`, `HS512`) — symmetric. Same secret signs and verifies. Simple and fast, but anyone who can verify can also forge.
- **RSA / ECDSA** (`RS256`, `ES256`, …) — asymmetric. Private key signs, public key verifies. Lets you hand the public key to other services without giving them the power to mint tokens.

## How verification works

The server only needs to **know the secret in advance**. The header and payload arrive **with the token**.

```mermaid
sequenceDiagram
    Client->>Server: Authorization: Bearer xxx.yyy.zzz
    Server->>Server: split on dots
    Server->>Server: recompute HMAC(xxx.yyy, secret)
    Server->>Server: compare to zzz
    alt match
        Server->>Server: base64-decode payload, check exp
        Server->>Client: 200 OK
    else mismatch
        Server->>Client: 401 Unauthorized
    end
```

If the recomputed signature matches the third segment, the client didn't tamper with the header or payload — any byte change produces a different signature, and only someone holding the secret can produce a valid one.

### The key insight: self-describing tokens

The server doesn't need to remember:

- who the user is
- when the token was issued
- when it expires
- what permissions it grants

All of that travels inside the token. The server only needs the **secret** to confirm "yes, I (or someone I trust with the secret) signed this." That's why JWTs are called **stateless** — no session storage, no DB lookup.

### Verification gotchas

- **Always verify the signature** before reading any claim. Never just base64-decode and trust.
- **Check `exp` yourself** — the signature stays valid forever; expiration is just a claim the server enforces.
- **Pin the algorithm.** Historically there were attacks where a client sent `alg: none` and naive servers accepted unsigned tokens.

## Opaque tokens — the other big format

Tokens you see on developer dashboards usually look like this:

```
sk-proj-abc123XYZ...           # OpenAI-style
ghp_16C7e42F292c6912E7710c...  # GitHub personal access token
xoxb-1234-5678-abcdef...       # Slack bot token
```

No dots. No structure visible to you. These are **opaque tokens** (also called "reference tokens"). The string itself means nothing — it's a lookup key. The server stores something like:

```
token "sk-proj-abc123..." → { user: "...", scopes: [...], expires: ... }
```

When a request comes in, the server looks it up to find out who it belongs to and what it can do.

## JWT vs. opaque — the core difference

| | **JWT** | **Opaque** |
|---|---|---|
| Format | `xxx.yyy.zzz` (three parts) | Random string, no public structure |
| Contains data? | Yes — claims are inside | No — just an ID |
| Verification | Cryptographic (signature check) | Database lookup |
| Server state | Stateless | Stateful (must store the token) |
| Revocation | Hard (need a blocklist) | Easy (delete the row) |
| Size | Larger (hundreds of bytes) | Small (~40 chars) |

### Why API providers usually pick opaque

- **Revocation matters a lot.** If a key leaks (gets pushed to GitHub), you want to kill it instantly. With JWT you'd need a blocklist that defeats the stateless benefit.
- **Long-lived.** API keys often live for months or years. JWT's "stateless" advantage shrinks when you need server state for revocation anyway.
- **Don't need claims in the token.** The server has the user record already; one DB lookup per request is fine.
- **Smaller, easier to copy/paste.**

### Why apps pick JWT

- **Short-lived** (minutes to hours), so revocation matters less.
- **Microservices** can verify locally without calling an auth server on every request.
- **Federation** — service A mints a token, service B verifies with a public key, no shared DB.

**Rule of thumb**: machine-to-machine API keys → opaque. User session/access tokens in distributed systems → JWT.

## Why you rarely *see* a JWT

A common observation: "I get tokens from many websites, but I've never encountered a JWT — who actually uses them?"

The answer: JWTs are usually **invisible to you**. They live behind the scenes in browser/app auth flows, not on developer dashboards.

### Where you've used JWTs without seeing them

- **"Sign in with Google/GitHub/Apple"** — the OIDC flow returns an `id_token` that's a JWT. Your browser hands it to the app; you never see it.
- **Firebase Auth, AWS Cognito, Auth0, Okta, Supabase Auth** — all hand out JWTs as session/access tokens.
- **Kubernetes service accounts** — pods get a JWT mounted at `/var/run/secrets/...` to talk to the API server.
- **Apple Push Notifications, App Store Connect API** — you sign requests with a JWT generated from a private key.
- **GitHub Apps** (not personal access tokens) — the app authenticates by signing a JWT with its private key, then exchanges it for an installation token.

### Why they stay invisible

JWTs are designed to be passed by software, not pasted by humans:

- **Long** (often 500–2000 chars) — annoying to copy
- **Short-lived** (5 min – 1 hour) — pointless to put on a dashboard
- **Auto-refreshed** — your browser/SDK silently swaps in a new one

API keys, by contrast, are designed for **humans to copy into config files**, so they're short, opaque, and long-lived.

| You see it on a dashboard? | Token type |
|---|---|
| Yes (you copy/paste it) | **API key** — opaque, long-lived |
| No (browser/SDK handles it) | **JWT** — typically in user auth flows |

### Try it yourself

Open DevTools → Application → Cookies or Local Storage on any modern web app. Look for something with three dot-separated chunks of base64 gibberish — that's a JWT. Most SaaS dashboards have one.

## Token prefixes — convention, not standard

The `sk-`, `ghp_`, `xoxb-` prefixes aren't a standard, but they exist for practical reasons:

**1. Secret scanning.** GitHub, GitGuardian, etc. scan public repos for leaked credentials. A distinctive prefix is easy to regex-match. When GitHub spots `ghp_...` in a public commit, it pings GitHub's API to auto-revoke it.

**2. Token-type routing.** A single service often issues many token kinds:

- `sk-` — secret key (server-side, full access)
- `pk-` — publishable key (safe in browsers)
- `rk-` — restricted key

The prefix lets the server (and humans) tell them apart at a glance.

**3. Vendor identification.** `sk-` alone is ambiguous; `sk-ant-` or `sk-proj-` tells you the vendor.

### GitHub's documented scheme

GitHub publishes their format:

```
ghp_  → personal access token
gho_  → OAuth access token
ghu_  → user-to-server token
ghs_  → server-to-server token
ghr_  → refresh token
```

After the prefix it's just random bytes plus a checksum. Still opaque to you — you don't decode it, you just send it.

### How servers handle them internally

Two common patterns:

- **Pure lookup.** The token is purely random; server hashes it and looks it up in a table. Most common.
- **Embedded info.** The token encodes the account ID or shard, e.g. `sk-proj-<projectId>-<random>-<checksum>`. The server parses out the project ID to route to the right database shard, then verifies the random part. Still opaque to clients.

## Bottom line

- **Standardized format with a public parser**: JWT. Anyone can decode it because the format is published.
- **No standard format, parser-per-vendor**: API keys / opaque tokens. Each provider invents its own scheme; you treat it as a black box.

The `sk-...` style is a **convention**, not a standard. There's no spec for it — each company chooses prefixes and (sometimes) documents them.
