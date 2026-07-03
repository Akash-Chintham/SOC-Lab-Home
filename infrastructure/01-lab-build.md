# Phase 1 — Lab Build (fresh rebuild)

Building the lab from scratch on a new machine. Order matters: hypervisor → VMs → networking → Wazuh agent → logging. Do not run any attack until Phase 2 confirms telemetry is flowing.

**Target end state**

| Host | Role | soclab IP | Notes |
|------|------|-----------|-------|
| Kali Linux | Attacker | 172.30.32.1 | internal net only |
| Windows 10 | Target | 172.30.32.5 | internal net + NAT to reach Wazuh |
| Wazuh Manager | SIEM | 10.0.0.5 (Azure) | reached over VPN |

---

## 1. Install the hypervisor (VirtualBox)

1. Download VirtualBox for Windows from the official site (virtualbox.org) plus the matching **Extension Pack**.
2. Install VirtualBox with defaults. Reboot if prompted (it installs host-only network drivers).
3. Install the Extension Pack: **File → Tools → Extension Pack Manager → Install**.
4. Verify: open VirtualBox, **Help → About** should show the version. `VBoxManage --version` in a terminal confirms the CLI works.

> If your CPU virtualization is disabled you'll get errors creating 64-bit VMs. Enable **Intel VT-x / AMD-V** in BIOS/UEFI.

---

## 2. Create the Windows 10 target VM

1. Download a Windows 10 evaluation ISO (Microsoft Evaluation Center) — 90-day eval is fine for a lab.
2. **New** → Name `AKASH-WIN-TARGET`, Type *Microsoft Windows*, Version *Windows 10 (64-bit)*.
3. Memory 4096 MB (2048 minimum), 2 vCPU, 50 GB dynamically-allocated VDI.
4. Mount the ISO, boot, complete a standard install. Create a local account (skip Microsoft account).
5. Install **Guest Additions** (Devices → Insert Guest Additions CD) for clipboard/display, then reboot.
6. Take a snapshot named `clean-install` before changing anything else.

## 3. Create the Kali attacker VM

1. Download the **VirtualBox** Kali image from kali.org (pre-built .vbox/.ova is fastest).
2. **File → Import Appliance** (for OVA) or **New** + attach the disk.
3. Memory 2048–4096 MB, 2 vCPU.
4. Boot, log in (`kali`/`kali` on the prebuilt image), then `sudo apt update && sudo apt full-upgrade -y`.
5. Change the default password. Snapshot as `clean-install`.

---

## 4. Networking

Two networks are in play. Get this right early — last time the real subnet differed from the plan and caused avoidable troubleshooting.

### Internal "soclab" network (attacker ↔ target)

For **both** VMs, add an adapter set to **Internal Network** named exactly `soclab`:

- VM **Settings → Network → Adapter 1 → Attached to: Internal Network → Name: `soclab`**.

Internal networks have no DHCP by default, so assign static IPs.

**Windows target (172.30.32.5):**
Control Panel → Network → adapter → IPv4 properties:
- IP `172.30.32.5`, mask `255.255.255.0`, gateway blank, DNS blank.

**Kali attacker (172.30.32.1):**
```bash
sudo ip addr add 172.30.32.1/24 dev eth0
sudo ip link set eth0 up
# persist via /etc/network/interfaces or nmcli if you want it to survive reboot
```

**Verify connectivity:**
```bash
# from Kali
ping -c3 172.30.32.5
```
If Windows doesn't reply, temporarily allow ICMP: on Windows run in an elevated PowerShell:
```powershell
New-NetFirewallRule -DisplayName "Allow ICMPv4-In" -Protocol ICMPv4 -IcmpType 8 -Direction Inbound -Action Allow
```

### NAT adapter (Windows → Wazuh manager over VPN)

Add a **second** adapter to the Windows VM only:
- **Adapter 2 → Attached to: NAT**.

This gives Windows outbound internet so it can reach the Azure Wazuh manager (`10.0.0.5`) once the VPN is connected on the host or in the guest, depending on your VPN setup.

> Keep the attacker VM off the NAT/internet path — it should only see `soclab`. This keeps the offensive tooling contained.

---

## 5. Install Sysmon on the Windows target

Sysmon massively enriches process, network, and file telemetry. Install before the Wazuh agent so the channel exists.

```powershell
# from an elevated PowerShell, in the Sysmon folder
.\Sysmon64.exe -accepteula -i sysmonconfig.xml
```
Use a well-known config (e.g. SwiftOnSecurity or Olaf Hartong's `sysmon-modular`) as a starting point. Confirm the service:
```powershell
Get-Service Sysmon64
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5
```

---

## 6. Install & register the Wazuh agent

1. From the Windows VM, download the Wazuh agent MSI matching the manager version.
2. Install pointing at the manager:
```powershell
# adjust version to match your manager
msiexec.exe /i wazuh-agent.msi /q WAZUH_MANAGER="10.0.0.5" WAZUH_AGENT_NAME="AKASHTESTWINDOWS"
```
3. Register the agent from the manager (Azure) side, then start the service:
```powershell
NET START Wazuh
```
4. Confirm on the manager the agent shows **Active** (previously it registered as `AKASHTESTWINDOWS`, ID 008).

### ossec.conf forwarding

Edit `C:\Program Files (x86)\ossec-agent\ossec.conf` to forward the channels this lab needs, with **no event filters** so nothing is silently dropped, and realtime FIM on user folders. Forward these channels:

- Security, Application, System
- Microsoft-Windows-PowerShell/Operational
- Microsoft-Windows-Windows Defender/Operational
- Microsoft-Windows-TaskScheduler/Operational
- Microsoft-Windows-Sysmon/Operational
- Microsoft-Windows-TerminalServices (RDP)

Add realtime FIM on Desktop and Documents (`<directories realtime="yes" check_all="yes">`). Restart the agent after editing:
```powershell
Restart-Service Wazuh
```
See [`wazuh-ossec.conf.md`](wazuh-ossec.conf.md) for the exact block.

---

## 7. Harden Windows logging

Default Windows logging is too sparse for SOC work. Enable each of these (details and commands in [`03-logging-hardening.md`](03-logging-hardening.md)):

- **PowerShell Script Block Logging** (registry / Group Policy)
- **Process Creation auditing** *with command line* (event 4688)
- **Logon auditing** (4624/4625)
- **Filtering Platform Connection auditing** (5156) for network visibility

Then reboot the target and snapshot as `lab-ready`.

---

## Definition of done for Phase 1

- [ ] Both VMs boot and reach each other on `172.30.32.0/24`
- [ ] Windows reaches the Wazuh manager; agent shows **Active** on the manager
- [ ] Sysmon Operational log producing events
- [ ] ossec.conf forwarding all listed channels; agent restarted cleanly
- [ ] PowerShell, process-creation-with-cmdline, logon, and FWP auditing all enabled
- [ ] `lab-ready` snapshot taken on both VMs

Proceed to [Phase 2 — telemetry verification](02-telemetry-verification.md).
