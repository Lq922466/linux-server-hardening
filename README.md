# Linux Server Hardening

**Status: Completed** · An Ubuntu Server administration and security lab by **INNNX.**

This project documents a completed hardening exercise: auditing a virtual server, creating a sudo administrator, securing SSH with Ed25519 authentication, enabling UFW, and integrating Fail2ban with the firewall. The English summary and reproduction guide are based on the supplied Spanish lab report and its terminal screenshots.

## Lab environment

| Component | Recorded environment |
| --- | --- |
| Server | Ubuntu 26.04 LTS, x86-64 |
| Hypervisor | Oracle VirtualBox |
| Resources | 2,048 MB configured RAM; 25 GB virtual disk |
| Networking | Bridged adapter; lab addresses omitted |
| Client | Windows PowerShell and OpenSSH |
| Security tools | OpenSSH, UFW, Fail2ban, systemd, unattended-upgrades |

## Work completed

- Audited the operating system, network interfaces, storage, memory, SSH service and initial firewall state.
- Created the `sysadmin` account, granted sudo access and verified administrative commands.
- Prepared SSH directory ownership and permissions (`700` for `.ssh`, `600` for `authorized_keys`) and verified Ed25519 login.
- Disabled direct root login, password login and keyboard-interactive authentication; limited SSH access to `sysadmin`.
- Resolved an OpenSSH configuration precedence issue: `50-cloud-init.conf` overrode a later `99-hardening.conf`. Renaming the custom file to `01-hardening.conf` produced the intended effective settings, verified using `sshd -t` and `sshd -T`.
- Enabled UFW with deny incoming / allow outgoing defaults and SSH access on TCP 22 for IPv4 and IPv6; checked that remote access still worked.
- Installed and configured the Fail2ban `sshd` jail with three attempts, a 600-second detection window, a 600-second ban, the systemd backend and the UFW action.
- Verified **manual** ban/unban integration using documentation address `192.0.2.10`: a temporary UFW rejection rule appeared and was removed after unbanning.
- Checked available package updates, the unattended-upgrades service and APT periodic settings.
- Performed final configuration and service checks.

## Results and evidence

| Check | Recorded result | Evidence |
| --- | --- | --- |
| Effective SSH settings | Root/password/keyboard-interactive login disabled; public keys enabled; `sysadmin` allowed | [Screenshot](screenshots/ssh-effective-settings.png) |
| Remote authentication | Key login succeeded; password-only attempt rejected | [Sanitized Spanish report](docs/original-es-sanitized.md#46-resultados) |
| UFW | Active; default incoming deny; SSH permitted | [Screenshot](screenshots/ufw-final-status.png) |
| Fail2ban → UFW | Manual ban generated a rejection rule; unban removed it | [Ban](screenshots/fail2ban-manual-ban.png) · [Unban](screenshots/fail2ban-manual-unban.png) |
| Final jail | Zero current bans; one historical manual test ban | [Screenshot](screenshots/fail2ban-final-jail.png) |

Automatic failed-login detection was **not directly tested**. Pending package updates were identified; the report does not establish that every pending update was installed. “Completed” refers to the documented lab scope, not an exhaustive vulnerability audit or a production security certification.

## Documentation

- [Reproduce the lab](docs/reproduce.md) — steps reconstructed from the documented work; use your own machine and credentials.
- [Sanitized original report (Spanish)](docs/original-es-sanitized.md) — text exported from the reviewed Word report.
- [Reviewed screenshot selection](screenshots/README.md).
- [Publication and sanitization notes](docs/publication-notes.md).

## Possible next steps

The report proposes Lynis audits, Wazuh monitoring, Ansible automation, automated backups, restricted SSH source addresses and automatic failed-login detection tests. These are future improvements and are not presented as completed work.

[Back to Infra & Security](https://github.com/Lq922466/Lq922466/blob/main/portfolio/infra-security.md) · [INNNX. profile](https://github.com/Lq922466)
