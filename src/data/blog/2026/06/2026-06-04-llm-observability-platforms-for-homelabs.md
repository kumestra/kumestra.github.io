---
title: "LLM Observability Platforms for Homelabs"
description: "A practical comparison of Langfuse-like tracing and evaluation platforms for personal Docker self-hosting."
pubDatetime: 2026-06-04T01:05:57Z
tags: ["llm-observability", "evals", "self-hosting", "homelab"]
---

LLM observability tools are becoming the boring infrastructure layer that AI apps need once they move beyond a few prompt calls. They collect traces, token usage, latency, cost, user feedback, prompt versions, datasets, and evaluation scores. In other words: they help answer what happened, why it happened, what it cost, and whether the result was any good.

For personal projects and homelabs, the question is not just "which tool has the most features?" It is also:

- Can I run it with Docker without turning my server into a space heater?
- Is the useful part actually open source?
- Will the cloud plan stay cheap if I give up on self-hosting?
- Does it handle evals, or only logs?
- Does it fit my stack, or does it assume I live inside one framework?

This comparison looks only at Langfuse-like LLM observability and evaluation platforms. It does not cover general APM tools, broad MLOps suites, or pure prompt-management products.

Prices and GitHub stars are a snapshot from 2026-06-04.

## Quick Comparison

| Tool | Homelab fit | Open source status | Docker/self-host posture | GitHub stars | Cloud pricing snapshot |
| --- | --- | --- | --- | ---: | --- |
| [Langfuse](https://langfuse.com/) | Best all-rounder if you have RAM | MIT for core, except `ee` folders | Official Docker Compose. Langfuse recommends at least 4 cores, 16 GiB RAM, and 100 GiB disk for a VM. | ~28.4k | Free 50k units/month, Core $29/month, Pro $199/month, Enterprise $2,499/month plus usage |
| [Opik](https://github.com/comet-ml/opik) | Strong Langfuse competitor | Apache-2.0 | Docker Compose via `./opik.sh`; estimate 4 vCPU, 8-16 GiB RAM, 50 GiB disk for comfortable local use | ~19.4k | Free cloud 25k spans/month, Pro $19/month with 100k spans/month, $5/100k extra spans |
| [Arize Phoenix](https://github.com/arize-ai/phoenix) | Lightest serious option | Elastic License 2.0, marketed as open source | Runs by pip, Docker, or Kubernetes; estimate 2 vCPU and 4-8 GiB RAM for small personal use | ~10k | Phoenix self-host is free; Arize AX Free has 25k spans/month; AX Pro is $50/month |
| [Helicone](https://github.com/Helicone/helicone) | Good if you want gateway-style logging | Apache-2.0 | Docker Compose, but stack includes web, worker, API server, Supabase, ClickHouse, and MinIO; estimate 4 vCPU and 8-16 GiB RAM | ~5.8k | Free 10k requests/month, Pro $79/month, Team $799/month, usage-based |
| [LangWatch](https://github.com/langwatch/langwatch) | Powerful, but heavier | Open-core: Apache-2.0 plus enterprise modules | Self-hosted. v3 favors Helm; Docker Compose is more local/dev. Official local minimum is 4 CPU, 16 GiB RAM, 50 GiB disk | ~3.3k | Developer free, Launch EUR59/month, Accelerate EUR199/month, Enterprise custom |
| [Laminar](https://github.com/lmnr-ai/lmnr) | Promising for agent tracing | Apache-2.0 | Docker Compose and Helm; estimate 2-4 vCPU and 4-8 GiB RAM for light use | ~3k | Managed cloud exists, but public pricing was not clearly surfaced in the checked sources |
| [Lunary](https://lunary.ai/) | Simple, but Docker is awkward | Apache-2.0 community edition | Free community edition can run from source; official Docker images are Enterprise/private registry. Estimate 2 vCPU, 4 GiB RAM, plus Postgres | ~1.4k | Free 10k events/month, Team $20/user/month, $10 per extra 50k events |
| [Langtrace](https://www.langtrace.ai/) | Small and simple | Open source | Self-host available; estimate 2 vCPU and 4 GiB RAM | ~433 | Free 5k spans/month, Growth $31/user/month, Enterprise custom |
| [LangSmith](https://www.langchain.com/pricing) | Not ideal for personal Docker | Proprietary platform | Self-host and hybrid are Enterprise only | N/A | Developer free with 5k base traces/month, Plus $39/seat/month, Enterprise custom |
| [Braintrust](https://www.braintrust.dev/docs/plans-and-limits) | Not ideal for personal Docker | Proprietary platform | Self-hosting is for enterprise customers | N/A | Starter free, Pro $249/month, Enterprise custom |

## What I Would Run First

If I were choosing for a homelab, I would start with either Phoenix or Langfuse.

Phoenix is the easiest serious answer when the priority is "I want traces and evals without feeding a database stack all my memory." It runs almost anywhere, it is friendly to notebooks and local development, and it has strong OpenTelemetry/OpenInference roots. It is not the most permissively licensed option because Phoenix uses the Elastic License 2.0, but it is still very practical for personal use.

Langfuse is the most complete open-source default. It covers tracing, prompt management, datasets, online and offline evaluation, feedback, cost tracking, and playground workflows. The tradeoff is infrastructure weight. The current Langfuse stack is built for real production telemetry, and the official VM recommendation is 4 cores, 16 GiB RAM, and 100 GiB disk. That is totally fine on a decent mini server, but it may be too much for a tiny VPS.

Opik is the third strong candidate. It feels especially compelling if evaluation workflows matter as much as tracing. Its cloud plan is also unusually friendly: $19/month for Pro with 100k spans/month is approachable for solo builders. For self-hosting, I would budget similar resources to Langfuse-lite: not absurd, but more than a toy app.

## How the Tools Feel Different

Langfuse is the "one table to rule them all" choice. It is broad, mature, and well adopted. If you want one place for traces, prompts, datasets, eval scores, annotation, cost, and experiments, Langfuse is hard to beat. It is also the tool I would compare everything else against first.

Phoenix is more developer-lab shaped. It is excellent when you want to instrument quickly, inspect traces, run evals, and keep the local setup low-friction. It is especially appealing if you like OpenTelemetry and want observability to look like observability, not a bespoke LLM-only island.

Opik leans into testing and evaluation. It has tracing, but the interesting part is the full loop around datasets, experiments, LLM-as-judge metrics, PyTest integration, and production monitoring. It is a good fit if your main pain is "how do I know this prompt or agent change made things better?"

Helicone is strongest when you want an LLM gateway and request analytics. It is very good at request logging, cost tracking, caching, rate limits, fallback behavior, and provider routing. If your app already routes model calls through a proxy layer, Helicone makes sense. If you mainly want offline evals and prompt experiments, I would compare it carefully against Langfuse or Opik.

LangWatch is interesting for agent-heavy systems. Its pitch is not just "trace this model call", but "simulate, evaluate, and monitor agents end to end." That makes it powerful, but also bigger. The official docs list a local Docker minimum of 4 CPU, 16 GiB RAM, and 50 GiB disk, and the production profile is much larger. I would not choose it as my first tiny homelab install, but I would keep it in mind for serious agent testing.

Laminar is the newer agent-observability entrant that feels worth watching. It is OpenTelemetry-native, self-hosts with Docker Compose, exposes SQL access to data, and focuses on real-time traces, dashboards, evals, annotation, and datasets. It is smaller than Langfuse or Phoenix, but the design direction is appealing.

Lunary is simpler and chatbot-product oriented. It has conversation tracking, feedback, prompt templates, analytics, topic classification, and evaluations. The catch for homelab users is Docker: the public docs say Docker setup is available only with Enterprise Edition, while the community edition can still run from source. That makes it less clean for "drop this into my compose stack."

Langtrace is the small option. It is lower-star, lower-footprint, and has a cheaper cloud tier. For a personal project that needs basic traces, metrics, annotations, dataset curation, and evals, it may be enough. I would not expect the same ecosystem gravity as Langfuse, Phoenix, or Opik.

LangSmith and Braintrust are both strong products, but they do not fit the personal Docker-first lens. LangSmith is excellent if you already live in LangChain or LangGraph, but self-hosting is gated behind Enterprise. Braintrust is one of the strongest eval platforms, but its self-hosting story is enterprise-oriented and Pro starts at $249/month. For a homelab, they are useful reference points more than likely installs.

## Hardware Reality

For LLM observability, storage and memory matter more than people expect. A trace is not just one row. Agent traces can contain nested spans, tool calls, retrieval context, prompts, completions, token counts, scores, metadata, and user feedback. Multiply that by experiments or chat sessions, and a "small" project can create a lot of analytical data.

There are three rough tiers:

| Tier | Good candidates | Practical hardware |
| --- | --- | --- |
| Tiny personal test | Phoenix, Langtrace, Lunary-from-source | 2 vCPU, 4-8 GiB RAM, 20-50 GiB disk |
| Comfortable homelab | Langfuse, Opik, Helicone, Laminar | 4 vCPU, 8-16 GiB RAM, 50-100 GiB disk |
| Heavy agent/eval stack | LangWatch, production Langfuse, production Opik | 4+ vCPU, 16+ GiB RAM, 100+ GiB disk, preferably planned retention |

The important trick is retention. Most homelab use does not need years of traces. If the tool lets you cap retention, export old data, or keep only eval datasets long-term, use that. Observability data ages quickly.

## The Shortlist

For most personal Docker users:

1. Use Phoenix if you want the lowest-friction serious local tool.
2. Use Langfuse if you want the most complete open-source LangSmith-like platform.
3. Use Opik if your workflow is eval-first and you want strong testing primitives.
4. Use Helicone if you want an LLM gateway/proxy with observability.
5. Use Laminar if you want an agent-focused open-source platform and are comfortable with a newer project.

For heavier agent testing, evaluate LangWatch. For simple logging with lower expectations, try Langtrace or Lunary. For SaaS-first teams with budget, LangSmith and Braintrust are worth comparing, but they are not the natural homelab answer.

My own bias for a personal server would be:

- Start with Phoenix to learn what I actually need.
- Move to Langfuse when I want prompt management, datasets, annotation, and evals in one place.
- Try Opik if I find myself building more regression tests than dashboards.

That progression keeps the first install light, but leaves room for a more complete engineering loop later.

## Sources Checked

- [Langfuse pricing](https://langfuse.com/pricing) and [Docker Compose self-hosting](https://langfuse.com/self-hosting/deployment/docker-compose)
- [Opik GitHub repository](https://github.com/comet-ml/opik) and [Comet pricing](https://www.comet.com/site/pricing/)
- [Arize Phoenix GitHub repository](https://github.com/arize-ai/phoenix) and [Arize pricing](https://arize.com/pricing/)
- [Helicone GitHub repository](https://github.com/Helicone/helicone) and [Helicone pricing](https://www.helicone.ai/pricing)
- [LangWatch GitHub repository](https://github.com/langwatch/langwatch), [self-hosting overview](https://langwatch.ai/docs/self-hosting/overview), and [sizing guide](https://langwatch.ai/docs/self-hosting/configuration/sizing-and-scaling)
- [Laminar GitHub repository](https://github.com/lmnr-ai/lmnr)
- [Lunary pricing](https://lunary.ai/pricing) and [Docker self-hosting docs](https://docs.lunary.ai/more/self-hosting/docker)
- [Langtrace pricing](https://www.langtrace.ai/pricing) and [documentation](https://docs.langtrace.ai/introduction)
- [LangSmith pricing](https://www.langchain.com/pricing)
- [Braintrust plans and limits](https://www.braintrust.dev/docs/plans-and-limits)
