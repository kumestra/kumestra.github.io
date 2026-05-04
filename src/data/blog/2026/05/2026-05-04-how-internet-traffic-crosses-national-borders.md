---
title: How Internet Traffic Crosses National Borders
description: The physical cables, logical gateways, and traffic patterns that connect a country to the global internet — with mainland China as a worked example.
pubDatetime: 2026-05-04T12:00:20Z
tags: [internet, networking, infrastructure, china, bgp, undersea-cables]
---

When data leaves your laptop for a server abroad, it does not simply "hop onto the internet." It funnels through a surprisingly small number of physical and logical chokepoints maintained by national-scale carriers. Understanding that funnel explains a lot of otherwise mysterious behavior — why latency to certain regions is asymmetric, why evenings feel slower than mornings, and why some countries can throttle or filter cross-border traffic at all.

## The Two Layers

Cross-border connectivity is built from two layers stacked on top of each other.

### 1. Physical Layer — Cables and Landing Stations

The overwhelming majority of intercontinental data travels through **submarine fiber-optic cables** buried on the sea floor. Where those cables come ashore, they terminate at **landing stations** — coastal facilities that house the equipment to bring the optical signal onto a country's terrestrial network.

A country with serious international traffic does not have one landing station; it has several, distributed across different coastal cities so that a single anchor strike, earthquake, or fire cannot disconnect the whole country. Land-locked or contiguous neighbors are connected via cross-border terrestrial fiber instead.

### 2. Logical Layer — International Gateways

Above the physical layer sit **International Gateways** (often called *IGW* or, in the Chinese system, *International Communications Gateway Bureaus*). These are not single routers — each one is a large data center, or a cluster of data centers, run by a major telecom carrier and packed with high-capacity routers, BGP peering equipment, and traffic-monitoring infrastructure.

Their core jobs are:

- **BGP route distribution** — deciding which submarine cable a packet should take to reach its destination AS.
- **Traffic accounting and metering** — telecoms settle international transit by volume.
- **Security and policy enforcement** — many states deploy deep packet inspection (DPI) and filtering at exactly these gateways, because they are the natural choke point.

## Why "A Few Routers" Is the Wrong Mental Model

It is tempting to picture a country's exit as a handful of big routers. Two pressures rule that out:

1. **Bandwidth.** A medium-sized country pushes tens of terabits per second across its borders at peak. No single device handles that.
2. **Redundancy.** If a single point fails, the country's entire international connectivity drops. Real designs are distributed across multiple cities, multiple cables, and multiple carriers.

So in practice every "core gateway" is itself a fleet of devices, and the country has several of these fleets spread geographically.

## A Worked Example: Mainland China

Mainland China is a useful case study because it has a small, named, and publicly tracked set of international gateways.

### From Three Hubs to Seven

For roughly thirty years, almost all international internet traffic in mainland China funnelled through three cities:

- **Beijing** — the northern hub, primarily towards North America, Europe, and parts of Asia.
- **Shanghai** — the largest, with multiple submarine cable landings (Chongming, Nanhui, Lingang) and the heaviest international load.
- **Guangzhou** — the southern hub, primarily towards Southeast Asia, Hong Kong/Macau/Taiwan, and Oceania.

In **July 2024**, the Ministry of Industry and Information Technology approved four additional International Communications Gateway Bureaus to relieve pressure on Beijing/Shanghai/Guangzhou and to shorten paths from inland and border regions:

| New gateway | Primary direction |
| --- | --- |
| Nanning (Guangxi) | ASEAN countries |
| Qingdao (Shandong) | Japan, Korea, Northeast Asia |
| Kunming (Yunnan) | South Asia and overland Southeast Asia |
| Haikou (Hainan) | Cross-border digital trade for the Hainan Free Trade Port |

The architecture is shifting from "three core exits" to "seven core exits."

### How a Packet Reaches the Border

```mermaid
flowchart LR
    A[End user] --> B[City/provincial network]
    B --> C[National backbone<br/>e.g. ChinaNet 163, Unicom 169]
    C --> D[International gateway<br/>BJ / SH / GZ / NN / QD / KM / HK]
    D --> E[Submarine or land cable]
    E --> F[Foreign AS]
```

1. Traffic from a user climbs into the metro / provincial network.
2. The provincial edge hands it off to a national backbone (China Telecom's ChinaNet 163, China Unicom's 169, etc.).
3. The backbone routes it to one of the seven international gateway facilities.
4. From there it leaves the country on a submarine cable (NCP, TPE, APCN-2, …) or an overland fiber route.

## Topology: "Stars Inside, Ropes Out"

Internally, the country's network is dense and highly redundant — provincial backbones, metropolitan area networks, multi-carrier IXPs, and a rich mesh of paths between cities. If one province goes dark, traffic reroutes through others.

Externally, all that complexity collapses into a small number of egress points. A useful way to picture it:

> **Stars inside, a few ropes out.**

This shape has three direct consequences:

- **Logical narrowing.** Whether a packet originates in a remote village or a tier-one city, if its destination is a foreign IP it ends up traversing one of those few gateway cities.
- **Natural filter location.** Any system that wants to inspect or control cross-border traffic — China's Great Firewall is the most discussed example — has an obvious place to deploy DPI hardware: the gateways themselves.
- **Physical concentration.** Submarine cable landings are expensive and geographically constrained, so cables are concentrated in a handful of cities rather than spread evenly along the coast.

## Capacity and Traffic, in Numbers

The numbers below are figures cited in the source conversation; treat them as orders of magnitude rather than precise current values, since both bandwidth and traffic grow fast.

### International Bandwidth (the "pipe size")

- End of **2019**: roughly **8.8 Tbps**.
- End of **2020**: roughly **11.5 Tbps** — over 30% year-on-year growth.
- Target around **2025**: on the order of **48 Tbps**, approaching a **50 Tbps** scale by 2026.

### Real Throughput (water actually flowing through the pipe)

Pipes are not run at 100% utilization. A reasonable rule of thumb:

- **Off-peak average:** ~50–70% of capacity, roughly **20–30 Tbps**.
- **Evening peak (≈ 20:00–23:00):** approaches the ceiling, **40+ Tbps**, with congestion, packet loss, and jitter all rising.

### What's Driving the Growth

A handful of "super consumers" dominate:

- **Generative AI.** Cross-border API calls and inter-cluster data sync became a serious traffic class.
- **Short video and cross-border live commerce.** TikTok / TikTok Shop / Temu / Shein streams are the single largest contributor — high-bitrate video does not compress its way out of being expensive.
- **Cloud and gaming companies going overseas.** Game patches, save-state sync, and CDN fan-out from Chinese platforms into global regions.
- **5G densification.** A larger 5G base-station footprint means individual mobile users sustain higher per-second bitrates than they did on 4G.

## Latency: Why "Wider" ≠ "Faster"

A common surprise: bandwidth keeps growing, but the *feel* of cross-border access does not improve at the same rate. Latency has its own constraints.

### By Distance

| Destination | Approximate RTT from mainland China |
| --- | --- |
| Domestic (5G / fiber) | < 10 ms |
| Japan / Korea via Qingdao | 30–50 ms |
| Southeast Asia via Kunming / Nanning | ~20 ms (down from ~70 ms before the new gateways) |
| US West Coast | 150–200 ms |
| Europe | 200–300 ms |

The intercontinental floor is set by physics — light through fiber covers ~200 km/ms — and inflated by every router hop and inspection step in between.

### Why Peak Hours Hurt More Than the Average Suggests

- **Congestion.** When throughput approaches capacity at the gateway, queues build and packets get dropped.
- **DPI cost.** Every packet that crosses the border is inspected. Inspection adds processing latency that scales with traffic volume.
- **BGP path quirks.** Carrier routing policy can send a packet on a non-shortest path — e.g. a request to a Southeast Asian server momentarily detouring via a US transit before coming back.
- **Jitter.** RTT bouncing between, say, 180 ms and 400 ms is far worse for VoIP, video calls, and games than a stable but higher number.

### What That Means in Practice

- ✅ **Domestic services:** effectively LAN-grade.
- ✅ **Email, academic resources, business apps:** usable, with noticeable but tolerable latency.
- ⚠️ **HD/4K cross-border video:** depends on buffering; degrades during peak hours.
- ❌ **Real-time international gaming:** typically unworkable without CDN/relay assistance to optimize the path.

## Takeaways

- Cross-border internet is built on two layers: **submarine/land cables** at the physical layer and **international gateway facilities** at the logical layer.
- A country's egress always looks like a few fat funnels rather than a true mesh, because cable landings are scarce and the operational/regulatory benefits of concentration are large.
- Mainland China codifies this pattern explicitly: long-running Beijing/Shanghai/Guangzhou, plus Nanning/Qingdao/Kunming/Haikou approved in 2024, totalling **seven** gateways.
- **Bandwidth** has roughly 5×'d in half a decade (≈ 10 Tbps → ≈ 50 Tbps), but **real throughput** sits well below the ceiling on average and presses against it at night.
- **Latency** is bounded by geography and inspection overhead, so wider pipes alone do not make cross-border access feel fast.
