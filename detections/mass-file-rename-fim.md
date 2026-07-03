# Detection: Mass File Rename / Encryption Storm (FIM)

**ATT&CK:** T1486 Data Encrypted for Impact
**Signal source:** Wazuh realtime FIM (syscheck) on Desktop/Documents

## Idea

A single process modifying and renaming many files in a few seconds is the defining behavior of ransomware. FIM produces a hash-change / rename alert per file; the detection is the **volume + rate**, not any single event.

## Wazuh approach

Use a frequency/correlation rule that fires when many syscheck alerts share a short time window. Sketch of a custom rule (`local_rules.xml`):

```xml
<group name="ransomware,syscheck,">
  <rule id="100200" level="12" frequency="15" timeframe="30">
    <if_matched_group>syscheck</if_matched_group>
    <description>Possible ransomware: high volume of file modifications in short window (T1486)</description>
    <mitre>
      <id>T1486</id>
    </mitre>
  </rule>
</group>
```
Tune `frequency` (event count) and `timeframe` (seconds) against your baseline so normal saves don't trip it.

## Corroborating signals to pivot on

- New files with anomalous extensions (`.locked`, random strings) appearing in bulk.
- A `READ_ME`/ransom-note file created in the same directory.
- Immediately preceded or followed by shadow copy deletion (see `shadow-copy-deletion.yml`) — chaining the two gives very high confidence.

## Testing

Run [`../attacks/02-ransomware-playbook.md`](../attacks/02-ransomware-playbook.md) step 1 and confirm the rule fires; then do a normal handful of file edits and confirm it stays quiet.
