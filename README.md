# Steam Machine
Create your own Steam machine: a lightweight setup that boots straight into Steam Big Picture. Desktop optional, no bloat, just a non-Steam-branded machine with your hardware of choice. With Valve reviving the Steam Machine concept, this guide documents a clean, minimal way to build your own today.

---

## Why?
Almost two months ago Valve announced their Steam Machine, essentially a mid-sized PC with moderate hardware. Many people wonder about future capabilities and better hardware, costs, etc. However, it is more or less a software configuration which anyone can set up themselves, and I had already been planning to do this even before the announcement. This is nothing against Valve; it's for those who want to use what is basically the same software stack that Steam uses on a different hardware configuration. The costs for an actual Steam Machine haven't been confirmed yet, and anyone who already has the hardware can set it up without buying another system. Note that operating systems like [Bazzite](https://bazzite.gg/) exist, which basically has the same software stack already installed on a Fedora Linux distro. This is for those who want to set their distro up.

My preference is Debian. SteamOS itself is restricted, is now based on Arch Linux and does not have a general installer. You could use other Linux distributions for this as well, but for this I chose [Debian net install](https://www.debian.org/CD/netinst/) because of its conservative updates, it is lightweight, predictable without surprises, and an excellent base for long-running systems. This will turn whatever hardware into a **stable** gaming appliance. There are several commands and some scripts, it may look intimidating for someone who doesn't work with command-line but you do this once and you're set. This system will boot straight to Big Picture mode, and you can use that in place of a desktop.

# Debian CLI Steam console setup

## 1. Install Debian
- Use Debian netinst
- Choose: minimal install (no desktop)
- Partition: entire disk (ext4 or btrfs), recommended for dedicated systems
- Skip adding any desktop environment for now
- Follow the prompts (name, etc.)

A minimal Debian install will boot into a command-line shell.

---

## 2. Update system
Type ```sudo apt update && sudo apt full-upgrade -y && sudo apt autoremove -y && sudo apt clean``` and enter this command in your shell.

This updates package lists, installs updates (including kernels, drivers, Mesa, and firmware), removes unused dependencies, and cleans cached packages.

---

## 3. Install drivers

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

### Audio

For audio drivers:
```bash
sudo apt install pipewire pipewire-audio pipewire-pulse pipewire-jack wireplumber
```

---

## 4. Install Xorg + LightDM
```bash
sudo apt install xorg lightdm
```
Xorg is the display server (renders graphics) and LightDM is the display manager (starts Xorg and logs you in automatically).

Why Xorg instead of Wayland? Wayland requires a compositor or desktop environment, which adds complexity and conflicts with a minimal "console style" setup.

This way is the minimal legacy path if you want just Steam Big Picture. Installing KDE will handle Wayland and all modern components if you want a desktop (later).

---

## 5. Install Steam + Proton
```bash
sudo apt install steam gamemode
```
This adds Steam + Proton as well as GameMode, GameMode automatically adjusts CPU governor, I/O priority, and scheduler settings when games launch. Steam supports it natively; no manual configuration is required. GameMode is optional but recommended.

Proton is enabled in Steam Settings -> Compatibility. Check "Enable Steam Play for supported titles" as well as "Enable Steam Play for all other titles."
Select the default Proton version (latest stable).

---

## 6. Recommended: Desktop
A desktop environment is not technically required for a Steam-only setup. But once you install this, you're done. No more scripts or commands.

If you just want the system to auto-login and launch Steam Big Picture, you can skip this section entirely and use the no-desktop setup below. Installing KDE allows the desktop to handle auto-login, updates, and Steam startup without custom scripts.

A desktop can also be useful if you want mouse interaction, non-Steam games (e.g. GOG via Heroic/Lutris), a web browser and file manager, etc.
This also adds Wayland compositor, PipeWire integration, etc. It will also allow you to skip future command line interactions. 

Simply type and enter:
```bash
sudo apt install kde-standard
```
You can realistically just stop here if you're using the desktop.


KDE installs its own display manager (SDDM), Wayland and X11 sessions are both available at login.

If you encounter issues on Wayland, simply select Plasma (X11) at login. 

From the desktop you can just initiate updates or open Steam on your own. Scripts after this point are for more advanced users.

---

## 7. (No-Desktop) Steam Big Picture Autostart (Optional)

If you installed KDE as recommended, this is optional and can be handled by the KDE desktop instead of custom scripts.

With KDE you can simply go to System Settings -> Apps -> Autostart, Add -> Applications and check Steam.
If you still want Big Picture to autostart, after adding it you can edit the command to be `steam -bigpicture` instead.

If you're relying on a pure-Steam setup with no desktop, create the script:
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

### Create the service
Then add a systemd service to auto-launch this script on startup:

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
Environment=XAUTHORITY=/home/USERNAME/.Xauthority
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

## 8. (No desktop) Daily Automated Update (optional)

Again, if you installed KDE as recommended, this is optional and can be handled by KDE. The KDE desktop will notify you of updates. You can click the notification, and initiate updates. 

Without KDE, this system is now designed to behave like a console. A daily scheduled update can be used on a no-desktop system to keep drivers, firmware, and security fixes current without user interaction. Updates apply quietly; kernel or driver changes take effect on the next reboot. These will not force a restart.


If you don't have the desktop, create the script
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
This signals that the script runs at 7:00 each morning and [can be adjusted](https://crontab.guru/#0_7_*_*_*) accordingly.

(Control+O and Control+X to save and exit again).

---
## 9. Multiplayer titles & kernel anticheat

Some multiplayer games require [kernel-level anticheat software.](https://levvvel.com/games-with-kernel-level-anti-cheat-software/) These programs run with full system privileges, giving them deep access to your system, running with the same privileged access as the kernel itself. This is a security issue, and these anticheat programs are fundamentally incompatible with Linux by design.

This is a **game developer decision**, not a limitation of Steam, Proton, or Linux. Linux supports userspace anticheat. A vast majority of games will work, just as they do on the Steam Deck and will on the Steam Machine. However, popular titles which use kernel-level anticheat simply will not be able to run. Tools like *Easy Anti-Cheat* (EAC) and *BattlEye* do have Linux support and can function with Proton when developers opt in. However, support is optional per-game at the developer's discretion. Other ones like Riot’s *Vanguard*, EA’s *Javelin*, Activision’s *Ricochet*, and others require elevated privileges, disallowing them from working on Linux systems.

I strongly recommend not trying to work around this and not trying to allow kernel-level access to your system. In my opinion, people shouldn't embrace kernel-level anticheat at all. Aside from it basically being a rootkit, it allows the developers to blame the players while not actually solving anything.

## 10. Non-Steam Games

Steam covers most Linux gaming, but some titles are distributed outside of Steam (Epic Games, GOG, Amazon Games, etc.). These work best through dedicated launchers rather than manual Wine or Proton setup.

### Epic Games Store & GOG
```
sudo apt install heroic
```
Heroic provides native Linux support for Epic and GOG libraries, manages Proton or Wine automatically, and works well with a desktop environment like KDE.

### Lutris
If you need broader launcher support (Battle.net, Ubisoft Connect, EA App, legacy installers), Lutris can be used.
```
sudo apt install lutris
```
Lutris supports many launchers through community install scripts and offers more flexibility, but also requires more user involvement. It is best suited for advanced users or mixed game libraries.

The Amazon Games Launcher does not have a native Linux client. Amazon-distributed games can typically be installed through Lutris using Wine-based launchers. Compatibility varies by title. Installed games can optionally be added to Steam afterward for Big Picture and controller support.

These can coexist with Steam games without issue. Adding non-Steam games to Steam is optional but recommended if using Big Picture mode.

---

## 10. Notes
- This seems like work but most of it is just typing and entering or saving info, if you install KDE it's a bit less work
- This is **only done once** and this will result in a stable Steam gaming system that uses nearly the same stack as Valve and behaves like SteamOS under the hood
- If Steam Big Picture doesn’t start automatically with your systemd service -> check ```systemctl status launch.service```
- If updates didn’t run -> check /var/log/update.log
- Steam Input handles native controller support (Xbox, Playstation, Nintendo Switch), games launched through Steam need no extra support
- If your controller isn’t recognized -> verify Steam Input is enabled
- There are gaming distros but those are desktop PC systems, this allows you to set up your own gaming-specific system
- I also have a headless CLI Plex media server that is exactly this but functions as a media server- this setup barely uses resources and allows maximum uptime
- If you do need to shut down entering ```sudo shutdown -r now``` will restart and ```sudo shutdown -h now``` will shut it off
- If you're worried about using a new system with command line, they are well documented and can always be looked up


