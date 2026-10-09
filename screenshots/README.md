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

## Español

Estas imágenes son recortes de capturas originales; no se ha recreado la salida del terminal. Se eliminaron los indicadores de terminal, los identificadores de equipo y las rutas del cliente. No contienen claves SSH.

| Evidencia | Qué demuestra |
| --- | --- |
| [Configuración efectiva de SSH](ssh-effective-settings.png) | Acceso de root, contraseñas y autenticación interactiva desactivados; claves públicas habilitadas; `sysadmin` autorizado. |
| [Estado final de UFW](ufw-final-status.png) | Firewall activo; denegación de entrada y permiso de salida; SSH permitido en IPv4 e IPv6. |
| [Bloqueo manual](fail2ban-manual-ban.png) | Bloqueo manual de la dirección de documentación `192.0.2.10` y creación de la regla de rechazo en UFW. |
| [Desbloqueo manual](fail2ban-manual-unban.png) | Eliminación del bloqueo de prueba y conservación de las reglas de permiso SSH. |
| [Estado final del jail](fail2ban-final-jail.png) | Ningún bloqueo actual y un bloqueo manual histórico. |

La prueba manual no demuestra la detección automática de intentos fallidos. Se excluyeron las capturas de generación de claves, claves autorizadas, direcciones de red e identificadores de máquina.

[English overview](../README.md) · [Resumen en español](../README.es.md)
