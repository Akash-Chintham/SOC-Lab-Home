# Detections

SIGMA rules and Wazuh notes derived from telemetry observed in this lab. Each rule links back to the attack phase that produced its signal.

| Rule | Detects | ATT&CK | Source phase |
|------|---------|--------|--------------|
| [`shadow-copy-deletion.yml`](shadow-copy-deletion.yml) | vssadmin/wmic shadow copy deletion | T1490 | Phase 4 |
| [`mass-file-rename-fim.md`](mass-file-rename-fim.md) | ransomware file-rename storm (FIM burst) | T1486 | Phase 4 |

Workflow: observe telemetry → identify the highest-fidelity field(s) → write SIGMA → (optionally) translate to a native Wazuh rule → test that it fires on the attack and stays quiet on benign activity.
