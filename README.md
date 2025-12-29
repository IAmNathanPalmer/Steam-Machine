# Steam Machine
Build your own Steam machine: a lightweight setup that boots straight into Steam Big Picture. Desktop optional, no bloat, just a non-Steam-branded machine with your hardware of choice. With Valve reviving the Steam Machine concept, this guide documents a clean, minimal way to build your own today.

---

### Why?
Almost two months ago Valve announced their Steam Machine, essentially a mid-sized PC with moderate hardware. Many people wonder about future capabilities and better hardware, costs, etc. However, it is more or less a software configuration which anyone can set up themselves, and I had already been planning to do this even before the announcement. This is nothing against Valve; it's for those who want to use what is basically the same software stack that Steam uses on a different hardware configuration. The costs for an actual Steam Machine haven't been confirmed yet, and anyone who already has the hardware can set it up without buying another system.

My preference is Debian. SteamOS itself is restricted, is now based on Arch Linux and does not have a general installer. You could use other Linux distributions for this as well, but for this I chose [Debian net install](https://www.debian.org/CD/netinst/) because of its conservative updates, it is lightweight, predictable without surprises, and an excellent base for long-running systems. This will turn whatever hardware into a **stable** gaming appliance. There are several commands and some scripts, it may look intimidating for someone who doesn't work with command-line but you do this once and you're set. This system will boot straight to Big Picture mode, and you can use that in place of a desktop.

# Debian CLI Steam console setup

## 1. Install Debian
- Use Debian netinst
- Choose: minimal install (no desktop)
- Partition: entire disk (ext4 or btrfs)
- Skip adding any desktop environment for now
- Follow the prompts (name, etc.)

A minimal Debian install will boot into a command-line shell.

---

## 2. Update system
Type ```sudo apt update && sudo apt full-upgrade -y && sudo apt autoremove -y && sudo apt clean``` and enter this command in your shell.

This updates package lists, installs updates (including kernels, drivers, Mesa, and firmware), removes unused dependencies, and cleans cached packages.

---

## 3. Install GPU drivers

### Intel/AMD (Mesa)
```bash
sudo apt install mesa-vulkan-drivers vulkan-tools firmware-linux firmware-misc-nonfree
```
The above provides Vulkan for Intel / AMD GPU users (Mesa stack).

### NVIDIA
```bash
sudo apt install nvidia-driver nvidia-settings vulkan-tools firmware-linux firmware-misc-nonfree
```
Use this second one if you have an Nvidia GPU.

---

## 4. Install Xorg + LightDM
```bash
sudo apt install xorg lightdm
```
Xorg is the display server (renders graphics) and LightDM is the display manager (starts Xorg and logs you in automatically).
Why Xorg instead of Wayland? Wayland requires a compositor or desktop environment, which adds complexity and conflicts with a minimal "console style" setup.

---

## 5. Install Steam + Proton
```bash
sudo apt install steam gamemode
```
This adds Steam + Proton as well as GameMode, GameMode automatically adjusts CPU governor, I/O priority, and scheduler settings when games launch. Steam supports it natively; no manual configuration is required. GameMode is optional but recommended.

Proton is enabled in Steam Settings -> Compatibility. Check "Enable Steam Play for supported titles" as well as "Enable Steam Play for all other titles."
Select the default Proton version (latest stable).

---

## 6. OPTIONAL: Lightweight desktop (fallback)
Example: Openbox  
```bash
sudo apt install openbox
```

To launch manually:
```bash
openbox
```
This is not required but if you need to close Big Picture and you prefer mouse interaction rather than command lines, it is there.
You can install any other desktop if you choose but OpenBox is minimal and lightweight if you do choose to use one.

---

## 7. Create Steam Big Picture startup script

Create:
```bash
sudo nano /usr/local/bin/launch.sh
```

Add:
```bash
#!/bin/bash
exec steam -bigpicture
```
(Control+O and Control+X to save and exit Nano).

Make executable:
```bash
sudo chmod +x /usr/local/bin/launch.sh
```

---

## 8. Create systemd service

```bash
sudo nano /etc/systemd/system/launch.service
```

Add:
```
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
Change ```USERNAME``` to the named user account you selected when installing Debian.

(Control+O and Control+X to save and exit Nano).

Enable the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable launch.service
```

---

## 9. Daily automated update (Optional)

This system is now designed to behave like a console. A daily scheduled update keeps drivers, firmware, and security fixes current without user intervention. Updates apply quietly; kernel or driver changes take effect on the next reboot. These will not force a restart.

Create the script: 
```bash
sudo nano /usr/local/sbin/update.sh
``` 

Add the contents for the update script:

```bash
#!/bin/bash
apt update && apt full-upgrade -y && apt autoremove -y --purge && apt clean
```
(Control+O and Control+X to save and exit).

Make the script executable:
```bash
sudo chmod +x /usr/local/sbin/update.sh
```

Now enter ```sudo crontab -e``` which will open your editor (Nano). Add the following line:

```
0 7 * * * /usr/local/sbin/update.sh >> /var/log/update.log 2>&1
```

(Control+O and Control+X to save and exit again).

---
## Multiplayer titles & kernel anticheat

Some multiplayer games require [kernel-level anticheat software.](https://levvvel.com/games-with-kernel-level-anti-cheat-software/) These programs run with full system privileges, giving them deep access to your system, running with the same privileged access as the kernel itself. This is a security issue, and these anticheat programs are fundamentally incompatible with Linux by design.

This is a **game developer decision**, not a limitation of Steam, Proton, or Linux. Linux supports userspace anticheat, and many games already work correctly when developers choose to enable it. Proton allows you to run games designed for Windows, and a vast majority of them will work, just as they do on the Steam Deck and will on the Steam Machine. However, popular titles like Call of Duty, Apex Legends, and others that use kernel-level anticheat simply will not be able to run.

Tools like Easy Anti-Cheat (EAC) and BattlEye do have Linux support and can function with Proton when developers opt in. However, support is optional per-game. Other ones like Riot’s Vanguard, EA’s Javelin, Activision’s Ricochet, and others give elevated privileges and simply will not run on a Linux system, regardless of Steam or Proton. 

I strongly recommend not trying to work around this and not trying to allow kernel-level access to your system. In my opinion, people shouldn't be embracing kernel-level anticheat at all. Aside from it basically being a rootkit, it allows the developers to blame the players while not actually solving anything. Allowing kernel-level access introduces serious security risks and does not meaningfully solve cheating problems. I recommend just avoiding the games that require this level of access to your system.

---

## 11. Notes
- This seems like work but most of it is just typing and entering or saving info
- This is **only done once** and this will result in a stable Steam Linux gaming system that will just start and run
- You can also add MangoHub or Gamescope if you're comfortable with these CLI tools. I left them out as they don't add much benefit in my opinion
- You can use another distro to do this including Fedora, etc. but Debian is stable and predictable
- Steam Input handles native controller support (Xbox, Playstation, Nintendo Switch), games launched through Steam need no extra support
- There are gaming distros but those are desktop PC systems, this allows you to set up your own gaming-specific system
- I also have a Plex media server that is exactly this but with Plex instead of Steam. The system barely uses resources and allows maximum uptime
- If you do need to shut down entering ```sudo shutdown -r now``` will restart and ```sudo shutdown -h now``` will shut it off
- If you're worried about using a new system with command line, they are well documented and can always be looked up
- After completing this, you now have a system that uses nearly the same stack as valve and behaves like SteamOS under the hood.


