# Phase 4 — Ransomware Behavior Write-up

**Date:**
**ATT&CK techniques:** T1486, T1490, T1053.005, T1547.001
**Attacker:** Kali 172.30.32.1 → **Target:** Windows 172.30.32.5

Run the playbook in [`../attacks/02-ransomware-playbook.md`](../attacks/02-ransomware-playbook.md), then fill this in.

## What I did

<Paste the exact commands run, in order.>

## Telemetry observed

| Time | Source (log/channel) | Event/Rule ID | What it means |
|------|----------------------|---------------|---------------|
| | FIM syscheck | | mass file rename storm |
| | Security / Sysmon | 4688 / EID 1 | vssadmin delete shadows |
| | Security | 4698 | scheduled task created |
| | Sysmon | EID 13 | Run key set |

## ATT&CK mapping

| Behavior | Technique | ID |
|----------|-----------|-----|
| Mass file rename/rewrite | Data Encrypted for Impact | T1486 |
| Shadow copy deletion | Inhibit System Recovery | T1490 |
| Scheduled task | Scheduled Task/Job | T1053.005 |
| Run key | Registry Run Keys | T1547.001 |

## Detection outcome

- Which existing rules fired?
- Did the custom FIM-burst rule (`detections/mass-file-rename-fim.md`) trigger?
- Did the shadow-copy SIGMA rule (`detections/shadow-copy-deletion.yml`) match?
- Gaps found / rules to add?

## Lessons learned

<Notes.>
