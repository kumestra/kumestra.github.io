---
title: Network Namespace Pattern 1 — Direct veth + NAT (Host as a Mini Router)
description: A walkthrough of the simplest way to give a new Linux network namespace internet access, using a veth pair and NAT — modeled exactly like a home router with one connected client.
pubDatetime: 2026-05-15T13:00:58Z
tags: [networking, linux, network-namespace, veth, nat, containers]
---

A **network namespace** is a Linux kernel feature that gives a process its own private copy of the network stack — its own interfaces, routing table, ARP cache, iptables rules, socket table, and port space. A fresh namespace has only `lo` (and it's down) — useless until you wire it up to somewhere with actual connectivity.

This note walks through **Pattern 1**: the simplest possible wiring, connecting one new namespace to the host via a single veth pair, with NAT on the host so packets reach the outside world. It's container networking stripped to its smallest possible shape.

## The mental model: host as a home router

Pattern 1 maps almost component-for-component onto a home Wi-Fi router serving one client.

| Home network | Pattern 1 |
|---|---|
| ISP cable plugged into router's WAN port | `ens33` (or whatever physical NIC) connected to the LAN |
| The router itself | The **host namespace** |
| Router's LAN-side interface | `veth-host` with `10.0.0.1/24` |
| Router's LAN IP (e.g. `192.168.1.1`) | `10.0.0.1` — the namespace's default gateway |
| Connected device (iPhone, MacBook) | The **demo namespace** with `eth0` = `10.0.0.2` |
| Device's private IP from DHCP | `10.0.0.2` (manually assigned in this demo) |
| Router's NAT | `iptables MASQUERADE` |
| "Share internet" toggle | `sysctl net.ipv4.ip_forward=1` |
| Default gateway on the device | `ip route add default via 10.0.0.1` |

Pattern 1 is *"host machine acting as a NAT router for one connected client."* Add a bridge in the middle (Pattern 2) and you've reinvented `docker0`. 🐳

## Structure

```mermaid
flowchart TB
    INET[("🌐 Internet / LAN<br/>8.8.8.8, etc.")]

    subgraph host["🖥️ Host namespace (the 'router')"]
        direction TB
        ENS["<b>ens33</b><br/>192.168.1.156/24<br/>= WAN side"]
        FWD{{"IP forwarding<br/>net.ipv4.ip_forward=1"}}
        NAT[/"iptables MASQUERADE<br/>POSTROUTING -s 10.0.0.0/24 -o ens33"/]
        VH["<b>veth-host</b><br/>10.0.0.1/24<br/>= LAN side"]
        RT1[("Routing table<br/>10.0.0.0/24 → veth-host<br/>default → ens33")]
    end

    subgraph ns["📦 demo namespace (the 'client')"]
        direction TB
        LO["lo<br/>127.0.0.1/8"]
        VD["<b>eth0</b> (was veth-demo)<br/>10.0.0.2/24"]
        RT2[("Routing table<br/>10.0.0.0/24 → eth0<br/>default → 10.0.0.1")]
    end

    INET <--> ENS
    ENS <--> FWD
    FWD <--> NAT
    NAT <--> VH
    VH <==>|"veth pair<br/>(kernel-internal wire)"| VD

    style host fill:#e8f4f8,stroke:#2980b9
    style ns fill:#fef5e7,stroke:#e67e22
    style INET fill:#eafaf1,stroke:#27ae60
```

Same picture in ASCII for terminal viewing:

```
                          ┌──────────────────────────┐
                          │   🌐 LAN / Internet       │
                          └────────────┬─────────────┘
                                       │
                                       │ physical
                                       ▼
   ┌───────────────────────────────────────────────────────────────┐
   │ Host namespace                                                │
   │                                                               │
   │   ┌────────────────┐                                          │
   │   │ ens33          │  192.168.1.156/24                        │
   │   │ (real NIC)     │                                          │
   │   └───────┬────────┘                                          │
   │           │                                                   │
   │           │  ┌──────────────────────────┐                     │
   │           ├──┤ iptables MASQUERADE      │                     │
   │           │  │ (rewrite src IP)         │                     │
   │           │  └──────────────────────────┘                     │
   │           │                                                   │
   │           │  ┌──────────────────────────┐                     │
   │           ├──┤ IP forwarding enabled    │                     │
   │           │  └──────────────────────────┘                     │
   │           │                                                   │
   │   ┌───────▼────────┐                                          │
   │   │ veth-host      │  10.0.0.1/24                             │
   │   └───────┬────────┘                                          │
   │           │                                                   │
   └───────────┼───────────────────────────────────────────────────┘
               │
               │  veth pair = virtual ethernet cable
               │  (one end here, the other in 'demo')
               │
   ┌───────────┼───────────────────────────────────────────────────┐
   │ demo namespace                                                │
   │   ┌───────▼────────┐                                          │
   │   │ eth0           │  10.0.0.2/24                             │
   │   │ (was veth-demo)│  default route → 10.0.0.1                │
   │   └────────────────┘                                          │
   │                                                               │
   │   ┌────────────────┐                                          │
   │   │ lo             │  127.0.0.1/8                             │
   │   └────────────────┘                                          │
   │                                                               │
   │   [ your process here — sees only this namespace's stack ]    │
   └───────────────────────────────────────────────────────────────┘
```

## The seven commands

```bash
# 1. Create the namespace
sudo ip netns add demo

# 2. Create the veth pair (both ends start in the host namespace)
sudo ip link add veth-host type veth peer name veth-demo

# 3. Move one end INTO the namespace
sudo ip link set veth-demo netns demo

# 4. Configure the host side
sudo ip addr add 10.0.0.1/24 dev veth-host
sudo ip link set veth-host up

# 5. Configure the namespace side
sudo ip netns exec demo ip addr add 10.0.0.2/24 dev veth-demo
sudo ip netns exec demo ip link set veth-demo up
sudo ip netns exec demo ip link set lo up

# 6. Give the namespace a default route via the host side
sudo ip netns exec demo ip route add default via 10.0.0.1

# 7. Enable forwarding + NAT on the host so packets reach the internet
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o ens33 -j MASQUERADE
```

Verify:

```bash
sudo ip netns exec demo ping 8.8.8.8
```

## Packet flow: ping 8.8.8.8 from inside demo

```mermaid
sequenceDiagram
    participant App as App in demo ns
    participant DemoRT as demo routing table
    participant VD as eth0 (10.0.0.2)
    participant VH as veth-host (10.0.0.1)
    participant HostRT as host routing table
    participant NAT as iptables NAT
    participant ENS as ens33 (192.168.1.156)
    participant Net as Internet

    App->>DemoRT: send to 8.8.8.8
    DemoRT->>VD: default route → 10.0.0.1
    VD->>VH: frame across veth pair
    Note right of VH: now in host namespace<br/>src=10.0.0.2, dst=8.8.8.8
    VH->>HostRT: which interface for 8.8.8.8?
    HostRT->>NAT: route says ens33
    NAT->>ENS: rewrite src 10.0.0.2 → 192.168.1.156
    ENS->>Net: send frame onto LAN
    Net-->>ENS: reply: src=8.8.8.8, dst=192.168.1.156
    ENS-->>NAT: NAT reverses: dst → 10.0.0.2
    NAT-->>VH: route to 10.0.0.0/24
    VH-->>VD: frame across veth pair
    VD-->>App: deliver to socket
```

## The three responsibilities the host has to take on

Pattern 1 only works because the host is playing **three roles** simultaneously. Strip any one and the connection breaks:

| Role | What it does | Provided by |
|---|---|---|
| **Routing** | "Packets to 10.0.0.0/24 go via veth-host; everything else via ens33" | host's routing table (auto-populated when IPs are added) |
| **IP forwarding** | Allows packets received on one interface to be sent out another | `net.ipv4.ip_forward=1` |
| **NAT (MASQUERADE)** | Rewrites the source IP from `10.0.0.2` to `192.168.1.156` so replies come back to the host | `iptables -t nat ... MASQUERADE` |

The **demo namespace** also needs:

| Need | Provided by |
|---|---|
| A path out | the veth pair |
| Knowing where to send | a default route via `10.0.0.1` |
| A working `lo` | `ip link set lo up` |
| DNS | a `/etc/resolv.conf` inside the namespace (file namespaces are separate — see [below](#dns-and-other-not-network-namespaces)) |

## What breaks if you skip something

| Removed | Symptom |
|---|---|
| `ip route add default via 10.0.0.1` in demo | App in `demo` gets *"Network is unreachable"* — routing table doesn't know where to send the packet |
| `net.ipv4.ip_forward=1` | Packet reaches the host's veth-host, but the host drops it instead of forwarding to `ens33` |
| `iptables MASQUERADE` | Outbound packet leaves with src=`10.0.0.2`; replies are addressed to `10.0.0.2`, which the LAN router doesn't know about; packets vanish |
| `ip link set lo up` in demo | App calling `127.0.0.1` inside demo fails — loopback is down |

## Where Pattern 1 vs a home router differs

The analogy is tight, but a few honest gaps are worth knowing:

1. **veth is point-to-point, Wi-Fi is shared.** A veth pair connects exactly **two** namespaces — more like one Ethernet cable than a Wi-Fi access point. To support many "clients," you need a bridge in the middle (Pattern 2) — that's the "router with built-in switch + Wi-Fi" shape.
2. **No DHCP / DNS by default.** A real router auto-leases IPs and runs a DNS forwarder. In netns Pattern 1 you do all that by hand.
3. **Same kernel.** A home router is a separate machine. The host namespace and the demo namespace share **one kernel** — they're just two views of the same stack. The "wire" between them is a kernel memory copy, not a real signal.
4. **No physical layer.** No radio, no encryption, no signal strength. Packets pop instantly from one end to the other.

## DNS and other "not network" namespaces

A subtle gotcha: **network namespaces only isolate the network stack.** They do *not* isolate the filesystem. So `/etc/resolv.conf` inside `demo` is **the same file** as outside, unless you also enter a different mount namespace.

Docker handles this by bind-mounting a per-container `resolv.conf` inside the container's mount namespace. For raw `ip netns`, the convention is to put a per-namespace resolv.conf at `/etc/netns/demo/resolv.conf`, which `ip netns exec` will bind-mount over `/etc/resolv.conf` automatically:

```bash
sudo mkdir -p /etc/netns/demo
echo 'nameserver 8.8.8.8' | sudo tee /etc/netns/demo/resolv.conf
sudo ip netns exec demo curl https://example.com   # DNS now resolves
```

## Cleanup

```bash
sudo iptables -t nat -D POSTROUTING -s 10.0.0.0/24 -o ens33 -j MASQUERADE
sudo ip link del veth-host           # also removes the peer
sudo ip netns del demo
```

## One-line summary

> Pattern 1 turns the host into a tiny NAT router with one client: a veth pair carries packets between the namespace and the host, the host's routing table picks the upstream interface, IP forwarding lets packets cross, and MASQUERADE rewrites the source IP so replies come home. Add a bridge in the middle and you have Docker. ⚙️
