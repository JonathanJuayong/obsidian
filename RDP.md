Remote Desktop Protocol

Created by Microsoft for remote access of desktops.

# xfreerdp Cheat Sheet

## Installation

```bash
# Debian / Ubuntu
sudo apt install freerdp2-x11

# Fedora / RHEL
sudo dnf install freerdp

# Arch Linux
sudo pacman -S freerdp
```

---

## Basic Syntax

```bash
xfreerdp /v:<host> /u:<username> /p:<password> [options]
```

---

## Connection Options

| Option | Description | Example |
|---|---|---|
| `/v:` | Target host (IP or hostname) | `/v:192.168.1.100` |
| `/port:` | RDP port (default: 3389) | `/port:3390` |
| `/u:` | Username | `/u:Administrator` |
| `/p:` | Password | `/p:MyPassword` |
| `/d:` | Domain | `/d:CORP` |

---

## Display Options

| Option | Description | Example |
|---|---|---|
| `/f` | Full screen | `/f` |
| `/w:` `/h:` | Custom resolution | `/w:1920 /h:1080` |
| `/dynamic-resolution` | Resize window dynamically | `/dynamic-resolution` |
| `/multimon` | Multi-monitor support | `/multimon` |
| `/span` | Span across monitors | `/span` |

---

## Security Options

| Option | Description |
|---|---|
| `/cert:ignore` | Ignore SSL certificate errors |
| `/cert:tofu` | Trust on first use |
| `/sec:rdp` | Force RDP security (disable NLA) |
| `/sec:tls` | Force TLS security |
| `/sec:nla` | Force NLA (default) |

---

## Feature Options

| Option | Description | Example |
|---|---|---|
| `/clipboard` | Enable clipboard sharing | `/clipboard` |
| `/drive:name,path` | Share a local folder | `/drive:shared,/home/user/docs` |
| `/sound` | Enable audio playback | `/sound` |
| `/microphone` | Enable microphone | `/microphone` |
| `/printer` | Enable printer sharing | `/printer` |
| `/usb` | Redirect USB devices | `/usb` |

---

## Common Examples

**Basic connection:**
```bash
xfreerdp /v:192.168.1.100 /u:Administrator /p:MyPassword
```

**With domain:**
```bash
xfreerdp /v:192.168.1.100 /u:myuser /p:MyPassword /d:MYDOMAIN
```

**Full screen at 1080p:**
```bash
xfreerdp /v:192.168.1.100 /u:myuser /p:MyPassword /f /w:1920 /h:1080
```

**Ignore certificate (self-signed):**
```bash
xfreerdp /v:192.168.1.100 /u:myuser /p:MyPassword /cert:ignore
```

**Custom port:**
```bash
xfreerdp /v:192.168.1.100:3390 /u:myuser /p:MyPassword
```

**Clipboard + shared folder:**
```bash
xfreerdp /v:192.168.1.100 /u:myuser /p:MyPassword /clipboard /drive:shared,/home/user/shared
```

**Prompt for password (secure):**
```bash
xfreerdp /v:192.168.1.100 /u:myuser
```

**Force RDP security (no NLA):**
```bash
xfreerdp /v:192.168.1.100 /u:myuser /p:MyPassword /sec:rdp
```

**Full-featured connection:**
```bash
xfreerdp /v:192.168.1.100 /u:myuser /p:MyPassword /d:CORP \
  /f /dynamic-resolution /clipboard /sound /cert:ignore
```

---

## Keyboard Shortcuts (in session)

| Shortcut | Action |
|---|---|
| `Ctrl + Alt + Enter` | Toggle full screen |
| `Ctrl + Alt + F12` | Grab / release keyboard |
| `Ctrl + Alt + F` | Toggle full screen (alt) |

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Certificate error | Add `/cert:ignore` or `/cert:tofu` |
| NLA auth failure | Add `/sec:rdp` to disable NLA |
| Black screen | Try `/gfx:rfx` or `/rfx` flags |
| Connection refused | Check port with `/port:XXXX` |
| Slow performance | Add `/compression` or lower `/bpp:16` |

---

## Tips

- Omit `/p:` to be prompted for the password — safer than plain text in shell history
- Use `/log-level:DEBUG` to diagnose connection issues
- Combine `/f` with `/dynamic-resolution` for the best full-screen experience
- RDP must be enabled on the Windows machine: *Settings → System → Remote Desktop*