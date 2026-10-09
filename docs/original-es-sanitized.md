# Linux Server Hardening — documentación original saneada

Copia textual en español del documento `Linux_Server_Hardening_Estilo_Unificado.docx`. Conserva el contenido y resultados originales; se han sustituido direcciones del laboratorio, nombres de equipo y rutas de usuario. No incluye metadatos de Word, claves SSH ni imágenes originales sin revisar. Las capturas seleccionadas se publican por separado con recortes o redacciones. Los comandos con marcadores requieren valores propios.

[English overview](../README.md) · [Reproduction guide](reproduce.md)

ADMINISTRACIÓN DE SISTEMAS INFORMÁTICOS EN RED

LINUX SERVER

HARDENING

Implementación y auditoría de seguridad en Ubuntu Server

DOCUMENTACIÓN TÉCNICA

Ubuntu Server 26.04 LTS  |  OpenSSH  |  UFW  |  Fail2ban

Oracle VirtualBox  ·  Windows PowerShell

PROYECTO DE ADMINISTRACIÓN DE SISTEMAS Y CIBERSEGURIDAD

Octubre de 2026

ÍNDICE

Contenido del proyecto

01   Preparación y auditoría inicial3

02   Gestión de usuarios y permisos6

03   Seguridad del servicio SSH7

04   Configuración de autenticación mediante claves SSH9

05   Configuración del firewall UFW17

06   Fail2ban y actualizaciones de seguridad19

07   Auditoría final de seguridad24

08   Problemas encontrados y soluciones26

09   Conclusiones generales28

10   Posibles mejoras futuras29

Las capturas de pantalla y las verificaciones técnicas están integradas en cada capítulo.



## 1. Preparación y auditoría inicial

### 1.1 Objetivo

El objetivo de esta fase es analizar el estado inicial de un servidor Ubuntu antes de aplicar medidas de seguridad, identificando su configuración, recursos disponibles, interfaces de red y servicios activos.

### 1.2 Entorno de trabajo

Hipervisor: Oracle VirtualBox

Sistema operativo: Ubuntu 26.04 LTS

Hostname: lab-server

Usuario: labuser

Memoria RAM configurada: 2048 MB

Disco virtual: 25 GB

Adaptador de red: Modo puente

### 1.3 Comprobación del sistema operativo

Se ejecuta el comando hostnamectl para identificar el sistema operativo, la versión del kernel y el nombre del equipo.

*Captura 01: Información del sistema. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



### 1.4 Comprobación de la red

Mediante ip -br a se identifican las interfaces de red, sus estados y las direcciones IP asignadas.

Esta información es necesaria para configurar y comprobar posteriormente el acceso remoto mediante SSH.

*Captura 02: Configuración de red. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



### 1.5 Comprobación de recursos

Se utilizan los siguientes comandos:

free -h: muestra la memoria RAM disponible y utilizada.

df -h: muestra la capacidad y el uso de los sistemas de archivos.

*Captura 03: Recursos del servidor. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



### 1.6 Comprobación del servicio SSH

Se utiliza systemctl status ssh para comprobar si el servicio de acceso remoto está instalado y activo.

*Captura 04: Estado inicial del servicio SSH. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



### 1.7 Comprobación del firewall

Se ejecuta sudo ufw status verbose para conocer el estado inicial del firewall y sus posibles reglas.

*Captura 05: Estado inicial de UFW. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



### 1.8 Resultados de la auditoría inicial

Tras realizar las comprobaciones iniciales, se han obtenido los siguientes resultados:

Elemento

Resultado

Sistema operativo

Ubuntu 26.04 LTS

Kernel

Linux 7.0.0-22-generic

Virtualización

Oracle VirtualBox

Hostname

lab-server

Dirección IP

<SERVER_IP>/24

Memoria RAM

Aproximadamente 1,6 GiB

Disco

25 GB, 24 % utilizado

Servicio SSH

Activo

Firewall UFW

Inactivo

Problemas de seguridad identificados

## 1. Firewall desactivado

El firewall UFW se encuentra inactivo, por lo que será necesario establecer reglas de acceso y habilitarlo de forma controlada.

## 2. Intentos fallidos de autenticación SSH

Se han identificado registros de intentos fallidos de autenticación mediante SSH, incluyendo el uso de un nombre de usuario no válido.

Estos registros justifican la revisión de los mecanismos de autenticación y la implementación de medidas de protección adicionales.

## 3. Autenticación mediante contraseña

Los registros muestran al menos una autenticación SSH correcta mediante contraseña. Se evaluará la implementación de autenticación mediante claves públicas.

Conclusión inicial

El servidor dispone de acceso remoto mediante SSH, pero presenta oportunidades de mejora en la protección del acceso y la configuración del firewall.

En las siguientes fases se aplicarán medidas de seguridad y se realizarán pruebas para verificar su funcionamiento.

## 2. Gestión de usuarios y permisos

### 2.1 Objetivo

Aplicar buenas prácticas de gestión de usuarios en Linux, creando una cuenta administrativa y verificando sus privilegios.

### 2.2 Comprobación del usuario actual

Se ejecutan los siguientes comandos:

whoami: identifica el usuario actual.

id: muestra el UID, GID y los grupos del usuario.

groups: permite comprobar los grupos a los que pertenece.

*Captura 06: Usuario y grupos. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



### 2.3 Creación de un usuario administrador

Se crea el usuario sysadmin mediante:

sudo adduser sysadmin

Posteriormente, se añade al grupo sudo:

sudo usermod -aG sudo sysadmin

La opción -aG permite añadir el usuario a un grupo suplementario sin eliminar sus grupos existentes.

### 2.4 Verificación de privilegios

Se comprueba la pertenencia al grupo sudo mediante id sysadmin.

A continuación, se inicia sesión con el nuevo usuario y se ejecuta sudo whoami para verificar la capacidad de realizar tareas administrativas.

*Captura 07: Verificación de privilegios. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



### 2.5 Resultado

Se ha creado correctamente el usuario sysadmin con UID 1001 y se ha añadido al grupo sudo.

Mediante el comando id sysadmin se ha verificado su pertenencia al grupo administrativo.

Posteriormente, se ha iniciado sesión con su - sysadmin y se ha ejecutado sudo whoami, obteniendo como resultado root.

La prueba confirma que el nuevo usuario puede realizar tareas administrativas mediante sudo sin necesidad de iniciar sesión directamente como root.

Resultado: prueba completada correctamente.

## 3. Seguridad del servicio SSH

### 3.1 Objetivo

Reforzar la seguridad del acceso remoto al servidor mediante la revisión de la configuración SSH, la autenticación con claves públicas y la restricción de accesos.

### 3.2 Auditoría de la configuración inicial

Se utiliza el comando sshd -T para consultar los parámetros efectivos del servidor SSH.

Se comprueban las siguientes opciones:

Port: puerto de escucha del servicio.

PermitRootLogin: acceso remoto del usuario root.

PasswordAuthentication: autenticación mediante contraseña.

PubkeyAuthentication: autenticación mediante clave pública.

AllowUsers: restricción de usuarios autorizados. (Si existen)

*Captura 08: Configuración inicial del servicio SSH. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



### 3.3 Comprobación del puerto de escucha

Se utiliza el comando ss -tlnp para comprobar los puertos TCP en escucha y verificar la exposición del servicio SSH.

*Captura 09: Puerto de escucha SSH. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



### 3.4 Comprobación del estado de la cuenta y permisos SSH

Se utiliza el comando sudo passwd -S sysadmin para comprobar el estado de la contraseña de la cuenta administrativa.

A continuación, se ejecuta sudo ls -ld /home/sysadmin /home/sysadmin/.ssh para verificar la existencia, el propietario y los permisos de los directorios.

Estas comprobaciones permiten preparar la configuración de la autenticación SSH mediante claves públicas.

*Captura 10: Estado de la cuenta y permisos de los directorios SSH. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



### 3.5 Resultados de las comprobaciones

La auditoría inicial del servicio SSH muestra los siguientes resultados:

Puerto SSH: 22.

PermitRootLogin: prohibit-password.

PasswordAuthentication: yes.

PubkeyAuthentication: yes.

Direcciones de escucha: 0.0.0.0:22 y [::]:22.

Estado de la cuenta sysadmin: contraseña configurada (P).

Directorio personal: /home/sysadmin, permisos 750 y propietario sysadmin.

Directorio .ssh: no existe todavía.

**Conclusión**

El servicio SSH está activo y permite la autenticación mediante contraseña y clave pública.

La cuenta sysadmin dispone de una contraseña configurada y su directorio personal tiene permisos adecuados.

El directorio .ssh todavía no existe, por lo que será necesario crearlo y configurar el archivo authorized_keys para permitir el acceso mediante claves públicas.

Resultado: comprobaciones completadas correctamente.

## 4. Configuración de autenticación mediante claves SSH

### 4.1 Objetivo

Mejorar la seguridad del acceso remoto al servidor Ubuntu mediante la implementación de autenticación basada en claves criptográficas SSH.

Este método permite autenticar a los usuarios mediante un par de claves pública y privada, reduciendo la dependencia de las contraseñas tradicionales.

### 4.2 Generación de claves SSH en Windows

Desde Windows PowerShell se utiliza el siguiente comando:

ssh-keygen -t ed25519 -C "linux-hardening" -f "$HOME\.ssh\linux_hardening_ed25519"

Explicación del comando:

ssh-keygen: genera un par de claves SSH.

-t ed25519: utiliza el algoritmo criptográfico Ed25519.

-C "linux-hardening": añade un comentario para identificar la clave.

-f: indica la ubicación y el nombre de los archivos generados.

Se generan dos archivos:

Clave privada (linux_hardening_ed25519): permanece en el equipo cliente y debe mantenerse protegida.

Clave pública (linux_hardening_ed25519.pub): se utilizará para autorizar el acceso al servidor Ubuntu.

Se recomienda establecer una frase de contraseña (passphrase) para proteger la clave privada.

*Captura 11: Generación de claves SSH mediante PowerShell.ç — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



### 4.3 Configuración de la clave pública en Ubuntu

Objetivo

Configurar el servidor Ubuntu para permitir la autenticación SSH del usuario sysadmin mediante la clave pública generada previamente en Windows.

Creación del directorio SSH

Se ejecutan los siguientes comandos:

sudo mkdir -p /home/sysadmin/.ssh

sudo chown sysadmin:sysadmin /home/sysadmin/.ssh

sudo chmod 700 /home/sysadmin/.ssh

Estos comandos crean el directorio .ssh, asignan su propiedad al usuario sysadmin y restringen su acceso al propietario.

Instalación de la clave pública

Desde Windows PowerShell se obtiene la clave pública mediante:

Get-Content "$HOME\.ssh\linux_hardening_ed25519.pub"

La clave pública se añade al archivo /home/sysadmin/.ssh/authorized_keys del servidor Ubuntu.

Posteriormente, se configuran los permisos:

sudo chown sysadmin:sysadmin /home/sysadmin/.ssh/authorized_keys

sudo chmod 600 /home/sysadmin/.ssh/authorized_keys

Estos permisos impiden que otros usuarios modifiquen el archivo de claves autorizadas.

Verificación

Se comprueban los permisos mediante los comandos ls -ld y ls -l.

*Captura 12: Configuración y permisos del directorio .ssh y del archivo authorized_keys. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*







### 4.4 Verificación de la conexión SSH

Objetivo

Comprobar que el usuario sysadmin puede acceder remotamente al servidor Ubuntu desde Windows utilizando la clave privada Ed25519.

Procedimiento

Desde Windows PowerShell se ejecuta:

ssh -i "$HOME\.ssh\linux_hardening_ed25519" -o IdentitiesOnly=yes sysadmin@<SERVER_IP>

Una vez establecida la conexión, se utilizan los comandos whoami y hostname para comprobar la identidad del usuario y del servidor.

Posteriormente, se realiza una segunda prueba deshabilitando la autenticación por contraseña desde el cliente SSH para verificar que el acceso se realiza mediante clave pública.

Verificación

Se comprobará que la conexión se establece correctamente y que el método de autenticación utilizado es publickey.

*Captura 13: Conexión remota desde Windows. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



*Captura 14: Verificación del método de autenticación mediante clave pública. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



Resultado

Pendiente de completar tras realizar las pruebas de conexión.

### 4.5 Restricción de autenticación SSH

Objetivo

Reforzar la seguridad del servicio SSH deshabilitando los métodos de autenticación menos seguros y restringiendo el acceso remoto a usuarios autorizados.

Configuración

Se crea el archivo /etc/ssh/sshd_config.d/99-hardening.conf con los siguientes parámetros:

PermitRootLogin no: impide el acceso remoto directo del usuario root.

PasswordAuthentication no: deshabilita la autenticación mediante contraseña.

KbdInteractiveAuthentication no: deshabilita la autenticación interactiva.

PubkeyAuthentication yes: permite la autenticación mediante claves públicas.

AllowUsers sysadmin: limita el acceso SSH al usuario autorizado.

Verificación

Se utiliza sshd -t para comprobar la sintaxis de la configuración y sshd -T para consultar los parámetros efectivos.

*Captura 15: Configuración de seguridad y comprobación de parámetros SSH. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



4.5.4 Análisis de conflictos en la configuración SSH

Durante la verificación de los parámetros efectivos mediante sshd -T, se detecta que la opción PasswordAuthentication continúa establecida en yes, a pesar de haber configurado el valor no en el archivo 99-hardening.conf.

El resto de los parámetros de seguridad comprobados presenta los valores esperados.

Para investigar el problema, se utiliza el comando:

sudo grep -RnE '^[[:space:]]*(Include|PasswordAuthentication)[[:space:]]+' /etc/ssh/sshd_config /etc/ssh/sshd_config.d/

Este comando permite localizar las directivas relacionadas con la autenticación por contraseña y comprobar qué archivos pueden estar influyendo en la configuración efectiva.

*Captura 16: Análisis de los archivos de configuración SSH. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



4.5.5 Corrección del conflicto de configuración SSH

Tras analizar los archivos de configuración, se identifica que 50-cloud-init.conf establece PasswordAuthentication yes, mientras que 99-hardening.conf establece PasswordAuthentication no.

Debido al orden de lectura de los archivos de configuración SSH, el valor definido en 50-cloud-init.conf tiene prioridad.

Para solucionar el conflicto sin modificar directamente el archivo gestionado por cloud-init, se renombra el archivo de configuración de seguridad:

sudo mv /etc/ssh/sshd_config.d/99-hardening.conf /etc/ssh/sshd_config.d/01-hardening.conf

De esta forma, la configuración de seguridad se procesa antes que 50-cloud-init.conf.

Posteriormente, se comprueba la sintaxis mediante sshd -t y se consultan los parámetros efectivos mediante sshd -T.

*Captura 17: Corrección del orden de carga y verificación de la configuración SSH. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



4.5.6 Aplicación y pruebas de seguridad SSH

Una vez validada la configuración, se recarga el servicio SSH mediante:

sudo systemctl reload ssh

Posteriormente, se comprueba su estado con:

systemctl status ssh --no-pager

Para verificar la seguridad del acceso remoto, se realizan dos pruebas:

Prueba 1 — Autenticación mediante clave pública

Se intenta acceder al servidor utilizando la clave privada Ed25519 del usuario sysadmin.



Prueba 2 — Autenticación mediante contraseña

Se realiza una conexión SSH deshabilitando el uso de claves públicas desde el cliente, para comprobar que el servidor rechaza los métodos de autenticación mediante contraseña.



*Captura 18: Estado del servicio SSH  — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*





### 4.6 Resultados

Se ha completado correctamente la configuración y el endurecimiento de la seguridad del servicio SSH en Ubuntu Server.

Durante esta fase se han realizado las siguientes tareas:

Generación de un par de claves SSH Ed25519 desde Windows.

Configuración del archivo authorized_keys para el usuario sysadmin.

Asignación de permisos seguros a los archivos y directorios SSH.

Verificación del acceso remoto mediante autenticación por clave pública.

Deshabilitación de la autenticación mediante contraseña.

Restricción del acceso SSH al usuario sysadmin.

Deshabilitación del acceso remoto directo del usuario root.

Identificación y resolución de un conflicto de configuración con 50-cloud-init.conf.

Tras aplicar los cambios, se ha comprobado que el servicio SSH continúa activo y funcionando correctamente.

Las pruebas realizadas confirman que el acceso mediante clave pública está permitido, mientras que los intentos de autenticación mediante contraseña son rechazados.

**Conclusión**

El servidor dispone ahora de una configuración SSH más segura, basada en autenticación mediante claves públicas y restricciones de acceso.

Además, se ha resuelto un conflicto relacionado con el orden de carga de los archivos de configuración, verificando finalmente que los parámetros de seguridad establecidos se aplican correctamente.

## 5. Configuración del firewall UFW

### 5.1 Objetivo

Configurar el firewall UFW en Ubuntu Server para controlar el tráfico de red, bloquear conexiones entrantes no autorizadas y permitir únicamente los servicios necesarios.

### 5.2 Comprobación del estado inicial

Se utiliza el comando sudo ufw status verbose para comprobar el estado inicial del firewall y las reglas existentes.

### 5.3 Configuración de reglas

Se permite el acceso al servicio SSH mediante:

sudo ufw allow 22/tcp

Posteriormente, se establecen las políticas predeterminadas:

sudo ufw default deny incoming

sudo ufw default allow outgoing

Estas políticas permiten bloquear las conexiones entrantes no autorizadas y mantener las conexiones salientes del servidor.

*Captura 21: Configuración de la regla de acceso SSH. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



### 5.4 Activación del firewall

Se activa el firewall mediante:

sudo ufw enable

A continuación, se comprueba su estado y las reglas configuradas:

sudo ufw status verbose

sudo ufw status numbered

*Captura 22: Estado y reglas del firewall UFW. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



### 5.5 Verificación de conectividad

Se realiza una conexión SSH desde Windows para comprobar que el acceso remoto continúa funcionando después de activar el firewall.

*Captura 23: Verificación de conectividad SSH con UFW activo. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



### 5.6 Resultados

Se ha configurado y activado correctamente el firewall UFW en el servidor Ubuntu.

Se han aplicado las siguientes medidas de seguridad:

Activación del firewall UFW.

Configuración de la política predeterminada deny para conexiones entrantes.

Configuración de la política predeterminada allow para conexiones salientes.

Creación de reglas para permitir el tráfico SSH mediante el puerto 22/TCP, tanto en IPv4 como en IPv6.

Habilitación del firewall durante el arranque del sistema.

Tras aplicar las reglas, se ha verificado que UFW se encuentra activo y que las políticas configuradas funcionan correctamente.

Además, se ha realizado una conexión SSH desde Windows utilizando la clave privada Ed25519 del usuario sysadmin, comprobando que el acceso remoto continúa funcionando después de activar el firewall.

**Conclusión**

El servidor dispone ahora de un firewall activo que bloquea por defecto las conexiones entrantes no autorizadas y permite el acceso al servicio SSH.

La configuración reduce la exposición de los servicios de red y mantiene la administración remota segura mediante autenticación por clave pública.

## 6. Fail2ban y actualizaciones de seguridad

### 6.1 Objetivo

Reforzar la seguridad del servidor Ubuntu mediante la implementación de Fail2ban para proteger el servicio SSH frente a intentos repetidos de autenticación fallida y comprobar la configuración de las actualizaciones de seguridad.

### 6.2 Instalación de Fail2ban

Se actualiza el índice de paquetes mediante:

sudo apt update

Posteriormente, se instala Fail2ban:

sudo apt install fail2ban -y

Se comprueba la versión instalada y el estado del servicio mediante fail2ban-client --version y systemctl status fail2ban.

*Captura 24: Instalación y comprobación de Fail2ban. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



### 6.3 Configuración de protección SSH

Se crea el archivo /etc/fail2ban/jail.d/sshd.local para configurar la protección del servicio SSH.

Se establecen las siguientes condiciones:

Máximo de 3 intentos fallidos.

Intervalo de detección de 600 segundos.

Duración del bloqueo de 600 segundos.

Lectura de registros mediante systemd.

Integración con UFW para aplicar las restricciones.

Se valida la configuración y se comprueba el estado del jail sshd mediante fail2ban-client.

*Captura 25: Configuración y estado del jail SSH. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*



### 6.4 Comprobación de actualizaciones de seguridad

Se comprueban los paquetes actualizables mediante apt list --upgradable.

También se revisa el estado de unattended-upgrades y la configuración de actualizaciones periódicas de APT.

*Captura 26: Comprobación de actualizaciones de seguridad. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*







### 6.5 Prueba controlada de bloqueo mediante Fail2ban

Se realiza una prueba controlada para verificar la integración entre Fail2ban y UFW, utilizando una dirección IP reservada para documentación.

Se ejecuta:

sudo fail2ban-client set sshd banip 192.0.2.10

Posteriormente, se comprueba el estado del jail SSH y las reglas del firewall mediante:

sudo fail2ban-client status sshd

sudo ufw status numbered

Finalmente, se elimina el bloqueo de prueba mediante:

sudo fail2ban-client set sshd unbanip 192.0.2.10

Esta prueba permite comprobar el mecanismo de bloqueo manual y su integración con el firewall, sin generar intentos reales de autenticación fallida.

*Captura 27: Prueba de bloqueo y desbloqueo mediante Fail2ban. — captura original omitida; véase la selección revisada en [screenshots](../screenshots/README.md).*





### 6.6 Resultados

Se ha instalado y configurado correctamente Fail2ban en Ubuntu Server para reforzar la protección del servicio SSH.

Se ha habilitado el jail sshd, configurando un máximo de tres intentos fallidos en un intervalo de diez minutos y un bloqueo temporal de diez minutos, utilizando UFW como mecanismo de bloqueo.

La configuración ha superado las pruebas de validación y el servicio Fail2ban se encuentra activo.

Para comprobar su funcionamiento, se ha realizado una prueba controlada utilizando la dirección IP 192.0.2.10.

Tras ejecutar el bloqueo manual, Fail2ban ha registrado la dirección IP en su lista de bloqueos y UFW ha generado automáticamente una regla REJECT IN para dicha dirección.

Posteriormente, se ha eliminado el bloqueo, verificando que la dirección IP desaparece de la lista de Fail2ban y que la regla temporal de UFW se elimina correctamente.

Esta prueba confirma el funcionamiento de las acciones de bloqueo y desbloqueo y su integración con UFW. La detección automática de intentos fallidos no se ha probado directamente.

Por otra parte, se han identificado paquetes pendientes de actualización y se ha comprobado que el sistema tiene configuradas las actualizaciones automáticas mediante unattended-upgrades.

**Conclusión**

El servidor dispone de una capa adicional de protección SSH mediante Fail2ban, integrada con el firewall UFW.

Las pruebas realizadas demuestran que el sistema puede aplicar y retirar restricciones de acceso de forma controlada, complementando las medidas de seguridad implementadas anteriormente.

## 7. Auditoría final de seguridad

### 7.1 Objetivo

Realizar una auditoría final del servidor Ubuntu para comprobar que las medidas de seguridad implementadas durante el proyecto se encuentran correctamente configuradas y operativas.

### 7.2 Verificación del sistema

Se utiliza hostnamectl para identificar el sistema operativo, la versión del kernel y el entorno de virtualización.



### 7.3 Verificación de SSH

Se comprueba la sintaxis de la configuración SSH mediante sshd -t y se revisan los parámetros efectivos de autenticación mediante sshd -T.



### 7.4 Verificación del firewall

Se utiliza ufw status verbose para comprobar que el firewall está activo y que las reglas de acceso SSH se mantienen correctamente configuradas.



### 7.5 Verificación de Fail2ban

Se comprueba el estado del jail sshd, las direcciones IP bloqueadas y el estado de los servicios de seguridad.



### 7.6 Resultados

Se ha realizado una auditoría final de seguridad del servidor Ubuntu 26.04 LTS, ejecutado en un entorno virtualizado mediante Oracle VirtualBox.

Las comprobaciones realizadas confirman los siguientes resultados:

Sistema operativo: Ubuntu 26.04 LTS, arquitectura x86-64.

Seguridad SSH: autenticación mediante clave pública habilitada, autenticación por contraseña deshabilitada y acceso remoto limitado al usuario sysadmin.

Acceso root: inicio de sesión remoto directo deshabilitado.

Firewall UFW: activo, con política predeterminada de denegación de conexiones entrantes y permiso para el servicio SSH en el puerto 22/TCP.

Fail2ban: servicio activo y jail sshd habilitado.

Bloqueos: ninguna dirección IP bloqueada actualmente. Se registra un bloqueo histórico correspondiente a la prueba controlada.

Servicios: SSH y Fail2ban se encuentran activos y funcionando correctamente.

La comprobación de la configuración SSH mediante sshd -t no ha detectado errores de sintaxis.

Asimismo, las pruebas realizadas durante el proyecto han confirmado que la autenticación mediante clave pública permite el acceso remoto, mientras que la autenticación mediante contraseña es rechazada.

También se ha verificado la integración entre Fail2ban y UFW mediante una prueba controlada de bloqueo y desbloqueo de una dirección IP.

**Conclusión**

El proyecto Linux Server Hardening se ha completado satisfactoriamente, implementando medidas de seguridad relacionadas con la gestión de usuarios, la autenticación SSH, el control del tráfico de red y la protección frente a intentos repetidos de acceso.

La auditoría final confirma que las principales configuraciones de seguridad se encuentran aplicadas y que los servicios necesarios permanecen operativos.

Este proyecto ha permitido aplicar conocimientos prácticos de administración de sistemas Linux, seguridad de redes, configuración de servicios y resolución de problemas.

## 8. Problemas encontrados y soluciones

Durante el desarrollo del proyecto se han identificado diferentes situaciones relacionadas con la configuración y seguridad del servidor. A continuación, se describen los problemas más relevantes y sus soluciones.

### 8.1 Conflicto en la configuración SSH

Problema:

Después de configurar PasswordAuthentication no en el archivo 99-hardening.conf, la comprobación mediante sshd -T seguía mostrando PasswordAuthentication yes.

Causa:

El archivo 50-cloud-init.conf establecía PasswordAuthentication yes y se procesaba antes que 99-hardening.conf.

Debido al orden de lectura de los archivos de configuración de OpenSSH, la primera directiva encontrada tenía prioridad.

Solución:

Se renombró el archivo personalizado para que se cargara antes que la configuración de cloud-init:

sudo mv /etc/ssh/sshd_config.d/99-hardening.conf /etc/ssh/sshd_config.d/01-hardening.conf

Posteriormente, se verificó la sintaxis mediante sshd -t y se confirmó con sshd -T que el valor efectivo era PasswordAuthentication no.

Resultado: conflicto resuelto correctamente.

### 8.2 Directorio SSH inexistente

Situación detectada:

Durante la comprobación inicial del usuario sysadmin, se observó que el directorio /home/sysadmin/.ssh todavía no existía.

Solución:

Se creó el directorio y se configuraron los permisos necesarios:

Directorio .ssh: permisos 700.

Archivo authorized_keys: permisos 600.

Propietario: sysadmin.

Esta configuración permitió preparar el acceso remoto mediante claves públicas.

Resultado: estructura de autenticación SSH configurada correctamente.

### 8.3 Protección del acceso remoto durante los cambios

Riesgo identificado:

La modificación de los parámetros de autenticación SSH o de las reglas del firewall podía provocar la pérdida del acceso remoto al servidor.

Medidas preventivas:

Mantener una sesión SSH abierta durante los cambios.

Verificar la autenticación mediante clave pública antes de deshabilitar las contraseñas.

Validar la configuración SSH antes de recargar el servicio.

Permitir el puerto 22/TCP antes de activar UFW.

Realizar nuevas pruebas de conexión después de aplicar los cambios.

Resultado: se mantuvo el acceso remoto durante la configuración y las pruebas.

### 8.4 Verificación de bloqueos de Fail2ban

Situación detectada:

Tras configurar Fail2ban, el jail sshd aparecía activo, pero no había direcciones IP bloqueadas.

Esta situación no representaba un error, ya que no se habían registrado eventos que provocaran un bloqueo.

Comprobación realizada:

Se utilizó una dirección IP reservada para documentación (192.0.2.10) para realizar una prueba controlada de bloqueo manual.

Se verificó que Fail2ban añadía una regla temporal de rechazo en UFW.

Posteriormente, se eliminó el bloqueo y se comprobó que la regla desaparecía correctamente.

Resultado: integración entre Fail2ban y UFW verificada mediante bloqueo y desbloqueo manual.

La detección automática de intentos fallidos no se comprobó directamente.

## 9. Conclusiones generales

El proyecto Linux Server Hardening ha permitido aplicar de forma práctica diferentes técnicas de administración y seguridad sobre un servidor Ubuntu 26.04 LTS virtualizado mediante Oracle VirtualBox.

A lo largo del proyecto se ha realizado una auditoría inicial del sistema, se han gestionado usuarios y permisos, se ha reforzado la autenticación SSH mediante claves Ed25519 y se han establecido restricciones para impedir el acceso remoto mediante contraseña y el inicio de sesión directo del usuario root.

Además, se ha configurado el firewall UFW con una política de denegación de conexiones entrantes y se ha implementado Fail2ban como mecanismo adicional de protección del servicio SSH.

Las pruebas realizadas han permitido comprobar el funcionamiento del acceso remoto mediante claves públicas, el rechazo de la autenticación por contraseña y la integración de Fail2ban con UFW para aplicar y retirar bloqueos de direcciones IP.

Uno de los aspectos más relevantes del proyecto ha sido la resolución de un conflicto entre archivos de configuración SSH, lo que ha permitido comprender mejor el funcionamiento y el orden de prioridad de las directivas del servicio.

Desde el punto de vista formativo, este proyecto ha contribuido al desarrollo de competencias relacionadas con:

Administración de sistemas Linux.

Gestión de usuarios, grupos y permisos.

Configuración y protección de servicios de red.

Autenticación criptográfica mediante SSH.

Configuración de firewalls y mecanismos de control de acceso.

Análisis de registros y resolución de problemas.

Verificación y documentación de configuraciones de seguridad.

En conclusión, se han alcanzado los objetivos principales del proyecto y se ha conseguido una configuración del servidor más segura que la inicial.

No obstante, el endurecimiento de un servidor es un proceso continuo que requiere actualizaciones, supervisión y revisiones periódicas. Las pruebas realizadas verifican las medidas implementadas, pero no constituyen una auditoría exhaustiva de vulnerabilidades.

## 10. Posibles mejoras futuras

Aunque se han implementado las principales medidas de seguridad previstas, el proyecto puede ampliarse mediante las siguientes mejoras:

## 1. Auditoría de seguridad con Lynis

Utilizar Lynis para analizar la configuración del sistema y obtener recomendaciones adicionales de endurecimiento.

## 2. Monitorización y centralización de registros

Implementar herramientas como Wazuh para supervisar eventos de seguridad, detectar comportamientos sospechosos y centralizar los registros.

## 3. Automatización mediante Ansible

Crear playbooks para aplicar automáticamente las configuraciones de seguridad y facilitar su reproducción en otros servidores.

## 4. Copias de seguridad automatizadas

Implementar scripts de backup y procedimientos de restauración para proteger los archivos de configuración y los datos importantes.

## 5. Restricción del acceso SSH por dirección IP

Limitar el acceso al puerto SSH a direcciones o redes administrativas autorizadas cuando el entorno de despliegue lo permita.

## 6. Pruebas de seguridad adicionales

Realizar una auditoría con herramientas de análisis de puertos y configuración, así como pruebas controladas de detección automática de intentos fallidos de autenticación.
