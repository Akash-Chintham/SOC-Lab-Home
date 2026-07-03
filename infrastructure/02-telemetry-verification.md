# Phase 2 — Telemetry Verification

**Gate rule:** no attack runs until the alerts below are confirmed in Wazuh. Telemetry must exist before there is anything to analyze.

## Generate benign test events

On the Windows target:

```powershell
# process creation with command line -> expect 4688 / Wazuh rule 67027
whoami /all
ipconfig /all

# PowerShell script block -> expect 4104
Write-Host "telemetry test $(Get-Date)"

# FIM test on Desktop -> expect realtime syscheck alert
"test $(Get-Date)" | Out-File "$env:USERPROFILE\Desktop\fim-test.txt"
Add-Content "$env:USERPROFILE\Desktop\fim-test.txt" "modified"
```

## Confirm in Wazuh

Check the manager / dashboard for:

| Signal | Wazuh rule | Meaning |
|--------|-----------|---------|
| Process creation w/ cmdline | **67027** | 4688 forwarding + cmdline auditing working |
| Audit policy change | **60112** | audit subsystem reporting correctly |
| FIM file hash change | syscheck (realtime) | ossec.conf FIM block working |

If any are missing, fix before continuing:
- No 4688 → check `auditpol` and the ProcessCreationIncludeCmdLine reg key.
- No FIM → check the `<directories realtime="yes">` block and restart the agent.
- Nothing at all → agent not Active on manager, or NAT/VPN path to `10.0.0.5` is down.

## Verification log

| Date | Signal tested | Rule | Result |
|------|---------------|------|--------|
| | process creation | 67027 | |
| | audit policy change | 60112 | |
| | FIM hash change | syscheck | |

Once all rows pass, proceed to [Phase 3 — reconnaissance](../attacks/01-recon-nmap.md).
