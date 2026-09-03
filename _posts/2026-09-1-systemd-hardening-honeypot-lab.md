---
title: "Hardening a Toy Honeypot Service with systemd, Permissions, and journald"
date: 2026-09-01 16:30:00 -0400
categories: [Home Lab, Linux]
tags: [systemd, linux-hardening, honeypot, journald, acl, permissions]
---

## Overview

First lab session working toward an eventual honeynet build. The goal
wasn't the honeypot itself — it was building and *proving* the foundational
Linux controls a honeynet depends on: an unprivileged service account, a
hardened systemd unit, a least-privilege permission model between two
service roles, and centralized logging via journald. The initial pass was
done on a single Ubuntu VM (VirtualBox), with each step manually verified
rather than assumed to work. A second session repeated the systemd hardening
and package-pinning drill on an AlmaLinux VM to directly compare
Debian-family vs RHEL-family behavior, including SELinux vs AppArmor.

## Stage 1: Unprivileged Service Account

Created a dedicated system account for the service instead of running it as
a normal user or root:

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin honeytest
```

- `--system` allocates a UID from the system range, marking this as a
  service identity rather than a person's login.
- `--no-create-home` — the account owns nothing outside what's explicitly
  granted to it.
- `--shell /usr/sbin/nologin` — even with valid credentials, no interactive
  shell is possible.

**Verification, not assumption:** confirmed the restriction actually works
by attempting `sudo su - honeytest` and getting `This account is currently
not available.` An earlier attempt (`su - honeytest` as a non-root user)
gave a false negative — it failed on missing-password auth before ever
reaching the shell check, which was a good reminder that a "denied" result
doesn't always mean it was denied for the *reason* you think.

## Stage 2: Toy Service

A minimal Python HTTP server (`python3 -m http.server`) serving
`/opt/honeypot`, confirmed working manually (as a normal user, foreground)
before ever being handed to systemd — so any later failure could be
isolated to the systemd config, not the service itself.

## Stage 3: Hardening the systemd Unit

Baseline unit file (`/etc/systemd/system/honeytest.service`) written with
**zero hardening directives first**, deliberately, to establish a known-good
starting point before changing anything:

```ini
[Unit]
Description=Toy honeypot HTTP listener
After=network.target

[Service]
Type=simple
User=honeytest
Group=honeytest
ExecStart=/usr/bin/python3 -m http.server 8080 --directory /opt/honeypot
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Baseline score via `systemd-analyze security honeytest.service`:

> **9.2 / UNSAFE**

Hardening directives were added one at a time under `[Service]`, each
followed by `daemon-reload` → `restart` → `status` → functional test →
re-score, so every change could be attributed individually:

| Directive | Purpose | Score after |
|---|---|---|
| `NoNewPrivileges=true` | Blocks the process (and anything it execs) from gaining more privileges than it started with | 9.0 |
| `ProtectSystem=strict` | Mounts the entire filesystem read-only for the service except a few systemd-managed paths | 8.7 |
| `PrivateTmp=true` | Gives the service its own isolated `/tmp`, invisible to other services on the same host | 8.5 |
| `ReadOnlyPaths=/` | Explicit read-only enforcement on `/` — mostly redundant here since `ProtectSystem=strict` already covers it, and not scored by the analyzer at all | 8.5 (no change) |

**Bug encountered and fixed:** `NoNewPrivileges=true` was initially typed
under the `[Install]` section instead of `[Service]`. Systemd directives are
section-scoped by where they appear in the file, not by name — placing it
in the wrong section caused it to be silently ignored (no error at
`daemon-reload` time, just no effect). Confirmed via journald after the
fact: `Unknown key 'NoNewPrivileges' in section [Install]`.

**Key takeaway on `ReadOnlyPaths=`:** the score didn't move when this was
added, but that doesn't mean it did nothing — `systemd-analyze security`
scores a fixed, known checklist of directives, and this one simply isn't on
it. A flat score is a heuristic, not a complete measure of hardening.

## Permissions Exercise: Two-Role Access Model

Goal: a `honeytest` service account that can **write** to shared data
directories, and a separate `honeyadmin` account that can **read** the same
data but not modify it — without giving either account ownership of the
other's files.

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin honeyadmin
sudo groupadd honeypot-data
sudo usermod -aG honeypot-data honeytest
sudo usermod -aG honeypot-data honeyadmin
```

Directory layout:

```bash
sudo mkdir -p /opt/honeypot/{bin,logs,captures}
sudo chown root:honeypot-data /opt/honeypot/bin
sudo chmod 750 /opt/honeypot/bin

sudo chown honeytest:honeypot-data /opt/honeypot/logs /opt/honeypot/captures
sudo chmod 2750 /opt/honeypot/logs /opt/honeypot/captures
```

**Design iteration worth documenting:** the first attempt used `2770`
(group write) so both accounts could write. That's wrong for the intended
model — standard Unix permissions only offer three buckets (owner, group,
other), and *every* member of a group gets identical rights. There's no way
to give one group member write and another only read using `chown`/`chmod`
alone. Switching to `2750`, with `honeytest` as the directory **owner**
(full `rwx` via the owner bucket) and `honeyadmin` only a group **member**
(`r-x` via the group bucket), achieved the intended split — because the
two accounts land in different permission buckets, not because of anything
group-specific.

The leading `2` in `2750`/`2750` is the **setgid bit** on the directory —
any file created inside inherits the directory's group (`honeypot-data`)
rather than the creator's primary group, which matters since `honeytest`
and `honeyadmin` didn't necessarily share the same primary group (one was
created with `useradd`, one with `adduser`, which have different default
behaviors around private-group creation).

Verified with real write/read attempts, not just permission bits:

```bash
sudo -u honeytest touch /opt/honeypot/logs/text-write.txt   # succeeds
sudo -u honeyadmin touch /opt/honeypot/logs/should-fail.txt # Permission denied
sudo -u honeyadmin cat /opt/honeypot/logs/text-write.txt    # succeeds
```

### ACL Detour

Explored POSIX ACLs (`setfacl`/`getfacl`) as the tool that *would* be
needed if a third role required different rights than either the owner or
group bucket allows — e.g., a future `honeyauditor` account with different
access than `honeyadmin`. Not needed for this two-role setup, but useful to
understand:

```bash
sudo setfacl -m u:ubuntu-testbox:r-x /opt/honeypot/logs
getfacl /opt/honeypot/logs
sudo setfacl -x u:ubuntu-testbox /opt/honeypot/logs   # cleanup
```

Notable: the ACL **mask** (`mask::r-x`) acts as a ceiling on all named
user/group ACL entries and persists even after individual entries are
removed — it only disappears if the ACL is stripped entirely (`setfacl -b`).
`ls -ld` shows a trailing `+` on any entry that has an active ACL, as a
signal to check `getfacl` for the full picture.

## journald Practice

Confirmed stdout/stderr from the service flows into journald automatically
with no extra config needed (`Type=simple`, no explicit log redirection):

```bash
sudo journalctl -u honeytest -f      # live-tail while curling from another terminal
sudo journalctl -u honeytest --since today
sudo journalctl -u honeytest --since "16:00" --until "16:30"
sudo journalctl -u honeytest --since today | grep "GET"
```

Structured export for future log-shipping (Phase 5 groundwork):

```bash
sudo journalctl -u honeytest --since today -o json-pretty | head -50
sudo journalctl -u honeytest --since today -o json | tail -5
```

`-o json` emits one compact JSON object per line ("JSON Lines" format) —
this is the actual format a log shipper or script would consume, as opposed
to `json-pretty`'s human-readable formatting. Used `jq` to slice specific
fields out of the compact stream:

```bash
sudo journalctl -u honeytest --since today -o json | tail -5 \
  | jq '{time: .__REALTIME_TIMESTAMP, message: .MESSAGE}'
```

## Debian-family vs RHEL-family Comparison (AlmaLinux)

Repeated the account setup and systemd hardening exactly on an AlmaLinux VM
to compare against the Ubuntu results.

**Service account and shell restriction:** identical behavior. `useradd
--system --no-create-home --shell /usr/sbin/nologin honeytest` produced a
private group automatically (same as Ubuntu's `useradd`, unlike Ubuntu's
`adduser` wrapper, which behaves differently). `sudo su - honeytest` refused
a shell on Alma exactly as it did on Ubuntu.

**System UID range — a real, citable difference:**

| | Ubuntu | AlmaLinux |
|---|---|---|
| `/etc/login.defs` system range | Not declared (`SYS_UID_MIN`/`MAX` absent) | `SYS_UID_MIN=201`, `SYS_UID_MAX=999` |
| Regular-user range | `UID_MIN=1000`, `UID_MAX=60000` | (not checked) |

**systemd hardening — fully portable, including under SELinux enforcing:**
Alma ships SELinux in `Enforcing` mode by default (vs. Ubuntu's AppArmor).
Every directive from the original four-item hardening list was added one at
a time and cross-checked against `ausearch -m avc -ts recent` for AVC
denials at each step — none occurred.

| Directive | Ubuntu score | Alma score | SELinux denial? |
|---|---|---|---|
| (baseline, unhardened) | 9.2 UNSAFE | 9.2 UNSAFE | — |
| `NoNewPrivileges=true` | 9.0 | 9.0 | No |
| `ProtectSystem=strict` | 8.7 | 8.7 | No |
| `PrivateTmp=true` | 8.5 | 8.5 | No |
| `ReadOnlyPaths=/` | 8.5 (unscored) | 8.5 (unscored) | No |

Result: for this specific unit, the hardening directives behaved
identically on both distros, both in exposure score and in absence of
mandatory-access-control interference — a legitimate finding, not just a
"nothing happened" non-result. Worth testing rather than assuming, since a
more complex real service (one that writes logs, binds privileged ports, or
touches non-standard paths) would be a more likely candidate to actually
trip SELinux where this toy server didn't.

**Package pinning — `apt-mark hold` vs `dnf versionlock`:**

- `apt-mark hold` is built into `apt` itself, no extra install required.
- `dnf versionlock` requires installing a separate plugin first:
  `sudo dnf install -y python3-dnf-plugin-versionlock`.
- Both mechanisms were configured and confirmed present in their respective
  listings (`apt-mark showhold` showed `curl`; `dnf versionlock list` showed
  `python3`).
- Neither was exercised against a real pending update — both VMs were
  freshly provisioned with no newer package versions available in their
  repos at test time (`apt list --upgradable` and `dnf check-update` both
  came back empty for the locked packages on their respective boxes). This
  is documented honestly as "lock mechanism configured and verified
  present, not proven to block an actual upgrade," rather than overclaiming
  a full test that didn't actually occur.

## Summary

| Item | Result |
|---|---|
| Starting exposure score (both distros) | 9.2 / UNSAFE |
| Ending exposure score (both distros) | 8.5 / EXPOSED |
| Service account isolation | Verified (`nologin`, no shell obtainable even via root `su`) — identical on both distros |
| Two-role permission split | Verified via live write/read tests, not just `ls -l` |
| Structured logging | Confirmed live capture + JSON export path for future shipping |
| SELinux vs AppArmor impact on this unit | None observed — hardening fully portable |
| Package pinning | Both mechanisms configured/verified present; not exercised against a real available update |

## Next Steps

- Re-test the package pinning drill against a package with an actual
  pending update, to get a real (not just configured) proof of both
  `apt-mark hold` and `dnf versionlock` blocking an upgrade
- Push `systemd-analyze security` further (untouched categories included
  `RestrictNamespaces=`, `SystemCallFilter=~@*` syscall groups, and
  `ProtectHostname=`/`ProtectClock=`)
- Try a service that actually needs to write somewhere (logs, a pidfile, a
  non-standard port) to get a more meaningful SELinux vs AppArmor
  comparison than this toy server provided
- Long-term goal: extend this single-service foundation into a full
  honeynet — multiple isolated honeypot services, network-level isolation
  between them (separate from anything systemd controls), and centralized
  log shipping building on today's journald work
