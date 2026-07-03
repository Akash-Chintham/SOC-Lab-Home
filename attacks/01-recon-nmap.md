# Phase 3 — Reconnaissance (Nmap)

**ATT&CK:** T1046 Network Service Discovery
**Attacker:** Kali (172.30.32.1) → **Target:** Windows (172.30.32.5)

## Commands

```bash
# host discovery
sudo nmap -sn 172.30.32.0/24

# service/version scan of the target
sudo nmap -sV -sC -p- 172.30.32.5 -oN recon-full.txt

# faster top-ports pass
sudo nmap -sV --top-ports 1000 172.30.32.5
```

## Expected result (baseline from prior build)

Open on the Windows target: **135** (RPC), **139** (NetBIOS), **445** (SMB), **3389** (RDP).
Hostname observed: `DESKTOP-MDUG0PR`, Windows 10.

## What to look for in Wazuh

- Spikes of **5156** (Filtering Platform Connection) from `172.30.32.1` across many ports — the fingerprint of a port sweep.
- Sysmon **Event ID 3** (network connection) entries if Sysmon network logging is on.
- Multiple short-lived connections to 135/139/445/3389 in a tight time window.

## Analysis notes

| Field | Value to capture |
|-------|------------------|
| Source IP | 172.30.32.1 |
| Ports touched | (from 5156 volume) |
| Time window | |
| Wazuh rules fired | |

Record findings in [`../writeups/phase-03-recon.md`](../writeups/phase-03-recon.md) and note whether a detection rule for "many distinct destination ports from one source in N seconds" would fire.
