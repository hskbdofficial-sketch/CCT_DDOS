
# CCT DDoS Tool v10.6-final

**Developer:** NASIF HIMADRI
**Owner:** ANIK
**Build Date:** 2026-10

---

## ⚠️ DISCLAIMER

**FOR EDUCATIONAL AND AUTHORIZED TESTING PURPOSES ONLY.**

This tool is intended strictly for:
- Security research on systems you own
- Authorized penetration testing with written permission
- Educational study of network protocols and DoS concepts

**Using this tool against any system without explicit written authorization is illegal** under laws like the Computer Fraud and Abuse Act (CFAA), the UK Computer Misuse Act, and similar legislation worldwide. The developers and owners assume **zero liability** for misuse. You are solely responsible for your actions.

---

## Overview

CCT is a multi-layer traffic stress-testing tool with support for HTTP/1.1, HTTP/2, HTTP/3 (QUIC), and raw TCP/UDP flood methods. It features a rich terminal UI, automatic dependency installation, and privilege-based tier escalation (non-root → root → root+Scapy).

---

## Features

- **23 unique attack/tool methods** across L7, L4, and reconnaissance
- **3 privilege tiers** with automatic capability detection:
  - `TIER-3` — Non-root (unprivileged socket attacks)
  - `TIER-2` — Root with `IPPROTO_RAW` (forged-source floods)
  - `TIER-1` — Root + Scapy (full packet forging)
- **Multi-engine architecture:** async (aiohttp), threaded pipelines, slow-connection holders, HTTP/2 (httpx), and raw QUIC (H3)
- **WAF bypass header rotation** — 50+ spoofed headers (XFF, CF-Connecting-IP, X-Real-IP, etc.)
- **Proxy support** via `proxy.txt`
- **Rich/Colorama UI** with branded banners

---

## Requirements

- **Python 3.8+**
- pip
- Linux / macOS / Windows / Termux (Android)

### Auto-installed on first run:
- `requests`, `colorama`, `rich`, `aiohttp`, `httpx`
- `scapy` (optional, root only — for TIER-1)

To skip auto-install prompts, set:
```bash
export CCT_NO_INSTALL=1
```

---

Installation

```bash
git clone https://github.com/hskbdofficial-sketch/CCT_DDOS
cd CCT_DDOS
chmod +x CCT.py
python3 CCT.py
```

On first launch, missing packages are detected and installed automatically.

For root/TIER-1 capabilities:

```bash
sudo python3 CCT.py
```

---

Usage

```bash
python3 CCT.py           # Normal launch
sudo python3 CCT.py      # Root launch (enables raw sockets / Scapy)
```

---

Method Reference

Layer 7 — HTTP

Command Aliases Description
http-get hg, g HTTP GET flood (async)
http-post hp, p HTTP POST flood (async)
http-head hh, h HTTP HEAD flood (async)
cf-bypass cfb Cloudflare bypass attempt
cf-proxy cfp Cloudflare proxy bypass (uses proxy.txt)
pipeline pl HTTP pipelining x64
shield sh Shield pipeline x32
socket so Socket pipeline x16
hulk hk HULK pipeline x24 (cache-busting)
slow-hold sl Slowloris multi-hold (500 conns/thread)
xff-spoof xf X-Forwarded-For spoof pipeline x16
http2-mux h2 HTTP/2 multiplexed flood
http3-quic h3 HTTP/3 QUIC UDP flood

Layer 4 — TCP/UDP

Command Aliases Description
tcp-syn tcp, t TCP SYN flood (raw + Scapy + fallback)
udp-flood udp, u UDP flood (raw + Scapy + fallback)
minecraft mc Minecraft handshake-packet UDP flood
valve vse, v Valve Source Engine query flood

Tools

Command Aliases Description
dns-lookup dns Reverse DNS lookup
geoip geo IP geolocation lookup
subnet sub Subnet calculator

---

Console Commands

Inside the attack console:

Command Action
help / ? Show all methods
clear / cls Clear screen
back Return to main menu
exit / quit Quit the tool
<method> Start attack (prompts for target)

For L7 methods you'll be asked for:

· URL — e.g. https://example.com/path
· THREADS — 1–20000
· TIME — seconds (1–3600)

For L4 methods:

· IP — IP or hostname
· PORT — target port
· THREADS / TIME — same limits

---

Proxy Support

Create a proxy.txt file in the working directory with one proxy per line:

```
127.0.0.1:8080
user:pass@10.0.0.5:3128
proxy.example.com:8888
```

Currently used by cf-proxy. The pool is loaded on startup.

---

Configuration (source-level)

At the top of cct.py:

```python
MAX_TH    = 200000 # Maximum threads per attack
MAX_DUR   = 36000 # Maximum duration in seconds
```

---

How Tiers Work

The tool probes capabilities at startup:

1. Identity constants (_x1.._x4) verify the binary hasn't been tampered with (_validate_identity()).
2. ENV class detects:
   · root status (geteuid() == 0)
   · IPPROTO_RAW availability (forged packets)
   · SO_REUSEPORT support
   · file-descriptor limit (raises to 1,048,576 on Unix)
   · Scapy availability
3. ENV.try_scapy() offers to install Scapy on first root run.

If you modify DEV, OWNER, or TOOL constants you must run python3 cct.py --resign and update the _x1.._x4 values, otherwise the identity check fails and the tool exits.

---

Architecture

```
main()
 ├── _validate_identity()          # integrity check
 ├── ENV()                         # capability probe
 ├── AUTH.login()                  # credential check + lockout
 ├── ENV.try_scapy()               # root upgrade
 └── UI.menu()
      └── UI._console()
           ├── AF.get(cmd)          # alias→method lookup
           └── Engine.run(xd, x, rb, ...)
                ├── AsyncEngine     # aiohttp (GET/POST/HEAD, CF)
                ├── PipeEngine      # pipelined HTTP over raw sockets
                ├── SlowEngine      # Slowloris
                ├── H2Engine        # httpx + HTTP/2
                ├── H3Engine        # raw QUIC UDP
                └── NASIF._ud_rooted / _tcp_rooted  # raw/Scapy sockets
```

---

Known Limitations

· No HTTP/3 crypto handshake — QUIC payloads are syntactically valid headers, not completed handshakes. Effectiveness against real HTTP/3 servers is limited.
· Cloudflare/WAF bypass is best-effort — modern WAFs detect most header-spoofing.
· Proxy support is minimal — only cf-proxy uses the pool.
· Slowloris thread scaling — 500 sockets per thread; high thread counts exhaust FDs even after RLIMIT_NOFILE raise.
· Identity constants — editing DEV/OWNER/TOOL requires --resign.
· Windows — raw sockets and Scapy support are limited; TIER-3 is typical.

---

Legal

This software is provided for authorized security testing and education only. The author(s) do not condone or support illegal use. By using this software you agree that:

· You have explicit written permission for any target you test.
· You will not use it to cause harm, disruption, or financial damage.
· You accept full legal and financial responsibility for your actions.

If you are unsure whether you have permission, do not use this tool.

---

Credits

· Developer: NASIF HIMADRI
· Owner: ANIK
· Tool: CCT_DDOS v10.6-final

Community:

· Facebook: ChandpurCyberTeam
· Telegram: @Chandpur_Cyber_Team

---

Changelog

v10.6-final (2025-01)

· Added HTTP/3 QUIC flood engine
· Added HTTP/2 multiplex engine
· Refactored all engines under unified XD/X/RB context
· Rich UI with panels, tables, branded banners
· Automatic dependency installer
· Tiered privilege model (3 → 1)
· Slowloris rewritten with 500 conns/thread
· Scapy auto-install on first root run
· --version and --resign flags

```

---

**End of README**
