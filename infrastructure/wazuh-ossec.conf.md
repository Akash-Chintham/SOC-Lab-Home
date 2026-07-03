# Wazuh agent ossec.conf — reference blocks

Location on the target: `C:\Program Files (x86)\ossec-agent\ossec.conf`.
Restart after editing: `Restart-Service Wazuh`.

## Log channel forwarding (no filters)

Forward each channel with no `<query>` filter so nothing is silently dropped:

```xml
<localfile>
  <location>Security</location>
  <log_format>eventchannel</log_format>
</localfile>

<localfile>
  <location>Application</location>
  <log_format>eventchannel</log_format>
</localfile>

<localfile>
  <location>System</location>
  <log_format>eventchannel</log_format>
</localfile>

<localfile>
  <location>Microsoft-Windows-PowerShell/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>

<localfile>
  <location>Microsoft-Windows-Windows Defender/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>

<localfile>
  <location>Microsoft-Windows-TaskScheduler/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>

<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>

<localfile>
  <location>Microsoft-Windows-TerminalServices-LocalSessionManager/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

## Realtime FIM on user folders

```xml
<syscheck>
  <disabled>no</disabled>
  <directories realtime="yes" check_all="yes" report_changes="yes">C:\Users\%USERNAME%\Desktop</directories>
  <directories realtime="yes" check_all="yes" report_changes="yes">C:\Users\%USERNAME%\Documents</directories>
</syscheck>
```

> Note: hard-coded user paths are simplest for a single-user lab. Confirm the actual username on the target and substitute it. `report_changes="yes"` records what changed, which is useful for the ransomware-behavior write-up.
