# Windows Logging Hardening

Default Windows logging misses most of what a SOC needs. Enable these on the target **before** any attack simulation. Run all commands in an **elevated** PowerShell/CMD.

## 1. Process creation auditing (Event ID 4688)

```cmd
auditpol /set /subcategory:"Process Creation" /success:enable /failure:enable
```

## 2. Include the command line in 4688

Without this, 4688 shows the process name but not the arguments — the arguments are where the attack lives.

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit" /v ProcessCreationIncludeCmdLine_Enabled /t REG_DWORD /d 1 /f
```

## 3. PowerShell Script Block Logging (Event ID 4104)

```cmd
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" /v EnableScriptBlockLogging /t REG_DWORD /d 1 /f
```
Optionally also enable module and transcription logging for fuller coverage.

## 4. Logon auditing (Event IDs 4624 / 4625)

```cmd
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
```

## 5. Filtering Platform Connection auditing (Event ID 5156)

Gives per-connection network visibility from the Windows side (complements Sysmon Event ID 3).

```cmd
auditpol /set /subcategory:"Filtering Platform Connection" /success:enable /failure:enable
```

## Verify what's enabled

```cmd
auditpol /get /category:*
```

Reboot, generate a test process (e.g. `whoami`), and confirm a 4688 with a command line appears in the Security log.
