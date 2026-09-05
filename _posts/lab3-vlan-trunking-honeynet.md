---
title: "Honeynet Lab 3: Trunk vs. Access Port Simulation in VirtualBox"
date: 2026-09-04 15:00:00 -0500
categories: [Homelab, Honeynet]
tags: [networking, vlan, pfsense, virtualbox, 802.1q, dhcp, linux]
---

## Objective

Practice the distinction between **trunk ports** (a single wire carrying multiple
tagged VLANs) and **access ports** (a wire dedicated to a single, untagged VLAN),
entirely in software using VirtualBox — no physical managed switch available.

This lab builds directly on the address plan from Lab 1 and the pfSense VM from
Lab 2:

| Zone | VLAN | Subnet |
|---|---|---|
| Home LAN | 1 | `192.168.1.0/24` |
| Management/Logging | 10 | `10.10.10.0/24` |
| Honeypot | 20 | `10.10.20.0/24` |

## Environment

- **Hypervisor:** VirtualBox
- **Firewall/Router:** pfSense Community Edition (VM)
- **Test clients:** two Ubuntu VMs (`testbox`, `testbox2`)
- **Trunk medium:** a dedicated VirtualBox Internal Network named `trunk_net`,
  attached as a new adapter on pfSense (landed as `em3`/OPT2 in interface
  assignments) and on each test VM

## Topology

pfSense's existing per-zone NICs from Lab 2 (one dedicated Internal Network per
VLAN) already function as **access ports** — one wire, one VLAN, no tagging
required. Lab 3 adds a **trunk port**: a single new NIC (`em3`) carrying both
VLAN 10 and VLAN 20 as 802.1Q-tagged traffic, verified by tagging traffic on
the client side and confirming pfSense correctly demultiplexes it.

## Build steps

1. Added a fourth NIC to the pfSense VM, set to Internal Network `trunk_net`.
2. In pfSense: **Interfaces → Assignments → VLANs**, created VLAN tag 10
   (description: `management`) and VLAN tag 20 (description: `Honeypot`), both
   with parent interface `em3`.
3. Assigned the VLAN interfaces as `OPT3` (VLAN 10) and `OPT4` (VLAN 20), each
   given a static IP (`10.10.10.1/24`, `10.10.20.1/24`) and enabled.
4. Configured a DHCP server scope on each new interface under
   **Services → DHCP Server**, matching the Lab 1 subnet plan.
5. On each test VM, attached a second NIC to `trunk_net` and created a tagged
   VLAN sub-interface with `nmcli`:
   ```
   sudo nmcli connection add type vlan con-name vlan20 ifname enp0s9.20 dev enp0s9 id 20
   sudo nmcli connection up vlan20
   ```
6. Verified each VM received a lease scoped to its own VLAN only.

## Issues encountered and resolutions

Documenting these honestly, since the debugging was most of the actual
learning in this lab.

**1. VLANs initially tagged on the wrong parent interfaces.**
The first attempt tagged VLAN 10 onto `em1` (the existing LAN NIC) and VLAN 20
onto `em2` (an existing OPT1 NIC) — two different parent interfaces, not a
trunk at all. Corrected by re-pointing both VLAN tags to the same dedicated
`em3` parent, which is the actual precondition for a trunk to exist.

**2. Promiscuous Mode was not enabled on `trunk_net` adapters.**
VirtualBox's internal switch can silently drop frames that appear to not
"belong" to a NIC's own MAC — which breaks VLAN tagging by default. Fixed by
setting Promiscuous Mode to **Allow All** on the trunk-connected adapter on
both pfSense and each test VM.

**3. A mistyped `ip link add` command created an orphaned "ghost" VLAN
device.** An early attempt (`epn0s9.20` instead of `enp0s9.20`) left a stray
kernel-level VLAN interface with tag 20 already bound to the parent NIC.
Because only one interface per VLAN ID is allowed per parent, this caused
confusing, inconsistent "device already exists" / "device does not exist"
errors on later attempts — `ip link show` didn't surface the ghost device
under the name being searched for. Resolved by running `ip -d link show type
vlan` (which lists all VLAN devices regardless of name) to find and delete the
actual stray interface.

**4. `dhclient` is not installed by default on current Ubuntu.** Newer Ubuntu
releases dropped ISC DHCP client from the base install in favor of
NetworkManager-integrated DHCP. Installed via `apt install isc-dhcp-client`,
though ultimately `nmcli connection up` (which handles DHCP internally) proved
more reliable than manually invoking `dhclient` against a NetworkManager-owned
interface.

**5. DHCP server showed as "enabled" in the config form but never issued a
lease.** `Status → Interfaces` confirmed tagged traffic was reaching pfSense
(RX packet count rising), and `Status → Services` showed `dhcpd` as running —
but zero leases were ever issued, and the DHCP log contained no `dhcpd`
server-side entries at all, only unrelated WAN client (`dhclient`/`dhcp6c`)
noise. Resolved by explicitly restarting the `dhcpd` service under
**Status → Services** — the saved config had not been picked up by the
running daemon.

**6. A static IP on an unrelated management NIC (`enp0s8`) reverted to a
stuck DHCP negotiation.** Lost web GUI access to pfSense mid-lab because the
Home LAN-facing NIC's NetworkManager profile was showing `ipv4.method:
manual` with the correct address, yet `ip a` showed no address applied at
all. Worked around with a manual `ip addr add` for the remainder of the
session; flagged as a known non-persistent workaround, not a real fix — the
NetworkManager profile likely needs to be deleted and recreated cleanly in a
future session.

## Verification results

| Test | Result |
|---|---|
| VLAN 20 tagged traffic reaches pfSense on `em3` | ✅ confirmed via `Status → Interfaces` RX counters |
| `testbox` VM (VLAN 20 tag) receives lease | ✅ `10.10.20.10/24` |
| `testbox2` VM (VLAN 10 tag) receives lease | ✅ `10.10.10.10/24` |
| Both VLANs correctly separated on a single shared wire | ✅ confirmed — two VMs, one `trunk_net`, two distinct subnets |

Access-port comparison (plugging a VM directly into an existing Lab 2
per-zone network with no tagging) was scoped as an optional follow-up and not
executed in this session.

## Key takeaways

- A trunk requires **one shared parent interface**; tagging VLANs onto
  separate parent NICs is not a trunk, regardless of how the VLAN config
  looks in isolation.
- VirtualBox's Internal Network switch needs **Promiscuous Mode: Allow All**
  on every adapter carrying tagged traffic — this is not obvious from the
  VLAN configuration alone and fails silently otherwise.
- `ip link show <name>` can miss orphaned interfaces created under a typo'd
  name; `ip -d link show type vlan` is the more reliable way to audit what
  VLAN devices actually exist on a system.
- A service showing "running" in a status panel does not guarantee it has
  picked up the latest configuration — restarting the service after a config
  change is sometimes necessary even when the save appeared to succeed.
- NetworkManager-reported success (`connection successfully activated`) is
  not always reflected in the kernel's actual interface state — worth
  cross-checking with `ip a` rather than trusting nmcli's return message
  alone.
