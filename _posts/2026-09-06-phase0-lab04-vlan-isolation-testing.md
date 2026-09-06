---
title: "Phase 0 – Lab 04: Proving VLAN Isolation, Don't Assume It"
date: 2026-09-06 02:00:00 -0400
categories: [Homelab, Phase 0 - Foundation]
tags: [pfsense, vlan, virtualbox, networking, honeynet, firewall, nat]
---

## Objective

Lab 4 picks up where Lab 3 left off: the 802.1Q trunk carrying VLAN 10 (Management) and VLAN 20 (Honeypot) to pfSense was proven functional, but functional trunking isn't the same as functional *isolation*. This lab's goal, straight from the curriculum:

> Prove isolation, don't assume it. From a VM on the honeypot VLAN, run ping/traceroute to a host on Management and to Home LAN — confirm they're blocked at the firewall, not just "not configured." Then confirm the honeypot VLAN *can* reach the internet.

That last sentence is the whole point of a honeypot: contained from internal networks, but alive to the outside world. This lab turned out to be less about running four ping commands and more about learning to distrust every "it's not working" result until I could prove *why*.

## Environment

- **pfSense CE** VM — 4 adapters: WAN (bridged to host Wi-Fi), `homelan` (Internal Network), `labnet` (unused/legacy), `trunk_net` (Internal Network, carries tagged VLAN 10/20)
- **Ubuntu testbox** — single adapter on `trunk_net`, VLAN 20 sub-interface (`enp0s9.20`), representing a Honeypot VLAN host
- **Ubuntu testbox2** — single adapter on `trunk_net`, VLAN 10 sub-interface (`enp0s8.10`), representing a live Management VLAN target
- **Alma testbox** — attached to `homelan` mid-lab specifically to reach the pfSense WebGUI once testbox's own path to it broke (see below)

## Test Plan

From testbox (10.10.20.10, VLAN 20):

1. `ping`/`traceroute` to 10.10.10.10 (Management) — expect blocked
2. `ping` to a Home LAN host (192.168.1.x) — expect blocked
3. `ping` to 8.8.8.8 — expect success (internet reachability is what makes this a viable honeypot)

Watch pfSense's firewall logs live during 1 and 2 to confirm blocks are firewall decisions, not silent failures from a misconfigured host.

## Pre-Flight Cleanup

Before running anything, both test VMs needed sanitizing:

- **testbox** originally had a second adapter (`homelan`) directly bridging it to the flat Home LAN — an untagged Layer 2 shortcut that would have completely invalidated any isolation test run from that box. Detached in VirtualBox settings, not just brought down in-guest.
- Both boxes had NAT adapters (Adapter 1) that needed disabling — VirtualBox's own NAT engine gives every VM an independent path to the internet, which would produce false positives on the pfSense-routed internet test.

## The Debugging Journey

This is where the lab actually happened. Three separate, unrelated misconfigurations stacked on top of each other, and untangling them was the real exercise in "prove it, don't assume it."

### Issue 1: A ghost route survived a clean-looking interface

First ping attempt to 10.10.10.10 came back with `Destination Host Unreachable` — but the source address in the error was suspicious. `ip route` on testbox revealed **two default routes**:

```
default via 192.168.1.1 dev enp0s9 proto static metric 20100
default via 10.10.20.1 dev enp0s9.20 proto dhcp metric 20400
```

A leftover static route from before the trunk was even configured was winning on metric, sending everything toward a dead gateway before it ever reached pfSense. I deleted it and the stale IP address at the runtime level with `ip route del` / `ip addr del` — which worked immediately, but didn't survive a later reboot (see Issue 3).

### Issue 2: pfSense had nothing to say about VLAN 20

With the routing cleaned up, `tcpdump -ni em3.20 icmp` on pfSense confirmed the ICMP echo requests were arriving perfectly — tagged correctly, no drops, no L2 issues. But pfSense sent back nothing. The filter log made it obvious:

```
pfSense filterlog[24493]: 1,,,1000000103,,,match,block,in,4,0x0,,64,10851,0,DF,6,10.10.20.10,10.10.20.1,...
```

The default deny rule was catching everything. Unlike LAN, a freshly created OPT interface in pfSense ships with **zero pass rules** — Lab 3 only proved DHCP worked (which bypasses firewall rules entirely), not that any actual traffic could pass.

Fix: added a single rule on the OPT4 (VLAN 20) interface —

- **Pass**, Source: OPT4 subnets, Destination: **Address/Alias, Invert Match, `RFC1918`**

Since pfSense's version here didn't ship a built-in "RFC 1918" alias, I built one manually (`Firewall → Aliases`) covering `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`. The inverted match means: pass to anywhere that *isn't* private address space. One rule gives the honeypot VLAN internet egress while leaving inter-VLAN traffic to fall through to the default block — exactly the isolation model the lab wants.

### Issue 3: The ghost route came back, and WAN never had an address

Reaching the pfSense GUI itself became a side quest — testbox had been my path to it, and once its network access broke, so did GUI access. Solved by attaching Alma testbox to the `homelan` segment as a dedicated management/GUI-access host.

After adding the firewall rule, internet still failed — and the stale 192.168.1.50/24 address was back on testbox after a reboot. The earlier `ip route del` fix had been runtime-only; NetworkManager's **"Wired connection 1"** profile still had the address baked in as persistent config and reapplied it on boot. Root cause fixed properly this time:

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.addresses "" ipv4.gateway "" ipv4.dns "" ipv4.method disabled ipv6.method disabled
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

But internet still didn't work — and this time the "Destination Host Unreachable" was coming from pfSense's own gateway (10.10.20.1), not from testbox locally. That's a meaningfully different signal: the packet was reaching pfSense and pfSense couldn't get it further out. `Firewall → NAT → Outbound` showed Automatic mode selected but an **empty automatic rules table** — no NAT rule existed for any interface, not just OPT4.

Root cause: pfSense's **WAN interface had no IPv4 address at all**. `Status → Interfaces` showed WAN "up" with zero packets and no DHCP lease — it had apparently never pulled one, likely a casualty of bridging to the host's Wi-Fi adapter rather than a wired connection. Explicitly forcing the WAN interface to DHCP and renewing resolved it immediately; `Status → Gateways` flipped from "Pending" to "Online" with a real lease from the home router (192.168.0.1).

## Final Results

| Test | Result | Interpretation |
|---|---|---|
| Ping 10.10.20.1 (own gateway) | Blocked | pfSense's default anti-lockout ICMP policy on its own interface address — expected pfSense behavior, not a lab failure |
| Ping 10.10.10.10 (Management VLAN) | Blocked, silent — confirmed via filter log | Isolation working as designed |
| Ping 192.168.1.x (Home LAN) | Blocked, silent | Isolation working as designed |
| Ping 8.8.8.8 (Internet) | **0% loss, ~24-32ms RTT** | Honeypot VLAN has real, pfSense-routed internet egress |

Isolation confirmed at the firewall, not assumed from silence. Internet reachability confirmed as actually routed through pfSense's NAT, not a VirtualBox shortcut.

## Lessons Learned

- **Silence isn't proof.** The first ping "failure" to 10.10.10.10 looked identical whether pfSense blocked it or the packet never left the host. Only a filter-log entry or a tcpdump on the far side actually proves which one happened.
- **Runtime fixes don't survive reboots.** `ip route del` / `ip addr del` cleared the symptom but not the cause. The actual fix had to happen in the persistent NetworkManager connection profile.
- **Fresh interfaces in pfSense default to deny.** LAN's permissive default rule is a special case, not the pattern — every new OPT interface needs explicit pass rules.
- **"Automatic" isn't magic.** Automatic Outbound NAT rule generation silently produces nothing if the WAN gateway itself was never actually established — worth checking `Status → Gateways` (not just the System → Routing config page, which shows intended config, not live state) early rather than assuming NAT is the problem.
- **Bridging to Wi-Fi adapters in VirtualBox is a soft spot.** Confirmed again here, separate from the promiscuous-mode gotcha already logged in Lab 3 — Wi-Fi-bridged adapters can silently fail to acquire a DHCP lease at all.

## What's Next

Lab 5 (NAT lab) is a natural continuation of what came up organically here — comparing NAT'd vs. directly-exposed inbound behavior is now something I've already partially observed from the outbound side while chasing the WAN DHCP issue.
