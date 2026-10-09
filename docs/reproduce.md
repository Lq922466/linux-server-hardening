# Reproduce the documented lab

This guide reconstructs the steps recorded in the Spanish report. It has not been rerun during portfolio publication. Commands below are for a disposable Ubuntu lab VM, with a Windows OpenSSH client. Replace `SERVER_IP` with your own server address. Create your own keys; no lab key is distributed.

## 1. Prepare and audit the VM

Create an Ubuntu Server VM in VirtualBox with 2,048 MB RAM, a 25 GB disk and a bridged network adapter, as recorded in the report. Install OpenSSH during server installation. Retain console access and an existing administrative session throughout SSH and firewall changes.

```sh
hostnamectl
ip -br a
free -h
df -h
systemctl status ssh --no-pager
sudo ufw status verbose
```

Record your own environment; the original lab recorded Ubuntu 26.04 LTS and initially inactive UFW.

## 2. Create an administrator

```sh
sudo adduser sysadmin
sudo usermod -aG sudo sysadmin
id sysadmin
su - sysadmin
sudo whoami
```

Choose your own password interactively. The last command should print `root`. Return to the existing admin session if needed.

## 3. Prepare key authentication

On the Windows client, generate a new passphrase-protected key:

```powershell
ssh-keygen -t ed25519 -C "my-hardening-lab" -f "$HOME\.ssh\hardening_lab"
```

On the server:

```sh
sudo install -d -m 700 -o sysadmin -g sysadmin /home/sysadmin/.ssh
sudo touch /home/sysadmin/.ssh/authorized_keys
sudo chown sysadmin:sysadmin /home/sysadmin/.ssh/authorized_keys
sudo chmod 600 /home/sysadmin/.ssh/authorized_keys
sudo nano /home/sysadmin/.ssh/authorized_keys
```

Copy only **your own new public key** into `authorized_keys` through the server console or your existing trusted session. Keep the private key on your client. This repository contains neither key.

Before disabling passwords, open a second Windows session and verify key-only access:

```powershell
ssh -i "$HOME\.ssh\hardening_lab" -o IdentitiesOnly=yes -o PreferredAuthentications=publickey -o PasswordAuthentication=no -o KbdInteractiveAuthentication=no sysadmin@SERVER_IP
```

A key passphrase prompt is different from a server account password prompt. Keep a working session open.

## 4. Apply and verify SSH settings

The lab initially used `99-hardening.conf`; `50-cloud-init.conf` supplied a conflicting password setting first. The successful configuration used `/etc/ssh/sshd_config.d/01-hardening.conf`:

```text
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
PubkeyAuthentication yes
AllowUsers sysadmin
```

Use `sudo nano /etc/ssh/sshd_config.d/01-hardening.conf` to write the settings. Inspect existing configuration before changing it. Do not create competing copies or overwrite an existing custom policy blindly. In the original lab, the existing file was renamed with:

```sh
sudo mv /etc/ssh/sshd_config.d/99-hardening.conf /etc/ssh/sshd_config.d/01-hardening.conf
```

That rename is relevant only if you previously created the same `99-hardening.conf`. Check the **effective** settings, not just the file you edited:

```sh
sudo sshd -t
sudo sshd -T | grep -E '^(permitrootlogin|passwordauthentication|kbdinteractiveauthentication|pubkeyauthentication|allowusers) '
sudo systemctl reload ssh
systemctl status ssh --no-pager
```

Proceed with reload only after syntax validation succeeds and effective settings match the intended policy. From a second client session, repeat key login and test password-only login:

```powershell
ssh -i "$HOME\.ssh\hardening_lab" -o IdentitiesOnly=yes sysadmin@SERVER_IP
ssh -o PubkeyAuthentication=no -o PreferredAuthentications=password,keyboard-interactive sysadmin@SERVER_IP
```

The report records successful key login and `Permission denied (publickey)` for the password-only attempt.

## 5. Enable UFW while preserving access

```sh
sudo ufw allow 22/tcp
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable
sudo ufw status verbose
sudo ufw status numbered
```

Allow SSH before enabling the firewall. Confirm SSH access in a new connection. The lab evidence shows IPv4 and IPv6 allow rules for TCP 22.

## 6. Configure Fail2ban

```sh
sudo apt update
sudo apt install fail2ban -y
fail2ban-client --version
sudo nano /etc/fail2ban/jail.d/sshd.local
```

Reconstructed configuration from the documented parameters:

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

## 7. Verify manual firewall integration

Use the same reserved documentation address as the original lab. Do not use your real SSH client address for this test.

```sh
sudo fail2ban-client set sshd banip 192.0.2.10
sudo fail2ban-client status sshd
sudo ufw status numbered
sudo fail2ban-client set sshd unbanip 192.0.2.10
sudo fail2ban-client status sshd
sudo ufw status numbered
```

The documented ban added a temporary UFW rejection rule; unban removed it. This tests the manual action path. Automatic detection of failed logins remains untested in the source report.

## 8. Check updates and final state

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

The report records periodic APT settings enabled, both services active, no current bans and one historical manual ban. Pending updates were found; installation of all pending updates was not established. Keep your own final evidence and handle updates according to your lab's requirements.

[English overview](../README.md) · [Sanitized Spanish report](original-es-sanitized.md)
