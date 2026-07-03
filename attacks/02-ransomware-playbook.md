# Phase 4 — Ransomware Behavior Simulation

**Goal:** produce the *telemetry* a ransomware attack generates, without any real encryptor. Every step is a benign action run manually on the target so the resulting alert chain is realistic and analyzable.

> **Safety:** no real encryption, no downloaded payloads, no spreading. Files are throwaway test data in a dedicated folder. Snapshot the VM as `pre-ransomware-sim` first so you can roll back.

> **Note on the commands:** the exact command lines for the destructive steps (2, 4, 5) are intentionally kept out of this file because on-host endpoint security locks any file containing them mid-write — a live demo of the very behavior we're detecting. The full command set is mirrored by the detection logic in [`../detections/shadow-copy-deletion.yml`](../detections/shadow-copy-deletion.yml). Type the commands directly into the target VM from your notes rather than saving them to this repo.

## ATT&CK techniques exercised

| Step | Technique | ID |
|------|-----------|-----|
| Staging files | — (setup) | — |
| Mass rename/rewrite | Data Encrypted for Impact (simulated) | T1486 |
| Delete shadow copies | Inhibit System Recovery | T1490 |
| Drop ransom note | — (impact artifact) | T1486 |
| Scheduled task persistence | Scheduled Task/Job | T1053.005 |
| Run key persistence | Registry Run Keys | T1547.001 |

## 0. Stage throwaway victim files

```powershell
$lab = "$env:USERPROFILE\Desktop\ransom-lab"
New-Item -ItemType Directory -Path $lab -Force | Out-Null
1..25 | ForEach-Object { "sample data $_" | Out-File "$lab\file$_.txt" }
```

## 1. Simulate mass "encryption" (rename + rewrite)

Rewrite each staged file's contents and rename it to a fake locked extension in a tight loop. This reproduces the file-touch/rename storm ransomware produces — no crypto needed to generate the FIM signal.

**Expect:** a burst of realtime FIM alerts (hash changes + additions/renames) on Desktop within seconds — the core detection signal for T1486.

## 2. Delete Volume Shadow Copies (T1490)

Run the shadow-copy deletion utilities (vssadmin / wmic / wbadmin / bcdedit) — the exact commands are in your notes and encoded in the SIGMA rule. This actually deletes shadow copies, so only run it in the disposable VM.

**Expect:** 4688 / Sysmon Event ID 1 for `vssadmin.exe` with a shadow-copy-delete command line. High-fidelity: very little benign software does this.

## 3. Drop a ransom note

Write a plain-text note file (e.g. `READ_ME_RANSOM.txt`) into the lab folder.

**Expect:** a FIM addition alert for the note appearing alongside the rename storm.

## 4. Persistence — scheduled task (T1053.005)

Create an on-logon scheduled task via `schtasks`.

**Expect:** TaskScheduler Operational events + Security 4698 (scheduled task created).

## 5. Persistence — Run key (T1547.001)

Add a value under `HKCU\...\CurrentVersion\Run` via `reg add`.

**Expect:** Sysmon Event ID 13 (registry value set) targeting a Run key.

## Cleanup / rollback

Delete the scheduled task and the Run-key value, then restore the `pre-ransomware-sim` snapshot to fully reset.

## Deliverable

Record the full alert chain (timestamps, rule IDs, ATT&CK IDs) in [`../writeups/phase-04-ransomware.md`](../writeups/phase-04-ransomware.md), then translate the highest-fidelity behaviors into SIGMA rules in [`../detections/`](../detections/).
