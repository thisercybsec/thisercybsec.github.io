---
title: "Phase 0 - Week 3 - Lab 03"
date: 2026-09-14 00:00:00 -0500
categories: [Homelab, Honeynet]
tags: [docker, honeypot, opencanary, cowrie, virtualbox, linux]
---

## Objective

Build the "layered model" that previews the T-Pot architecture pattern: a single dedicated honeypot host, isolated from the management network, running multiple Dockerized honeypots side by side. This lab reused an existing VM rather than provisioning a fresh one, and added a second honeypot (OpenCanary) alongside the Cowrie SSH honeypot already running on the box from an earlier lab.

## Environment

- **Host VM:** `testbox3`, Ubuntu Server, already sitting on the Honeypot VLAN (`10.10.20.0/24`) from earlier network segmentation work — confirmed via `ip a` before starting, so no re-attachment to pfSense was needed.
- **Existing container:** Cowrie SSH honeypot, running via Docker on ports 2222/2223 from a prior lab.
- **New container:** OpenCanary, built from source.

## Building the OpenCanary image

Cloned the upstream repo and found two Dockerfiles sitting side by side: `Dockerfile.latest` and `Dockerfile.stable`. These aren't just naming variants — they build the image two fundamentally different ways:

- `Dockerfile.stable` installs the last **published PyPI release** (`uv pip install --system "opencanary[required-for-snmp]"`), ignoring the local clone entirely.
- `Dockerfile.latest` builds from the **actual source in the clone** (`uv sync --frozen --no-dev --extra required-for-snmp`), which is the current git HEAD.

Went with `Dockerfile.latest` to stay consistent with the "build from source so I know what's running" approach already used for Cowrie.

### Detour: first build failed silently

```
docker build -f Dockerfile.latest -t opencanary:local .
```

The first attempt returned `[+] Building 10.4s (3/3) FINISHED` — a suspiciously short step count for a build that should pull a base image, install system deps, and sync a Python project. Buried in the output was the real story:

```
CANCELED [internal] load metadata for docker.io/astral/uv:0.11.7
ERROR    [internal] load metadata for docker.io/library/python:3.10-bookworm
failed to fetch anonymous token: ... net/http: TLS handshake timeout
```

The build had actually failed to reach Docker Hub, not succeeded. A `docker images` check confirmed `opencanary:local` was nowhere to be found. Also uncovered along the way: `docker build` was being run without `sudo`, hitting `permission denied while trying to connect to the docker API` — the user account isn't in the `docker` group, so every Docker command needs an explicit `sudo` prefix for now (a fix for a future session).

Rebuilding with `sudo` and a clean run this time produced a real 15-step build, and `sudo docker images` confirmed `opencanary:local` present.

## Config

No `.sample` config shipped in this repo. Two config-like files turned up under `find . -iname "*conf*"`:

- `./opencanary/test/opencanary.conf` — a test fixture, not meant for a real run.
- `./data/.opencanary.conf` — the actual default config, used as the starting point.

Copied it out to `~/opencanary.conf` and reviewed the enabled services via `grep -n "enabled"`. Default state had `ftp`, `http`, `https`, `portscan`, and `mongodb` all set to `true`. Trimmed this down for a lean, low-conflict lure set:

- **Kept:** `ftp` (port 21), `http` (port 80), `portscan`
- **Disabled `https`** — its config pointed at a certificate/key pair (`/etc/ssl/opencanary/opencanary.pem`) that doesn't exist on this box, which would have crashed startup.
- **Disabled `mongodb`** — extra attack surface not needed for this exercise; kept the lure count intentionally small to leave clean headroom for the resource/scale test planned in the next lab.

Confirmed no port collisions with Cowrie's existing 2222/2223.

## Detour: config path didn't match the docs

Both the official docs (`docs/starting/configuration.rst`) and the source (`opencanary/config.py`, `SETTINGS = "opencanary.conf"`) pointed toward `/etc/opencanary/opencanary.conf` as the expected config location. Mounting the edited config there and running the container produced a clean-looking `docker run` — but `docker ps` showed no running `opencanary` container, only `Exited (1)`.

`docker logs opencanary` told the real story — the binary actually searches, in order:

1. `/etc/opencanaryd/opencanary.conf` (note the **-d**)
2. `/root/.opencanary.conf`
3. `./opencanary.conf` (relative to cwd)

None of which matched the path used in the mount. Docs and source were both describing intent, not actual runtime behavior. Re-ran with the corrected mount path (`/etc/opencanaryd/opencanary.conf`) and the container came up clean.

## Detour: VM resource exhaustion

Mid-lab, the VM froze twice — once after the build, once after a routine `grep`. Task Manager on the host showed high CPU and memory pressure. Since a `grep` hanging (not just running slow) is a classic symptom of memory exhaustion and swap thrashing rather than raw CPU starvation, bumped both RAM and vCPU allocation for `testbox3` in VirtualBox settings before continuing. No further freezes after the resize.

## Verification

With both containers running:

```
$ sudo docker ps
```

showed `opencanary` (ports 21, 80) and `docker-cowrie` (ports 2223, 2323) running side by side.

`docker logs opencanary` confirmed a clean startup: `CanaryFTP` and `CanaryHTTP` both registered, `FTPFactory` bound to 21, `CanaryHttpServiceSite` bound to 80, and the process dropped privileges to `uid/gid 65534/65534` (nobody/nogroup) as expected from the Dockerfile's `CMD`. `portscan` logged that it can't function inside a container (it needs host-level iptables access) and disabled itself automatically — a known, expected limitation rather than a bug.

Functional test from the host:

```
$ curl http://localhost:80
```

returned the fake login page HTML (`nasLogin` skin, redirect to `/index`) — confirms the HTTP lure is live and convincing.

```
$ curl ftp://localhost:21
curl: (67) Access denied: 530
```

The `530` rejection is correct behavior for an anonymous login attempt — a log entry for the failed login should appear in the OpenCanary logs, capturing the "attack" the same way a real FTP server's rejection would.

## Outcome

Lab objective met: `testbox3` now serves as a dedicated, VLAN-isolated honeypot host running two Dockerized honeypots (Cowrie + OpenCanary) side by side, both logging independently, neither interfering with the other's ports. This is a direct, hands-on rehearsal of the T-Pot architecture pattern ahead of deploying T-Pot itself in a later phase.

## Lessons for next time

- Don't trust a short build step count at face value — check for `CANCELED`/`ERROR` lines even on an apparent success.
- Docs and source comments describe intended behavior; `docker logs` on a failed container shows *actual* behavior. When they disagree, trust the logs.
- A frozen VM after a lightweight command (like `grep`) is more likely a memory/swap problem than a CPU problem — check `free -h` before reaching for more vCPUs.
