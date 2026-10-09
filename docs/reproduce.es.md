# Reproducir el laboratorio documentado

[English](reproduce.md) · **Español**

Esta guía reconstruye los pasos del informe original. El laboratorio no se ha ejecutado de nuevo durante la publicación del portfolio. Los comandos son para una máquina virtual de laboratorio Ubuntu y un cliente Windows con OpenSSH. Sustituye `SERVER_IP` por la dirección de tu servidor. Genera tus propias claves; no se distribuyen las del laboratorio.

## 1. Preparar y auditar la máquina virtual

Crea una máquina Ubuntu Server en VirtualBox con 2048 MB de RAM, disco de 25 GB y adaptador de red en modo puente, según el informe. Instala OpenSSH durante la instalación del servidor. Conserva acceso a la consola y una sesión administrativa abierta mientras modificas SSH y el firewall.

```sh
hostnamectl
ip -br a
free -h
df -h
systemctl status ssh --no-pager
sudo ufw status verbose
```

Registra tu propio entorno. En el laboratorio original se documentaron Ubuntu 26.04 LTS y UFW inicialmente inactivo.

## 2. Crear un administrador

```sh
sudo adduser sysadmin
sudo usermod -aG sudo sysadmin
id sysadmin
su - sysadmin
sudo whoami
```

Elige tu propia contraseña de forma interactiva. El último comando debe mostrar `root`. Si es necesario, vuelve a la sesión administrativa existente.

## 3. Preparar la autenticación con claves

En el cliente Windows, genera una nueva clave protegida con frase de paso:

```powershell
ssh-keygen -t ed25519 -C "my-hardening-lab" -f "$HOME\.ssh\hardening_lab"
```

En el servidor:

```sh
sudo install -d -m 700 -o sysadmin -g sysadmin /home/sysadmin/.ssh
sudo touch /home/sysadmin/.ssh/authorized_keys
sudo chown sysadmin:sysadmin /home/sysadmin/.ssh/authorized_keys
sudo chmod 600 /home/sysadmin/.ssh/authorized_keys
sudo nano /home/sysadmin/.ssh/authorized_keys
```

Copia únicamente **tu propia clave pública nueva** en `authorized_keys`, mediante la consola o tu sesión de confianza. Mantén la clave privada en el cliente. Este repositorio no contiene ninguna de las dos claves.

Antes de desactivar las contraseñas, abre una segunda sesión de Windows y comprueba el acceso únicamente con clave:

```powershell
ssh -i "$HOME\.ssh\hardening_lab" -o IdentitiesOnly=yes -o PreferredAuthentications=publickey -o PasswordAuthentication=no -o KbdInteractiveAuthentication=no sysadmin@SERVER_IP
```

La frase de paso de una clave es distinta de la contraseña de una cuenta del servidor. Mantén abierta una sesión que funcione.

## 4. Aplicar y verificar la configuración SSH

Inicialmente se utilizó `99-hardening.conf`, pero `50-cloud-init.conf` proporcionaba antes un valor contradictorio para las contraseñas. La configuración que funcionó se guardó en `/etc/ssh/sshd_config.d/01-hardening.conf`:

```text
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
PubkeyAuthentication yes
AllowUsers sysadmin
```

Escribe los valores con `sudo nano /etc/ssh/sshd_config.d/01-hardening.conf`. Revisa la configuración existente antes de modificarla. No crees copias contradictorias ni sobrescribas una política personalizada sin comprobarla. En el laboratorio original se renombró el archivo existente:

```sh
sudo mv /etc/ssh/sshd_config.d/99-hardening.conf /etc/ssh/sshd_config.d/01-hardening.conf
```

Ese cambio de nombre solo corresponde si previamente creaste el mismo `99-hardening.conf`. Comprueba los valores **efectivos**, además del archivo editado:

```sh
sudo sshd -t
sudo sshd -T | grep -E '^(permitrootlogin|passwordauthentication|kbdinteractiveauthentication|pubkeyauthentication|allowusers) '
sudo systemctl reload ssh
systemctl status ssh --no-pager
```

Recarga el servicio únicamente cuando la validación de sintaxis sea correcta y los valores efectivos coincidan con la política prevista. Desde una segunda sesión del cliente, repite el acceso con clave y prueba el acceso solo con contraseña:

```powershell
ssh -i "$HOME\.ssh\hardening_lab" -o IdentitiesOnly=yes sysadmin@SERVER_IP
ssh -o PubkeyAuthentication=no -o PreferredAuthentications=password,keyboard-interactive sysadmin@SERVER_IP
```

El informe documenta el acceso correcto con clave y `Permission denied (publickey)` en el intento solo con contraseña.

## 5. Activar UFW conservando el acceso

```sh
sudo ufw allow 22/tcp
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable
sudo ufw status verbose
sudo ufw status numbered
```

Permite SSH antes de activar el firewall. Comprueba el acceso en una conexión nueva. Las evidencias muestran reglas de permiso para TCP 22 en IPv4 e IPv6.

## 6. Configurar Fail2ban

```sh
sudo apt update
sudo apt install fail2ban -y
fail2ban-client --version
sudo nano /etc/fail2ban/jail.d/sshd.local
```

Configuración reconstruida a partir de los parámetros documentados:

```ini
[sshd]
enabled = true
maxretry = 3
findtime = 600
bantime = 600
backend = systemd
banaction = ufw
```

```sh
sudo fail2ban-client -t
sudo systemctl enable --now fail2ban
sudo systemctl restart fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

## 7. Verificar la integración manual con el firewall

Utiliza la misma dirección reservada para documentación que en el laboratorio original. No uses la dirección real de tu cliente SSH para esta prueba.

```sh
sudo fail2ban-client set sshd banip 192.0.2.10
sudo fail2ban-client status sshd
sudo ufw status numbered
sudo fail2ban-client set sshd unbanip 192.0.2.10
sudo fail2ban-client status sshd
sudo ufw status numbered
```

El bloqueo documentado añadió una regla temporal de rechazo en UFW; el desbloqueo la eliminó. Esta prueba verifica las acciones manuales. La detección automática de intentos fallidos no se probó directamente en el informe.

## 8. Comprobar las actualizaciones y el estado final

```sh
apt list --upgradable
systemctl status unattended-upgrades --no-pager
apt-config dump | grep -E 'APT::Periodic::(Update-Package-Lists|Unattended-Upgrade)'
sudo sshd -t
sudo sshd -T | grep -E '^(permitrootlogin|passwordauthentication|kbdinteractiveauthentication|pubkeyauthentication|allowusers) '
sudo ufw status verbose
sudo fail2ban-client status sshd
systemctl is-active ssh fail2ban
```

El informe documenta la configuración periódica de APT habilitada, ambos servicios activos, ningún bloqueo actual y un bloqueo manual histórico. Se encontraron actualizaciones pendientes; no se demostró que se instalaran todas. Conserva tus propias evidencias finales y gestiona las actualizaciones según los requisitos de tu laboratorio.

[Resumen en español](../README.es.md) · [Informe original saneado](original-es-sanitized.md)
