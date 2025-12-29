# Steam Machine – Command Summary

This file contains a minimal, command-only summary of the Steam Machine setup.
All commands are listed exactly as they appear in README.md.

---

## 1. Initial system update

```bash
sudo apt update && sudo apt full-upgrade -y && sudo apt autoremove -y && sudo apt clean
```

---

## 2. GPU drivers

### Intel / AMD (Mesa)

```bash
sudo apt install mesa-vulkan-drivers vulkan-tools firmware-linux firmware-misc-nonfree
```

### NVIDIA

```bash
sudo apt install nvidia-driver nvidia-settings vulkan-tools firmware-linux firmware-misc-nonfree
```

---

## 3. Xorg + LightDM

```bash
sudo apt install xorg lightdm
```

---

## 4. Steam + Proton

```bash
sudo apt install steam gamemode
```

(Proton is enabled in Steam settings.)

---

## 5. Optional lightweight desktop (fallback)

```bash
sudo apt install openbox
```

Launch manually:

```bash
openbox
```

---

## 6. Steam Big Picture launch script

```bash
sudo nano /usr/local/bin/launch.sh
```

Contents:

```bash
#!/bin/bash
exec steam -bigpicture
```

Make executable:

```bash
sudo chmod +x /usr/local/bin/launch.sh
```

---

## 7. systemd service for auto-launch

```bash
sudo nano /etc/systemd/system/launch.service
```

Contents:

```ini
[Unit]
Description=Auto-launch Steam Big Picture
After=graphical.target

[Service]
User=USERNAME
Type=simple
Environment=DISPLAY=:0
ExecStart=/usr/local/bin/launch.sh

[Install]
WantedBy=graphical.target
```

Enable service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable launch.service
```

---

## 8. Automated daily updates (optional)

Create update script:

```bash
sudo nano /usr/local/sbin/update.sh
```

Contents:

```bash
#!/bin/bash
apt update && apt full-upgrade -y && apt autoremove -y --purge && apt clean
```

Make executable:

```bash
sudo chmod +x /usr/local/sbin/update.sh
```

Edit root crontab:

```bash
sudo crontab -e
```

Cron entry:

```cron
0 7 * * * /usr/local/sbin/update.sh >> /var/log/update.log 2>&1
```

---
