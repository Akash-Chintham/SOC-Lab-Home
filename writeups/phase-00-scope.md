# Phase 0 — Scope & Objectives

## Purpose

Build an "Attack & Defend" SOC lab from scratch to develop hands-on detection-engineering skills and produce a portfolio, while preparing for an authorized cyber-drill engagement.

## Learning priorities

Detection over offense: capture telemetry in a SIEM, analyze raw alerts, map to MITRE ATT&CK, and write detection rules. Understand every step from fundamentals rather than skipping ahead; favor manual behavioral simulation over automated frameworks where it builds deeper understanding.

## In scope

- Isolated VMs on owned hardware (VirtualBox)
- Telemetry collection (Wazuh + Sysmon + hardened Windows auditing)
- Manual, benign simulation of attacker behavior (recon, ransomware behavior, persistence)
- ATT&CK mapping and SIGMA rule authoring
- Atomic Red Team as a controlled simulation tool (in progress)

## Out of scope

- Working malicious payloads / real ransomware encryptors — behavior is simulated instead
- Any activity outside the owned lab network
- Agentic AI alert-pipeline integration (explicitly deferred)

## Guiding principles

1. Logging must be in place and **verified before** any attack — telemetry must exist to analyze.
2. Default Windows logging is insufficient; harden it explicitly.
3. Validate network assumptions early (subnet, reachability).
4. Each phase ends with a commit to keep portfolio history clean.
5. Phase-gated progression: infrastructure → telemetry → attacks → detection → documentation.

## Authorization

The client cyber-drill engagement is contracted by management with Akash as on-site practitioner. All lab activity runs on Akash-owned equipment. Nothing in this repo is run against third-party systems.
