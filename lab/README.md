# Cybersecurity Home Lab - VirtualBox Setup Guide

## Overview

A comprehensive, multi-purpose cybersecurity lab covering:
- Penetration Testing
- Blue Team / SOC (SIEM, Detection)
- Malware Analysis & Reverse Engineering
- Active Directory Attacks
- Network Security (Firewall, IDS/IPS)
- CTF / VulnHub Challenges
- Pwn / Binary Exploitation

### Hardware Constraints: 16GB RAM

With 16GB RAM, we cannot run everything simultaneously. The lab is designed in **phases** — start with Phase 1, expand as needed. Each phase is self-contained.

---

## Network Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    HOST (Windows 11)                     │
│                                                          │
│  VirtualBox                                              │
│  ┌───────────────────────────────────────────────────┐   │
│  │  NAT Network: "attack-net" (10.0.1.0/24)          │   │
│  │  (Pentesting + CTF - machines talk to each other)  │   │
│  │                                                    │   │
│  │  ┌──────────┐  ┌──────────────┐  ┌──────────────┐ │   │
│  │  │ Kali     │  │ Metasploitable│  │ VulnHub VMs  │ │   │
│  │  │ Linux    │  │ 2/3          │  │ (one at a    │ │   │
│  │  │ (2GB)    │  │ (1GB)        │  │  time, 1-2GB)│ │   │
│  │  └──────────┘  └──────────────┘  └──────────────┘ │   │
│  └───────────────────────────────────────────────────┘   │
│                                                          │
│  ┌───────────────────────────────────────────────────┐   │
│  │  Host-Only: "vboxnet0" (192.168.56.0/24)          │   │
│  │  (SSH Management - host-to-VM access)             │   │
│  │                                                    │   │
│  │  Kali: 192.168.56.10  Metasploitable: 192.168.56.20│   │
│  └───────────────────────────────────────────────────┘   │
│                                                          │
│  ┌───────────────────────────────────────────────────┐   │
│  │  NAT Network: "corp-net" (10.0.2.0/24)            │   │
│  │  (Active Directory Lab)                            │   │
│  │                                                    │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────────┐│   │
│  │  │ Win Srv  │  │ Win 10   │  │ Kali (shared     ││   │
│  │  │ 2022 DC  │  │ Pro      │  │  via 2nd NIC)    ││   │
│  │  │ (4GB)    │  │ (2GB)    │  │                  ││   │
│  │  └──────────┘  └──────────┘  └──────────────────┘│   │
│  └───────────────────────────────────────────────────┘   │
│                                                          │
│  ┌───────────────────────────────────────────────────┐   │
│  │  Host-Only: "malware-net" (192.168.56.0/24)       │   │
│  │  (Malware Analysis - ISOLATED, no internet)        │   │
│  │                                                    │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────────┐│   │
│  │  │ FlareVM  │  │ REMnux   │  │ Win 10 Victim    ││   │
│  │  │ (Win RE) │  │ (Linux   │  │ (clean snapshot) ││   │
│  │  │ (4GB)    │  │  analy)  │  │ (2GB)            ││   │
│  │  │          │  │ (2GB)    │  │                  ││   │
│  │  └──────────┘  └──────────┘  └──────────────────┘│   │
│  └───────────────────────────────────────────────────┘   │
│                                                          │
│  ┌───────────────────────────────────────────────────┐   │
│  │  NAT Network: "soc-net" (10.0.3.0/24)             │   │
│  │  (Blue Team / SOC - monitoring + targets)          │   │
│  │                                                    │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────────┐│   │
│  │  │ pfSense  │  │ Wazuh    │  │ Target Server    ││   │
│  │  │ Firewall │  │ SIEM     │  │ (Ubuntu/Win)     ││   │
│  │  │ (1GB)    │  │ (4GB)    │  │ (1-2GB)          ││   │
│  │  └──────────┘  └──────────┘  └──────────────────┘│   │
│  └───────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

## Phase 1: Foundation (Run these first)

### VM 1: Kali Linux (Attack Machine)
- **RAM:** 2GB
- **Disk:** 40GB
- **Network:** NAT Network "attack-net"
- **Purpose:** Primary penetration testing platform
- **Download:** https://www.kali.org/get-kali/ (VirtualBox pre-built image)
- **Pre-installed tools:** Nmap, Metasploit, Burp Suite, Hydra, John, SQLMap, Gobuster, Impacket, BloodHound, Ghidra, pwntools

### VM 2: Metasploitable 2 (Target)
- **RAM:** 512MB - 1GB
- **Disk:** Pre-built (~2GB)
- **Network:** NAT Network "attack-net"
- **Purpose:** Intentionally vulnerable Linux for pentesting practice
- **Download:** https://sourceforge.net/projects/metasploitable/files/Metasploitable2/

### VM 3: Metasploitable 3 (Advanced Target)
- **RAM:** 2GB
- **Disk:** 40GB
- **Network:** NAT Network "attack-net"
- **Purpose:** Modern vulnerable Windows + Linux services
- **Download:** https://github.com/rapid7/metasploitable3 (Packer/Vagrant build)

---

## Phase 2: Active Directory Lab

### VM 4: Windows Server 2022 (Domain Controller)
- **RAM:** 4GB
- **Disk:** 60GB
- **Network:** NAT Network "corp-net"
- **Purpose:** Domain Controller with AD DS, DNS, DHCP
- **ISO:** Windows Server 2022 Evaluation (free from Microsoft)
- **Setup:**
  1. Install Windows Server 2022
  2. Add AD DS role → Promote to Domain Controller
  3. Create new forest: `corp.local`
  4. Create OUs, users, groups for attack practice

### VM 5: Windows 10/11 Pro (Domain Joined Workstation)
- **RAM:** 2GB
- **Disk:** 40GB
- **Network:** NAT Network "corp-net"
- **Purpose:** Domain-joined endpoint for lateral movement practice
- **Intentional misconfigurations to add:**
  - Stored credentials in registry
  - Weak service account permissions
  - LAPS not configured
  - SMB signing disabled
  - PowerShell Constrained Language Mode off

### AD Attack Practice:
- Kerberoasting, AS-REP Roasting
- Pass-the-Hash, Pass-the-Ticket
- Golden/Silver Ticket attacks
- DCSync, Domain Dominance
- LLMNR/NBT-NS poisoning (Responder)
- BloodHound enumeration
- GPO abuse, ACL abuse

---

## Phase 3: Malware Analysis & Reverse Engineering

### VM 6: FLARE-VM (Windows RE Workstation)
- **RAM:** 4GB
- **Disk:** 60GB
- **Network:** Host-Only "malware-net" (NO internet)
- **Purpose:** Windows malware analysis and reverse engineering
- **Setup:**
  1. Install clean Windows 10 VM
  2. Disable Windows Update, Defender, SmartScreen
  3. Install FLARE-VM: https://github.com/mandiant/flare-vm
  4. Take a clean snapshot BEFORE any malware analysis
- **Tools included:** IDA Free, x64dbg, Ghidra, PEStudio, Process Monitor, Sysinternals, YARA, FLOSS

### VM 7: REMnux (Linux Analysis Server)
- **RAM:** 2GB
- **Disk:** 40GB
- **Network:** Host-Only "malware-net"
- **Purpose:** Linux toolkit for malware analysis, network traffic analysis
- **Download:** https://docs.remnux.org/install-distro/get-virtual-appliance (OVA)
- **Tools included:** Wireshark, INetSim, FakeDNS, YARA, Radare2, CyberChef, exiftool, strings

### VM 8: Windows 10 Victim (Clean Target)
- **RAM:** 2GB
- **Disk:** 40GB
- **Network:** Host-Only "malware-net"
- **Purpose:** Clean VM to execute malware samples and observe behavior
- **Setup:**
  1. Install Windows 10
  2. Disable all security features
  3. Install Sysmon, Process Monitor, Process Explorer
  4. Take a clean snapshot
  5. NEVER share files via drag-and-drop or shared folders

### Malware Analysis Isolation Rules:
- Host-Only network ONLY — no internet, no host access
- Disable drag-and-drop between host and VM
- Disable shared clipboard
- Disable USB passthrough
- Always snapshot before running a sample
- Hash every sample (SHA256) before analysis
- Restore snapshot after each analysis

---

## Phase 4: Blue Team / SOC Lab

### VM 9: pfSense (Firewall)
- **RAM:** 1GB
- **Disk:** 8GB (dynamic)
- **Network:** 3 NICs — WAN (NAT), LAN (NAT Network "soc-net"), OPT1 (Host-Only vboxnet0)
- **Purpose:** Network firewall, IDS/IPS (Snort/Suricata), traffic monitoring
- **Setup:**
   1. Create VM → FreeBSD 64-bit → 1GB RAM → 8GB disk
   2. NIC 1: NAT (WAN), NIC 2: soc-net (LAN), NIC 3: vboxnet0 (OPT1/SSH)
   3. Install pfSense → Assign interfaces: em0=WAN, em1=LAN, em2=OPT1
   4. LAN IP: 10.0.3.1/24, OPT1 IP: 192.168.56.80/24
   5. Enable SSH via console option 14
   6. Web GUI at http://192.168.56.80 (admin/pfsense)
   7. Firewall rules: allow TCP 22 from OPT1 for SSH, allow any from OPT1 to LAN
   8. (Optional) Install Suricata package for IDS/IPS

### VM 10: Wazuh SIEM (Detection Platform)
- **RAM:** 4GB (reduced from default 8GB)
- **Disk:** 50GB (VMDK from OVA)
- **CPU:** 2 (reduced from default 4)
- **Network:** NIC 1 (soc-net: 10.0.3.85/24), NIC 2 (Host-Only vboxnet0: 192.168.56.85)
- **Purpose:** SIEM, log aggregation, intrusion detection, file integrity monitoring
- **Deploy:** Import `wazuh-4.14.5.ova` with reduced specs
- **Credentials:** wazuh-user/wazuh (SSH), admin/admin (Dashboard)
- **Dashboard:** https://192.168.56.85
- **Practice:**
  - Deploy Wazuh agents on target VMs (Kali, Metasploitable, Win11, DC, etc.)
  - Create custom detection rules
  - Monitor AD attack logs
  - Detect malware C2 beacons
  - File integrity monitoring (FIM)

---

## Phase 5: CTF / VulnHub

### Additional VulnHub VMs (one at a time)
- **RAM:** 1-2GB each
- **Network:** NAT Network "attack-net"
- **Recommended machines (beginner to advanced):**
  - Mr. Robot (web-focused)
  - Kioptrix series (classic)
  - HackNos series
  - Sunset series
  - DC series (1-9, Active Directory focused)
- **Download:** https://www.vulnhub.com/

### Pwn / Binary Exploitation Practice:
- **pwn.college** (online, free)
- **ROP Emporium** (local challenges)
- **Pwnable.kr** (online)
- **Exploit Education** (Phoenix, Nebula VMs)
- **Protostar VM** (classic buffer overflow practice)

---

## RAM Budget (16GB Total)

| Phase | VMs Running Simultaneously | RAM Used |
|-------|---------------------------|----------|
| Phase 1 | Kali (2GB) + Metasploitable 2 (1GB) + 1 VulnHub (1-2GB) | ~5GB |
| Phase 2 | Kali (2GB) + Win Server DC (4GB) + Win 11 AD (2GB) | ~8GB |
| Phase 3 | FLARE-VM (4GB) + REMnux (2GB) + Win11 Victim (2GB) | ~8GB |
| Phase 4 | pfSense (1GB) + Wazuh (4GB) + Target (1-2GB) | ~7GB |
| Phase 5 | Kali (2GB) + 1 VulnHub VM (1-2GB) | ~4GB |

**You can run 2 phases at a time if needed** (e.g., Phase 1 + Phase 2 = ~13GB). Avoid running Phase 3 + Phase 4 simultaneously (would exceed 16GB).

---

## VirtualBox Network Setup Commands

### Create NAT Networks:
```powershell
# Attack network
VBoxManage natnetwork add --netname attack-net --network "10.0.1.0/24" --enable --dhcp on

# Corporate/AD network
VBoxManage natnetwork add --netname corp-net --network "10.0.2.0/24" --enable --dhcp on

# SOC network
VBoxManage natnetwork add --netname soc-net --network "10.0.3.0/24" --enable --dhcp on
```

### Create Host-Only Network (Malware Lab):
```powershell
# Create host-only adapter
VBoxManage hostonlyif create
VBoxManage hostonlyif ipconfig vboxnet0 --ip 192.168.56.1 --netmask 255.255.255.0

# Disable DHCP on host-only network (isolation!)
VBoxManage dhcpserver remove --netname HostInterfaceNetworking-vboxnet0
```

---

## Host SSH Management Network

All VMs get a NIC attached to the host-only network `vboxnet0` (192.168.56.0/24) for SSH management AND cross-phase connectivity. Phase 3 malware analysis VMs (FLARE-VM, REMnux, Victim) also use this same network for isolation (no internet access).

### Static IP Assignments

| VM | Phase | IP | SSH Alias |
|----|-------|----|-----------|
| Kali Linux | 1 | 192.168.56.10 | `ssh kali` |
| Metasploitable 2 | 1 | 192.168.56.20 | `ssh ms2` |
| Win Server 2022 DC | 2 | 192.168.56.30 | `ssh dc01` |
| Win 11 Pro (AD) | 2 | 192.168.56.40 | `ssh win11` |
| REMnux | 3 | 192.168.56.50 | `ssh remnux` |
| FLARE-VM | 3 | 192.168.56.60 | `ssh flarevm` |
| Win11 Victim | 3 | 192.168.56.70 | `ssh victim` |

### SSH Config (`~/.ssh/config`)

```
Host kali
    HostName 192.168.56.10
    User zenx

Host ms2
    HostName 192.168.56.20
    User msfadmin
    HostKeyAlgorithms +ssh-rsa
    PubkeyAcceptedKeyTypes +ssh-rsa

Host dc01
    HostName 192.168.56.30
    User administrator

Host win11
    HostName 192.168.56.40
    User administrator

Host remnux
    HostName 192.168.56.50
    User remnux

Host flarevm
    HostName 192.168.56.60
    User flare

Host victim
    HostName 192.168.56.70
    User Victim
```

### Adding host-only NIC to a new VM

1. VM Settings → Network → Adapter 2 → Attach to: Host-Only → Name: `VirtualBox Host-Only Ethernet Adapter`
2. Inside the VM, configure eth1 with a static IP from 192.168.56.0/24
3. Copy SSH public key: `Get-Content $env:USERPROFILE\.ssh\id_rsa.pub | ssh user@vm-ip "mkdir -p .ssh && cat >> .ssh/authorized_keys"`
4. Add the alias to `~/.ssh/config`

### Kali (NetworkManager)
```bash
sudo nmcli connection add type ethernet con-name host-only ifname eth1 ip4 192.168.56.10/24
sudo nmcli connection up host-only
```

### Legacy Linux (e.g., Metasploitable 2)
```bash
sudo nano /etc/network/interfaces
# Add:
# auto eth1
# iface eth1 inet static
# address 192.168.56.20
# netmask 255.255.255.0
sudo ifup eth1
```

---

## Quick Start Order

1. **Install VirtualBox + Extension Pack**
   - https://www.virtualbox.org/wiki/Downloads

2. **Create NAT Networks** (commands above)

3. **Phase 1 first:** Kali Linux + Metasploitable 2
   - Practice: Nmap scanning, Metasploit, web app attacks

4. **Phase 2:** Build AD lab
   - Practice: Kerberoasting, BloodHound, lateral movement

5. **Phase 3:** Malware analysis lab
   - Practice: Static/dynamic analysis, unpacking, C2 detection

6. **Phase 4:** SOC/Blue team
   - Practice: Detection engineering, SIEM rules, log analysis

7. **Phase 5:** CTF/VulnHub rotation
   - Practice: Full penetration test methodology

---

## Essential Downloads

| Resource | URL |
|----------|-----|
| VirtualBox | https://www.virtualbox.org/wiki/Downloads |
| Kali Linux | https://www.kali.org/get-kali/ |
| Metasploitable 2 | https://sourceforge.net/projects/metasploitable/files/Metasploitable2/ |
| Metasploitable 3 | https://github.com/rapid7/metasploitable3 |
| Windows Server Eval | https://www.microsoft.com/evalcenter/evaluate-windows-server-2022 |
| Windows 10/11 ISO | https://www.microsoft.com/software-download/ |
| FLARE-VM | https://github.com/mandiant/flare-vm |
| REMnux OVA | https://docs.remnux.org/install-distro/get-virtual-appliance |
| pfSense | https://www.pfsense.org/download/ |
| Wazuh | https://wazuh.com/download/ |
| VulnHub | https://www.vulnhub.com/ |
| Exploit Education | https://exploit.education/ |

---

## Snapshot Strategy

Take snapshots at these points:
- **Clean OS install** (before any tool installation)
- **After tool installation** (before any attacks)
- **Before malware analysis** (always restore after)
- **Before AD attack practice** (to reset domain state)

Name snapshots clearly: `clean-install`, `tools-ready`, `pre-malware`, `pre-attack`

---

## Security Notes

- The malware analysis network MUST be isolated (Host-Only, no DHCP)
- Never run malware on a VM with internet access
- Never share folders between host and malware VMs
- Use INetSim on REMnux to simulate internet services for malware
- Keep all VMs updated EXCEPT malware victim VMs
- Document everything — attacks, findings, tool configurations
