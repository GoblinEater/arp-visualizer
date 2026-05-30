# ARP Poisoning Visualizer

An interactive, browser-based tool that demonstrates how the Address Resolution Protocol works under normal conditions — and how an attacker exploits its lack of authentication to become a man-in-the-middle.

Built as part of the [Goblin Eaters Cyber Cavern](https://ubritsa.com) cybersecurity study project.

---

## What It Does

The visualizer runs two independent animated walkthroughs:

**Normal Flow** — 5 steps showing legitimate ARP resolution from broadcast request through to established communication, including the switch's flood behavior and ARP cache population.

**ARP Poisoning** — 8 steps showing the full attack sequence: the initial broadcast, Host B's legitimate reply, the attacker's fake reply arriving after it, the cache being silently overwritten, bidirectional poisoning, and the final misdirected data frame relayed through the attacker.

Each step includes:
- Animated packet traversal on a live canvas with color-coded paths
- An Ethernet frame dissector showing the exact fields — with `FALSIFIED` / `CORRECT` badges on poisoned fields in attack mode
- A step explanation panel with label, summary, and technical detail
- A live ARP cache display that updates as the scenario progresses, flipping from green (correct) to red (poisoned) at the moment the cache is overwritten

---

## The Key Concept

ARP operates on **last-write-wins with no authentication**. Any device on the L2 domain can claim any IP-to-MAC mapping, and the most recent reply silently overwrites whatever was cached before it.

Host B does everything right — it replies honestly and on time. It doesn't matter. The attacker replies *after* Host B, and the correct cache entry is gone.

**Host B lost the race.**

---

## Features

- Two modes: Normal Flow / ARP Poisoning (toggle in the header)
- Step-by-step navigation with Prev / Next / Reset controls
- Auto Play mode with 3.4-second cadence
- Each step's animation loops automatically until you advance — no more missing the packet if you blink
- Broadcast correctly modeled: switch floods to all ports *except* the ingress port
- Misdirect step shows the full 4-hop relay: `Host A → Switch` (blue), `Switch → Attacker` (blue), `Attacker → Switch` (red), `Switch → Host B` (red)
- Frame dissector shows real hex values for EtherType, operation codes, and IP/MAC fields
- Zero dependencies — single HTML file, no build step, no framework

---

## Usage

Open `arp-visualizer-v2_12.html` in any modern browser. No server required.

```bash
git clone https://github.com/GoblinEater/arp-poisoning-visualizer.git
cd arp-poisoning-visualizer
open arp-visualizer-v2_12.html   # macOS
# or just double-click the file
```

---

## Network Topology

```
  Host A              Switch              Host B
192.168.1.10   ←——→  [SW]  ←——→   192.168.1.20
AA:BB:CC:11:22:33            DD:EE:FF:44:55:66

                       ↕  (attack mode only)

                    Attacker
                  192.168.1.99
                DE:AD:BE:EF:00:01
```

---

## Attack Sequence (ARP Poisoning Mode)

| Step | Event |
|------|-------|
| 1 | Host A broadcasts ARP request → Switch |
| 2 | Switch floods to all ports — Host B and Attacker both receive it |
| 3 | Host B replies legitimately — correct MAC cached ✓ |
| 4 | Cache holds correct entry — briefly |
| 5 | Attacker replies after Host B — fake MAC, valid frame structure |
| 6 | **Host B lost the race** — cache overwritten with attacker's MAC ✗ |
| 7 | Attacker poisons Host B in the other direction (bidirectional) |
| 8 | Host A's data frames misdirected through the attacker |

---

## Defenses (not modeled, but worth knowing)

- **Dynamic ARP Inspection (DAI)** — switch validates ARP replies against a trusted DHCP snooping table; forged replies are dropped
- **Static ARP entries** — manually pinned IP-to-MAC mappings that cannot be overwritten by incoming replies
- **802.1X port authentication** — limits which devices can participate on the L2 segment at all
- **Encrypted protocols (TLS/HTTPS)** — even with a poisoned cache, an attacker intercepting encrypted traffic cannot read or modify it without a valid certificate

---

## Context

Built for CEH and CySA+ study at Wake Tech Community College. Part of a broader series of cybersecurity protocol visualizers published at [ubritsa.com/tools](https://ubritsa.com/tools).

---

## License

MIT
