# AGENTS.md — CS Home Lab

This repo is a **cybersecurity home lab** (VirtualBox-based) — not a software project.

## Structure

- `lab/README.md` — full architecture guide (5 phases, network design, RAM budget)
- `lab/downloads.md` — download links for every VM/ISO/OVA
- `lab/resources/` — downloaded ISOs, OVAs, VM images, and tools

## Conventions

- **No git repo yet.** Initialize with `git init` before making commits.
- When adding new VM guides or lab exercises, document them as `.md` files in `lab/` following the existing format.
- Place downloaded binaries/ISOs in `lab/resources/`.
- `@RTK.md` is referenced by the user at session start but does not exist in this repo — treat it as external context the user provides.

## VMs Created

| VM | Phase | IP (attack-net) | IP (corp-net) | IP (host-only) | SSH Alias | Credentials |
|----|-------|-----------------|--------------|----------------|-----------|-------------|
| Kali Linux | 1 | 10.0.1.3 | — | 192.168.56.10 | `ssh kali` | zenx/zenx |
| Metasploitable 2 | 1 | 10.0.1.4 | — | 192.168.56.20 | `ssh ms2` | msfadmin/msfadmin |
| Windows Server 2022 DC | 2 | — | 10.0.2.3 | 192.168.56.30 | `ssh dc01` | Administrator |
| Windows 11 Pro (AD) | 2 | — | 10.0.2.4 | 192.168.56.40 | `ssh win11` | Administrator |
| REMnux | 3 | — | — | 192.168.56.50 | `ssh remnux` | remnux/remnux |
| FLARE-VM | 3 | — | — | 192.168.56.60 | `ssh flarevm` | flare |
| Win11 Victim | 3 | — | — | 192.168.56.70 | `ssh victim` | Victim |
| pfSense | 4 | — | 10.0.3.1 (LAN) | 192.168.56.80 | `ssh pfsense` | admin |
| Wazuh SIEM | 4 | — | 10.0.3.85 | 192.168.56.85 | `ssh wazuh` | wazuh-user/wazuh |

## Networks

- **attack-net** (10.0.1.0/24) — NAT Network, VM-to-VM traffic for pentesting/CTF (Phase 1)
- **corp-net** (10.0.2.0/24) — NAT Network, Active Directory lab (Phase 2)
- **soc-net** (10.0.3.0/24) — NAT Network, Blue Team / SOC (Phase 4)
- **vboxnet0** (192.168.56.0/24) — Host-Only, SSH management network for host-to-VM access (all phases)

## SSH Config

All VMs have NIC 2 on `vboxnet0` with static IPs. SSH config at `~/.ssh/config`:

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

Host pfsense
    HostName 192.168.56.80
    User admin

Host wazuh
    HostName 192.168.56.85
    User wazuh-user
```

## Session Log

- **20 May 2026:** Downloaded most ISOs/OVAs (Kali, Wazuh, REMnux, pfSense, Win Server 2022, Win 11, Kioptrix). Set up Phase 1 (Kali + Metasploitable 2) with dual NICs (attack-net + host-only SSH access). Created SSH config for passwordless key-based access from host.
- **23 May 2026:** Completed Phase 2 (Windows Server 2022 DC + Win 11 AD client on corp-net). Completed Phase 3: imported REMnux OVA (static IP 192.168.56.50, SSH working), installed FLARE-VM (Win 11 Pro + 24 tool categories, 100GB disk, disabled NAT NIC for isolation), created Win11 Victim (2GB RAM, host-only only, 192.168.56.70, SSH working). All VMs configured with host-only NIC on vboxnet0 for SSH management.
- **23 May 2026 (cont):** Completed Phase 4: Imported pfSense OVA (WAN: NAT, LAN: 10.0.3.1/24 on soc-net, OPT1: 192.168.56.80/24 for SSH). Configured NAT networks (attack-net, corp-net, soc-net). Imported Wazuh 4.14.5 OVA with 4GB RAM, 2 CPUs, configured on soc-net (10.0.3.85/24) with host-only SSH access (192.168.56.85). Wazuh dashboard accessible at https://192.168.56.85.

## Workflow

- All lab practice happens inside VMs launched via VirtualBox, not in this directory.
- This repo tracks the lab blueprint, documentation, and resource downloads only.
