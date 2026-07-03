# Attack & Defend SOC Lab

A hands-on detection-engineering home lab. The goal is to generate real attacker telemetry in a controlled environment, analyze it in a SIEM, map findings to MITRE ATT&CK, and write detection rules — building both practical SOC skills and a portfolio.

> **Scope & ethics:** Every technique here runs against VMs I own, on an isolated internal network. No working malicious payloads (e.g. real ransomware encryptors) are used — attacker behavior is simulated manually so the resulting telemetry is realistic while the risk stays zero.

## Architecture

```
                 ┌─────────────────────────────┐
                 │        Azure (cloud)         │
                 │   Wazuh Manager  10.0.0.5    │
                 └──────────────▲──────────────┘
                                │ agent traffic (over VPN)
                                │ NAT adapter
   internal net "soclab"        │
   172.30.32.0/24               │
  ┌──────────────┐      ┌───────┴────────────┐
  │  Kali Linux  │◄────►│   Windows 10       │
  │  (attacker)  │      │   (target)         │
  │ 172.30.32.1  │      │ 172.30.32.5        │
  └──────────────┘      │ Wazuh agent (008)  │
                        └────────────────────┘
```

- **Hypervisor:** Oracle VirtualBox
- **Attacker:** Kali Linux
- **Target:** Windows 10, hardened logging, Sysmon + Wazuh agent
- **SIEM:** Wazuh manager hosted in Azure, reached over VPN
- **Networking:** VMs share an internal-only network (`soclab`); the Windows VM carries a second NAT adapter purely to reach the Wazuh manager

## Repository layout

| Folder | Contents |
|--------|----------|
| [`infrastructure/`](infrastructure/) | Build guides: VirtualBox, VM setup, networking, Wazuh agent, logging hardening |
| [`attacks/`](attacks/) | Attack simulation playbooks (recon, ransomware behavior, persistence) with the exact commands run |
| [`detections/`](detections/) | SIGMA rules and Wazuh rule notes derived from observed telemetry |
| [`writeups/`](writeups/) | Per-phase analysis: alerts triggered, ATT&CK mapping, lessons learned |

## Phase roadmap

The lab progresses in gates — telemetry must exist and be verified *before* any attack is run.

- [x] **Phase 0 — Plan & scope** ([writeup](writeups/phase-00-scope.md))
- [ ] **Phase 1 — Infrastructure** — VirtualBox, VMs, network, Wazuh agent, logging ([guide](infrastructure/01-lab-build.md))
- [ ] **Phase 2 — Telemetry verification** — confirm the expected events reach Wazuh ([guide](infrastructure/02-telemetry-verification.md))
- [ ] **Phase 3 — Reconnaissance** — Nmap from Kali, analyze scan telemetry ([playbook](attacks/01-recon-nmap.md))
- [ ] **Phase 4 — Ransomware behavior simulation** — FIM, mass file ops, shadow copy deletion, persistence ([playbook](attacks/02-ransomware-playbook.md))
- [ ] **Phase 5 — Detection engineering** — write SIGMA rules from observed telemetry ([detections](detections/))
- [ ] **Phase 6 — Documentation** — full ATT&CK-mapped write-ups per phase ([writeups](writeups/))

## MITRE ATT&CK coverage (planned)

| Tactic | Technique | Status |
|--------|-----------|--------|
| Reconnaissance / Discovery | T1046 Network Service Discovery | planned |
| Execution | T1059.001 PowerShell / T1059.003 cmd | planned |
| Impact | T1486 Data Encrypted for Impact (simulated) | planned |
| Impact | T1490 Inhibit System Recovery (shadow copy deletion) | planned |
| Persistence | T1053.005 Scheduled Task | planned |
| Persistence | T1547.001 Registry Run Keys | planned |

## Toolchain

Virtualization: VirtualBox · SIEM: Wazuh · Endpoint: Sysmon · Frameworks: MITRE ATT&CK, SIGMA · Simulation: manual PowerShell/CMD, Atomic Red Team (in progress) · Planned: Metasploit, Mimikatz, BloodHound, ATT&CK Navigator

---
*Portfolio project by Akash. Each phase ends with a commit to keep history clean.*
