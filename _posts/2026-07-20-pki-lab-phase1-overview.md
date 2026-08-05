---
title: "Two-Tier PKI Home Lab — Phase 1: Design & Build"
date: 2026-07-20 09:00:00 -0400
categories: [Projects, PKI Lab]
tags: [pki, active-directory, windows-server, virtualbox, certificate-services]
img_path: /assets/img/posts/pki-lab-overview
---

> This is the project overview for a two-tier PKI home lab built in
> VirtualBox. Individual machine build logs are linked below and marked
> **v1** — later rebuilds or reconfigurations of a given machine will get
> their own versioned post rather than overwriting this one.
{: .prompt-info }

## Goal

Build a realistic two-tier Microsoft PKI hierarchy from scratch — an
offline Root CA, an online Enterprise Subordinate/Issuing CA, and a
domain controller to tie it all together — the way it would actually be
deployed in a production Active Directory environment, not a flattened
single-CA shortcut.

**Environment:** VirtualBox, isolated Internal Network (`LABNET`), Windows
Server 2022 Standard Evaluation across all server roles, domain `lab.local`.

## Topology

| VM | Role | Domain-joined? | Static IP | DNS |
|---|---|---|---|---|
| DC01 | Domain Controller + DNS | N/A (is the DC) | 192.168.10.1 | 127.0.0.1 |
| CA01 | Offline Root CA | No — workgroup | 192.168.10.2 | (blank) |
| CA02 | Issuing / Subordinate CA | Yes | 192.168.10.3 | 192.168.10.1 |
| CLIENT | Test client (Win10/11) | Planned | TBD | 192.168.10.1 |

All server VMs were cloned from a single common base image (patched,
Guest Additions installed) *before* any role-specific configuration —
done specifically to avoid duplicate-SID issues that can come from
cloning after domain promotion or CA configuration. Networking is
deliberately isolated: a VirtualBox Internal Network only, no Bridged
adapter, keeping the DC and CA machines fully off the home LAN.

## Why a two-tier hierarchy

A single flat CA is simpler to stand up, but it doesn't reflect how PKI
is actually deployed at any real scale, and it has one specific weakness
this design solves: if the Root CA is online and gets compromised, the
entire trust chain is compromised with it. Keeping the Root CA (**CA01**)
offline — powered off except when explicitly signing a subordinate's
certificate — means its private key is never exposed to network-based
attack. The Issuing CA (**CA02**) handles all day-to-day certificate
issuance and stays online and domain-joined, since that's the box that
actually needs to be reachable by clients.

## Build logs (per machine)

Each machine has its own detailed build/troubleshooting log:

1. [DC01 — Domain Controller (v1)]({% post_url 2026-07-21-pki-lab-dc01-v1 %})
2. [CA01 — Offline Root CA (v1)]({% post_url 2026-07-22-pki-lab-ca01-v1 %})
3. [CA02 — Issuing/Subordinate CA (v1)]({% post_url 2026-07-23-pki-lab-ca02-v1 %})

## Phase 1 status

- ✅ DC01 — domain controller for `lab.local`, healthy, DNS verified
- ✅ CA01 — offline Root CA, signed CA02's subordinate certificate, CDP/AIA configured, verified via pkiview
- ✅ CA02 — online Enterprise Issuing CA, Certificate Services running, `pkiview.msc` shows all green
- ⏳ CA01 confirmed powered off / disconnected to restore its offline posture

## Phase 2 (planned)

Phase 1 covers the core CA hierarchy. Phase 2 will cover:

- **Group Policy publishing** on DC01 — pushing the Root CA cert to
  Trusted Root Certification Authorities and the Subordinate CA cert to
  Intermediate Certification Authorities domain-wide
- **Building the CLIENT VM**, joining it to `lab.local`, and confirming
  both CA certs land correctly via `gpupdate /force` and `certmgr.msc`
- **End-to-end enrollment test** — requesting a certificate from the
  client via `certmgr.msc` or the `certsrv` web enrollment page, to prove
  the entire chain works from a client's perspective, not just on the
  CAs themselves

I'll post Phase 2 as its own writeup once the client build and enrollment
test are done.
