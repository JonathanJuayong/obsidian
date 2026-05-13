Open-sourced implementation of [[SMB]] that allows file sharing between Linux/unix and other operating systems.

# Samba File Sharing Setup Guide for Linux

## Prerequisites

- A Linux system with `sudo` privileges
- Basic familiarity with the terminal

---

## 1. Install Samba

**Debian/Ubuntu:**

```bash
sudo apt update && sudo apt install samba -y
```

**RHEL/CentOS/Fedora:**

```bash
sudo dnf install samba samba-common samba-client -y
```

---

## 2. Create a Shared Directory

```bash
sudo mkdir -p /srv/samba/shared
sudo chmod 777 /srv/samba/shared        # Permissive; tighten as needed
sudo chown nobody:nogroup /srv/samba/shared
```

---

## 3. Configure `/etc/samba/smb.conf`

Open the config file:

```bash
sudo nano /etc/samba/smb.conf
```

Add a share definition at the bottom:

```ini
[global]
   workgroup = WORKGROUP
   server string = Samba Server
   security = user
   map to guest = Bad User        # Remove this line for auth-only access

[shared]
   path = /srv/samba/shared
   browseable = yes
   read only = no
   guest ok = yes                 # Set to "no" for password-protected shares
   create mask = 0755
```

Test the config for syntax errors:

```bash
testparm
```

---

## 4. Add a Samba User (Password-Protected Shares)

The user must already exist as a Linux system user:

```bash
sudo useradd -M -s /sbin/nologin sambauser   # Create system-only user
sudo smbpasswd -a sambauser                  # Set Samba password
sudo smbpasswd -e sambauser                  # Enable the account
```

Then update the share section to require authentication:

```ini
[shared]
   ...
   guest ok = no
   valid users = sambauser
```

---

## 5. Start and Enable Samba

```bash
sudo systemctl enable --now smbd nmbd
```

---

## 6. Open the Firewall

**UFW (Ubuntu):**

```bash
sudo ufw allow samba
```

**firewalld (RHEL/Fedora):**

```bash
sudo firewall-cmd --permanent --add-service=samba
sudo firewall-cmd --reload
```

---

## 7. Connect from Clients

|Operating System|Address|
|---|---|
|Windows|`\\<server-ip>\shared` in File Explorer|
|macOS|`smb://<server-ip>/shared` via Finder → Go → Connect to Server|
|Linux|`smb://<server-ip>/shared` in file manager, or via `smbclient`|

**Linux CLI connection:**

```bash
smbclient //server-ip/shared -U sambauser
```

---

## Useful Commands

|Command|Description|
|---|---|
|`sudo systemctl restart smbd`|Restart after config changes|
|`sudo smbstatus`|View active connections|
|`sudo pdbedit -L`|List all Samba users|
|`testparm -s`|Show effective configuration|

---

## Tips & Notes

> **SELinux (RHEL/Fedora):** Run the following if SELinux is enforcing:
> 
> ```bash
> sudo setsebool -P samba_enable_home_dirs on
> chcon -t samba_share_t /srv/samba/shared
> ```

- For home directory shares, uncomment the `[homes]` section already present in `smb.conf`
- Always restart `smbd` after editing `smb.conf`
- Use `testparm` to validate your config before restarting the service

