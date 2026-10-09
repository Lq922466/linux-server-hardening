# Linux Server Hardening

[English](README.md) · **Español**

**Estado: Completado** · Laboratorio de administración y seguridad de Ubuntu Server de **INNNX.**

Este proyecto documenta un ejercicio de endurecimiento completado: auditoría de un servidor virtual, creación de un administrador con sudo, protección de SSH mediante autenticación Ed25519, activación de UFW e integración de Fail2ban con el firewall. El resumen y la guía de reproducción se basan en el informe original en español y sus capturas de terminal.

## Entorno del laboratorio

| Componente | Entorno documentado |
| --- | --- |
| Servidor | Ubuntu 26.04 LTS, x86-64 |
| Hipervisor | Oracle VirtualBox |
| Recursos | 2048 MB de RAM configurada; disco virtual de 25 GB |
| Red | Adaptador en modo puente; direcciones del laboratorio omitidas |
| Cliente | Windows PowerShell y OpenSSH |
| Herramientas de seguridad | OpenSSH, UFW, Fail2ban, systemd, unattended-upgrades |

## Trabajo realizado

- Auditoría del sistema operativo, interfaces de red, almacenamiento, memoria, servicio SSH y estado inicial del firewall.
- Creación de la cuenta `sysadmin`, concesión de permisos sudo y comprobación de comandos administrativos.
- Configuración del propietario y los permisos de SSH (`700` para `.ssh` y `600` para `authorized_keys`) y verificación del acceso con Ed25519.
- Desactivación del acceso directo de root, de la autenticación por contraseña y de la autenticación interactiva de teclado; acceso SSH limitado a `sysadmin`.
- Resolución de un conflicto de prioridad en OpenSSH: `50-cloud-init.conf` prevalecía sobre el archivo posterior `99-hardening.conf`. Al renombrar el archivo personalizado como `01-hardening.conf`, se obtuvieron los valores efectivos previstos, comprobados con `sshd -t` y `sshd -T`.
- Activación de UFW con denegación de conexiones entrantes y permiso de conexiones salientes, manteniendo el acceso SSH por TCP 22 en IPv4 e IPv6; comprobación posterior del acceso remoto.
- Instalación y configuración del jail `sshd` de Fail2ban: tres intentos, intervalo de detección de 600 segundos, bloqueo de 600 segundos, backend systemd y acción UFW.
- Verificación del bloqueo y desbloqueo **manuales** con la dirección reservada para documentación `192.0.2.10`: se creó una regla temporal de rechazo en UFW y se eliminó al desbloquearla.
- Comprobación de paquetes actualizables, del servicio unattended-upgrades y de la configuración periódica de APT.
- Verificación final de la configuración y de los servicios.

## Resultados y evidencias

| Comprobación | Resultado documentado | Evidencia |
| --- | --- | --- |
| Configuración efectiva de SSH | Acceso de root, contraseñas y autenticación interactiva desactivados; claves públicas habilitadas; `sysadmin` autorizado | [Captura](screenshots/ssh-effective-settings.png) |
| Autenticación remota | Acceso con clave correcto; intento de acceso solo con contraseña rechazado | [Informe saneado](docs/original-es-sanitized.md#46-resultados) |
| UFW | Activo; denegación de entrada predeterminada; SSH permitido | [Captura](screenshots/ufw-final-status.png) |
| Fail2ban → UFW | El bloqueo manual generó una regla de rechazo; el desbloqueo la eliminó | [Bloqueo](screenshots/fail2ban-manual-ban.png) · [Desbloqueo](screenshots/fail2ban-manual-unban.png) |
| Estado final del jail | Ningún bloqueo actual; un bloqueo manual histórico | [Captura](screenshots/fail2ban-final-jail.png) |

La detección automática de intentos fallidos de autenticación **no se probó directamente**. Se identificaron actualizaciones pendientes; el informe no demuestra que se instalaran todas. «Completado» se refiere al alcance del laboratorio documentado, no a una auditoría exhaustiva de vulnerabilidades ni a una certificación de seguridad para producción.

## Documentación

- Guía de reproducción: [Español](docs/reproduce.es.md) · [English](docs/reproduce.md) — pasos reconstruidos a partir del trabajo documentado; utiliza tu propio equipo y tus propias credenciales.
- [Informe original saneado (español)](docs/original-es-sanitized.md) — texto extraído del documento Word revisado.
- [Selección de capturas revisadas](screenshots/README.md#español).
- [Notas de publicación y saneamiento](docs/publication-notes.es.md).

## Posibles mejoras

El informe propone auditorías con Lynis, monitorización con Wazuh, automatización con Ansible, copias de seguridad automáticas, restricciones de las direcciones de origen para SSH y pruebas de detección automática de intentos fallidos. Son mejoras futuras y no se presentan como trabajo completado.

[Volver a Infra & Security](https://github.com/Lq922466/Lq922466/blob/main/portfolio/infra-security.md) · [Perfil de INNNX.](https://github.com/Lq922466)
