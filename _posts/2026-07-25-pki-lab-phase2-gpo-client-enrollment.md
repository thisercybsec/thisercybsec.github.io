---
title: "Two-Tier PKI Home Lab — Phase 2: Group Policy Publishing & Client Enrollment"
date: 2026-07-25 09:00:00 -0400
categories: [Projects, PKI Lab]
tags: [pki, group-policy, active-directory, kerberos, windows-11, troubleshooting]
img_path: /assets/img/posts/pki-lab-phase2
---

> Part of the [Two-Tier PKI Home Lab]({% post_url 2026-07-20-pki-lab-phase1-overview %})
> project. This closes out the build: publishing trust domain-wide via
> Group Policy, standing up a test client, and proving end-to-end
> certificate enrollment actually works — not just that the CAs trust
> each other.
{: .prompt-info }

[Previous: CA02 — Issuing/Subordinate CA]({% post_url 2026-07-23-pki-lab-ca02-v1 %})

## Goal

Phase 1 proved the CA hierarchy itself was healthy — `pkiview.msc`
all-green on CA02. That's necessary but not sufficient: a PKI hierarchy
that only trusts itself isn't useful. Phase 2 proves the chain actually
works from a domain client's perspective — cert stores populated
correctly via Group Policy, and a real certificate successfully
requested and issued.

## Step 1 — Group Policy: Publishing Trust to Domain Clients

1. Copied both cert files (`LabRootCA.cer`, `CA02-Subordinate.cer`) from
   CA02 to DC01 (`C:\Certs`) — straightforward since both machines are
   domain-joined and on the same internal network.
2. On DC01: Group Policy Management → right-click `lab.local` → **Create
   a GPO in this domain, and Link it here...** → named `PKI-Trust-GPO`.
3. Edited the GPO → **Computer Configuration → Policies → Windows
   Settings → Security Settings → Public Key Policies**.
4. Right-click **Trusted Root Certification Authorities** → All Tasks →
   Import → `LabRootCA.cer`.
5. Right-click **Intermediate Certification Authorities** → All Tasks →
   Import → `CA02-Subordinate.cer`.
6. Restarted Certificate Services on CA02 to apply.

No troubleshooting needed here — both imports succeeded cleanly on the
first attempt.

## Step 2 — Sales1: Test Client Build

1. New VM, `Sales1` — Windows 11 (initially selected Home during Setup,
   corrected to **Windows 11 Pro**, since Home can't join an AD domain).
2. Adapter set to the same internal network as the rest of the lab.
3. During OOBE, bypassed the Microsoft account sign-in flow via
   **Sign-in options → Domain join instead** to create a local account
   first.
4. Static IP: `192.168.10.10` / `255.255.255.0`, DNS `192.168.10.1`.
5. Renamed to `Sales1`, joined to `lab.local` via `sysdm.cpl` using
   `lab\Administrator` credentials, restarted.
6. Ran `gpupdate /force` to pull the GPO-distributed trust certs.

```powershell
gpupdate /force
```

7. Verified via `certmgr.msc`: **Lab Root CA** present under Trusted
   Root Certification Authorities, **Lab Issuing CA** present under
   Intermediate Certification Authorities — GPO delivery confirmed
   working correctly.

## Step 3 — Certificate Enrollment (the real test)

Personal → All Tasks → Request New Certificate → Active Directory
Enrollment Policy → **User** template → Enroll.

**Result: Succeeded.** Certificate issued by Lab Issuing CA, confirmed
present under Personal → Certificates. This is the point the whole
project was actually building toward — proof the chain works
end-to-end, not just that the CAs trust each other in isolation.

## Troubleshooting

This phase had more friction than the entire CA build combined — worth
documenting in detail since the fixes weren't obvious from the
surface-level errors.

### Black screen on first VM boot

VirtualBox 7.2.6 has a known bug affecting Windows 11 EFI guests that
causes a black screen on boot. Confirmed via research this was a
version-specific issue, not something wrong with my config.

**Fix:** increased vCPU count from 2 to 3 in VM Settings → System →
Processor. Resolved it completely — consistent with other reports of
the same workaround for this VirtualBox version.

### Looping back to Windows Setup language screen

After the CPU fix, the VM booted into the **EFI firmware Boot Manager**
menu instead of Windows Setup, looping back to the language-select
screen. Fix: entered Boot Manager from the EFI menu and explicitly
selected the UEFI CD-ROM/optical entry to force it to load Setup from
the attached ISO.

### `RPC_S_SERVER_UNAVAILABLE` during certificate enrollment

The one that actually cost real troubleshooting time. Enrollment failed
with:

```
0x800706ba
RPC_S_SERVER_UNAVAILABLE
CA02.lab.local\Lab-Issuing-CA is unreachable
```

...despite `ping` and `nslookup` from Sales1 to CA02 both succeeding.
Systematically ruled out:

- CA02 service state — confirmed running
- Windows Firewall — confirmed off on all three profiles (Domain/
  Private/Public) on CA02
- RPC / RPC Endpoint Mapper services — confirmed running on CA02

Also noticed CA02's network adapter was misclassified as "Public/
Unidentified network" instead of "Domain" — a common VirtualBox
Internal Network quirk — but this turned out to be a red herring, since
firewall was already fully disabled regardless of profile.

**Root cause, found almost by accident:** while attempting to adjust CA
security permissions (Authenticated Users → Read/Request Certificates)
on CA02, the *real* underlying error surfaced:

```
0x80070576
WIN32: 1398 ERROR_TIME_SKEW
```

A clock/date mismatch between CA02 and DC01 was breaking Kerberos
authentication — which the RPC/DCOM-based certificate enrollment
process depends on. The original `RPC_S_SERVER_UNAVAILABLE` error was
misleading by design: it looked like a connectivity problem, but the
actual failure was an authentication problem one layer down.

```powershell
w32tm /resync /force
```

This initially failed with *"the computer did not resync because no
time data was available"* — CA02 had no reachable, configured time
source at all. Fixed by configuring DC01 as the authoritative domain
time source:

```powershell
w32tm /config /manualpeerlist:"time.windows.com" /syncfromflags:manual /reliable:yes /update
```

...followed by a `w32time` service restart and forced resync. The step
that actually resolved the enrollment failure, though, was manually
triggering **Date & Time settings → Sync now** on CA02 directly, rather
than relying solely on the `w32tm` command-line resync. Certificate
enrollment succeeded immediately after the clocks corrected.

**Lesson worth remembering:** Kerberos/RPC failures that present as
connectivity problems (`RPC_S_SERVER_UNAVAILABLE`, DCOM errors) are
often actually time skew between domain machines. Check `w32tm` and
system clocks before assuming firewall, DNS, or permissions — the
surface-level error pointed toward networking and cost real time before
the actual cause turned up.

## Known Open Issue

After the manual "Sync now" fix, CA02's clock drifted back to an
incorrect time shortly afterward. Suspected cause: VirtualBox Guest
Additions' default host-time synchronization (guest clock follows host
clock) fighting against the domain's Kerberos time hierarchy, which
expects all domain members to sync from the PDC emulator instead.

**Not yet fixed** — flagged as a follow-up, since recurring drift will
eventually break Kerberos-dependent operations again if left alone.
Planned fix, not yet applied (VM powered off, run from the host):

```powershell
VBoxManage setextradata "<VMName>" "VBoxInternal/Devices/VMMDev/0/Config/GetHostTimeDisabled" 1
```

Run per-VM for DC01, CA02, and Sales1.

## Final Project Status

- ✅ DC01 — domain controller, healthy, DNS verified
- ✅ CA01 — offline Root CA, signed CA02's cert, taken offline after signing
- ✅ CA02 — online Issuing CA, `pkiview.msc` all green
- ✅ GPO — `PKI-Trust-GPO` created and linked, both certs imported into the correct stores
- ✅ Sales1 — Windows 11 Pro client, domain-joined, trust chain confirmed via `certmgr.msc`, **successfully enrolled a User certificate signed by Lab Issuing CA**

**The core objective — build and validate a realistic two-tier PKI
hierarchy in a home lab, end to end — is complete.**

## Possible Follow-Ons

1. **Fix the recurring CA02 time-skew issue** (above) — highest priority, since it could silently break other domain operations later
2. **Auto-enrollment via GPO** — Computer/User certificate auto-enrollment instead of manual `certmgr.msc` requests
3. **Custom certificate templates** — practice template permissions/publishing beyond the default User template
4. **CRL/OCSP monitoring** — track when CA01 needs to come back online to reissue its CRL before the current one expires
5. **A second issuing CA** — practice multi-CA redundancy/segmentation
6. **Actually consume the PKI** — IPsec or 802.1X using certs issued from this hierarchy, rather than just validating enrollment
