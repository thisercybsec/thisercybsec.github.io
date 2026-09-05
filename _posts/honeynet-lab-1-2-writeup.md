---
title: "Honeynet Labs 1–2: Network Planning & pfSense Setup"
date: 2026-09-04
categories: [Homelab, Networking, Security]
tags: [honeynet, pfsense, vlans, network-segmentation, freebsd]
---

# Honeynet Labs 1–2: Network Planning & pfSense Setup

## Lab 1: Network Address Planning

### Objective
Design and document the three-zone network architecture for the honeynet lab, ensuring no overlap and room for future expansion.

### Design

**Three isolated zones, carved from RFC1918 private address space:**

| Zone | CIDR | VLAN ID | Purpose | Trust Level |
|---|---|---|---|---|
| Home LAN | `192.168.1.0/24` | VLAN 1 | Personal devices, management access | Trusted |
| Management/Logging | `10.10.10.0/24` | VLAN 10 | pfSense, log collectors, monitoring | High trust, isolated |
| Honeypot | `10.10.20.0/24` | VLAN 20 | Vulnerable VMs, attack surface | Untrusted, contained |

**Supernet strategy:**
- Management and Honeypot zones carved from `10.10.0.0/16`
- Allows for future expansion: `10.10.30.0/24` (DMZ), `10.10.40.0/24` (IoT), etc.
- Home LAN on separate `192.168.x.x` range, untouched by lab infrastructure

**Key insight:** No overlap, clear semantic separation, room to grow without redesign.

### Deliverable
Visual network diagram showing pfSense as central router with three zones, each connected to a VLAN interface.

---

## Lab 2: pfSense VM Setup & Configuration

### Objective
Stand up a pfSense Community Edition 2.9.0 firewall VM with three network adapters, establish connectivity to the management network, and verify web GUI access.

### Steps Completed

#### 1. Download & Verify Installer
- Downloaded `netgate-installer-v1.2-RELEASE-amd64.iso.gz` from Netgate store
- Verified SHA-256 checksum against email confirmation
- **Learning point:** Netgate now uses a centralized Installer model (small bootstrap) that fetches pfSense packages live during install, rather than shipping full ISOs

#### 2. VM Creation
- **OS Type:** BSD / FreeBSD (64-bit) — critical for CPU long-mode support
- **vCPU:** 2 (sufficient for routing + future IDS packages)
- **RAM:** 2GB initial (upgraded to 4GB for headroom with Suricata later)
- **Disk:** 20GB VDI, dynamically allocated
- **I/O APIC:** Enabled in BIOS settings (necessary for multi-NIC stability)

#### 3. Network Adapter Configuration
Three Intel PRO/1000 MT emulated NICs (changed from default PCnet-FAST III):

| Adapter | VirtualBox Config | Purpose | pfSense Name |
|---|---|---|---|
| 1 | NAT | WAN, internet access | em0 |
| 2 | Internal Network: `homelan` | Home LAN management | em1 |
| 3 | Internal Network: `labnet` | Future trunk for VLAN tagging | em2 |

**Issue discovered:** Initial OS type set to "Other/Unknown" instead of BSD (64-bit). This disabled VT-x exposure to the guest, causing "CPU doesn't support long mode" error during boot. Fixed by:
1. Changing VM OS type to "BSD / FreeBSD (64-bit)"
2. Enabling "Enable PAE/NX" in System → Processor
3. Confirming "Enable VT-x/AMD-V" in System → Acceleration

#### 4. Installation
- Booted from mounted ISO via Netgate Installer
- Selected **Install CE** when prompted (vs. Plus)
- Auto-detected WAN interface (em0, picked up DHCP: `192.168.0.59/24`)
- Declined VLAN setup at console (deferred to web GUI for cleaner config)

**Installer network detection limitation:** Internal Network adapters (em1, em2) not recognized by console auto-detection tool — they require either manual shell config or persistent config file edits.

#### 5. LAN Interface Setup
**Challenge:** Console tool couldn't see em1/em2 adapters. Solved by:
1. Manually configuring em1 at shell: `ifconfig em1 192.168.1.1 netmask 255.255.255.0 up`
2. Editing `/conf/config.xml` to persist the `<lan>` block across reboots:
```xml
<lan>
  <if>em1</if>
  <ipaddr>192.168.1.1</ipaddr>
  <subnet>24</subnet>
  <descr>LAN</descr>
  <enable>1</enable>
</lan>
```
3. Rebooting to apply persistent config

#### 6. Ubuntu Management Access
- Added second NIC to Ubuntu testbox, configured first as homelan (Internal Network)
- Set static IP: `192.168.1.10/24`
- Verified web GUI access: `https://192.168.1.1`
- Confirmed default credentials (`admin / pfsense`) — changed immediately

**Issue resolved:** Adapters not connected/showing as DOWN — confirmed "Cable connected" checkbox was enabled in VirtualBox adapter settings.

### Technical Decisions & Trade-offs

**NAT vs. Bridged for WAN:**
- Chose NAT for simplicity and isolation during lab setup
- Trade-off: Port forwarding into the honeypot requires additional config; can upgrade to Bridged later if needed for direct access

**Config file editing vs. console tool:**
- Console tool (`Assign Interfaces`) only detected em0, not Internal Network adapters
- XML editing was faster than repeatedly cycling through troubleshooting
- **Lesson:** FreeBSD/pfSense sometimes needs direct config for non-physical networks; don't rely on interactive tools for virtual labs

**Intel PRO/1000 MT over PCnet-FAST III:**
- Changed all three adapters to match NIC type for stability
- PCnet-FAST III is older emulation; Intel drivers in FreeBSD more mature
- Eliminated cryptic interface detection failures

### Current State

✓ pfSense 2.9.0 running, fully booted  
✓ em0 (WAN) has internet via NAT  
✓ em1 (LAN) configured at `192.168.1.1/24`, persistent across reboots  
✓ em2 (OPT1/trunk) present but not yet configured  
✓ Web GUI accessible and responsive  
✓ Ubuntu management VM on same homelan segment, can reach GUI  

### Next Steps (Lab 3)

1. Log into web GUI (https://192.168.1.1)
2. Change default admin password
3. Configure VLAN interfaces: 
   - VLAN 10 on em2 (Management/Logging)
   - VLAN 20 on em2 (Honeypot)
4. Assign static IPs to each VLAN
5. Create firewall rules to enforce segmentation and isolation

### Key Learnings

- **Netgate Installer model** reduces download size but requires real-time internet during install
- **Console interface tools** in pfSense have limitations with virtual-only adapters; XML config editing is more reliable for lab setups
- **NIC emulation consistency** matters — mixing adapter types causes driver mismatches in FreeBSD
- **Internal Network naming** is case-sensitive and must match exactly across VMs; VirtualBox auto-names them sometimes, worth verifying
- **Cable connected** checkbox in VirtualBox must be checked per-adapter; easy to miss and causes silent failures

---

## Files & References

- **Network diagram:** `honeynet-lab-1-address-plan.svg`
- **pfSense VM snapshot:** Saved pre-VLAN config (rollback point for Lab 3 iterations)
- **Ubuntu testbox:** Second adapter (NAT) added for future internet access if needed

## Time & Effort

- **Lab 1 (Planning):** ~30 min (address planning, diagram)
- **Lab 2 (Setup):** ~3 hours (installer download, VM creation, troubleshooting adapter detection, config file editing, validation)
- **Blockers:** OS type setting, adapter visibility in console, network persistence — all resolved with direct config

---

**Date completed:** September 4, 2026  
**Lab series:** Honeynet Fundamentals  
**Next:** Lab 3 — VLAN Interface Creation & Firewall Rules
