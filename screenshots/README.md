# Reviewed lab evidence

These are crops of original screenshots, not recreated terminal output. Command prompts, host identifiers and client paths were removed. No SSH key material is included.

| Evidence | What it establishes |
| --- | --- |
| [Effective SSH settings](ssh-effective-settings.png) | Root/password/keyboard-interactive login disabled; public-key authentication enabled; `sysadmin` allowed. |
| [Final UFW status](ufw-final-status.png) | Active firewall, deny incoming/allow outgoing, SSH allowed over IPv4 and IPv6. |
| [Manual ban](fail2ban-manual-ban.png) | Documentation address `192.0.2.10` manually banned; UFW reject rule created. |
| [Manual unban](fail2ban-manual-unban.png) | Test ban removed and SSH allow rules retained. |
| [Final jail](fail2ban-final-jail.png) | Zero current bans and one historical manual ban. |

The manual test does not demonstrate automatic failed-login detection. Key-generation, authorized-key, network-address and machine-ID screenshots were excluded.
