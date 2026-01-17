# Steam Machine

Create your own Steam machine: a lightweight setup that boots Steam Big Picture. No bloat, just a non-Steam-branded machine with 
your hardware of choice. With Valve reviving the Steam Machine concept, this guide documents a clean, minimal way to build your own today.

## Why?

About two months ago Valve announced their Steam Machine, essentially a mid-sized PC with moderate hardware. Many people wonder about
future capabilities and better hardware, costs, etc. However, it is more or less a software configuration which anyone can set up themselves,
and I had already been planning to do this even before the announcement. This is nothing against Valve; it's for those who want to use what 
is basically the same software stack that Steam uses on a different hardware configuration. The costs for an actual Steam Machine haven't been 
confirmed yet, and anyone who already has the hardware can set it up without buying another system. Note that operating systems like Bazzite 
exist, which basically has the same software stack already installed on a Fedora Linux distro. This is for those who want to set their distro 
up for gaming. If your goal is just to set it up and play games with a console-first mindset, just install [Bazzite](https://bazzite.gg/).

SteamOS itself is restricted, now based on Arch Linux, locked down and does not have a general installer. You could use other Linux 
distributions for this as well, but for this setup I chose [Fedora KDE](https://fedoraproject.org/kde/download) because it already 
assumes modern Linux components like PipeWire and Wayland, tracks newer kernels and graphics stacks, and requires very little manual 
configuration for gaming. It provides a clean, general-purpose desktop that can still behave like a console when configured to launch 
Steam automatically. Once set up, it turns standard PC hardware into a reliable gaming system without locking it down or hiding how 
the system works. For Debian, go [here](debian.md).

## 1. Install Fedora KDE

- Download **Fedora KDE Spin**
- Install normally using the graphical installer
- Default partitioning (btrfs) is fine
- Create a normal user account
- Reboot into KDE Plasma

You now have a modern Linux desktop with sane defaults.

---

## 2. Update the System

Open **Konsole** and run:

```bash
sudo dnf upgrade --refresh -y
```

This updates the kernel, Mesa, firmware and KDE components.

Reboot once after major updates:

```bash
sudo reboot
```

---

## 3. Enable RPM Fusion (Required)

Steam and many multimedia components are provided via RPM Fusion.

```bash
sudo dnf install -y \
  https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

Refresh metadata:

```bash
sudo dnf upgrade --refresh -y
```

---

## 4. GPU Drivers

### Intel/AMD (Mesa)

Nothing special is required. Fedora ships excellent Mesa drivers by default.

Optional Vulkan utilities:

```bash
sudo dnf install -y vulkan-tools
```

### NVIDIA

Install the proprietary driver from RPM Fusion:

```bash
sudo dnf install -y akmod-nvidia xorg-x11-drv-nvidia-cuda
```

Reboot after installation:

```bash
sudo reboot
```

Verify driver status:

```bash
nvidia-smi
```

---

## 5. Install Steam + GameMode

```bash
sudo dnf install -y steam gamemode
```

- Steam includes Proton
- GameMode is automatically used by Steam
- 32-bit libraries are handled automatically by Fedora

Launch Steam once to allow it to self-update.

---

## 6. Enable Proton

In **Steam -> Settings -> Compatibility**:

- Enable Steam Play for supported titles
- Enable Steam Play for all other titles
- Select the latest **Proton (stable)** as default

Your system is now ready for gaming.

---

## 7. Auto-start Steam (Optional)

1. Open **System Settings**
2. Go to **Apps -> Autostart**
3. Add -> Application -> **Steam**

### For Big Picture
Edit the entry and change the command to:

```
steam -bigpicture
```

### For Steam Deck UI
Edit the entry and change the command to:

```
steam -gamepadui
```

Either option provides a controller-friendly boot while keeping the desktop as a fallback if needed.
Only one should be used for autostart at a time; change as desired.

---

## 8. Non-Steam Games

### Heroic (Epic Games Store & GOG)

```bash
sudo dnf install -y heroic-games-launcher
```

Heroic provides Epic Games Store, GOG and Proton/Wine management.

### Lutris (Advanced / Mixed Libraries)

```bash
sudo dnf install -y lutris
```

Lutris supports Battle.net, Ubisoft Connect, EA App, legacy installers and more, but requires more manual configuration.

Games from Heroic or Lutris can be added to Steam for Big Picture and controller support.

---

## 9. Audio, Controllers, Display

Fedora KDE already includes PipeWire/WirePlumber, Steam Input for controllers and a Wayland session by default.

No manual audio or controller configuration is required.

If something doesn’t work:
- Verify Steam Input is enabled
- Check controller settings in Steam
- As a fallback, try launching Steam under an X11 session from login (rarely needed)

---

## 10. Multiplayer titles & kernel anticheat

Some multiplayer games require [kernel-level anticheat software.](https://levvvel.com/games-with-kernel-level-anti-cheat-software/) 
These programs run with full system privileges, giving them deep access to your system, running with the same privileged access as 
the kernel itself. This is a security issue, and these anticheat programs are fundamentally incompatible with Linux by design.

This is a **game developer decision**, not a limitation of Steam, Proton, or Linux. Linux supports userspace anticheat. A vast 
majority of games will work, just as they do on the Steam Deck and will on the Steam Machine. However, popular titles which use 
kernel-level anticheat simply will not be able to run. Tools like *Easy Anti-Cheat* (EAC) and *BattlEye* do have Linux support 
and can function with Proton when developers opt in. However, support is optional per-game at the developer's discretion. Other 
ones like Riot’s *Vanguard*, EA’s *Javelin*, Activision’s *Ricochet* and others require elevated privileges, disallowing them 
from working on Linux systems.

I strongly recommend not trying to work around this and not trying to allow kernel-level access to your system. In my opinion, 
people shouldn't embrace kernel-level anticheat at all. Aside from it basically being a rootkit, it allows the developers to 
blame the players while not actually solving anything.

---

## 11. Notes

You can use Bazzite if you want NO setup beyond install and boot. With Bazzite you get good defaults but an 
immutable OS, system changes are layered, traditional package installs are discouraged, debugging is different, 
fixes sometimes mean reboot, rollback, waiting for upstream. Bazzite makes choices for you: Steam-first behavior, 
GameMode assumptions, Deck-style UI expectations and certain kernel or module configurations. These are good 
defaults, but they are still someone else’s decisions. For some, this is fine. For others, it is not. This guide 
is for the latter.

- Fedora KDE is the default, low-friction path
- No scripts required unless intentionally removing the desktop
- Steam updates itself; system updates come via KDE notifications
- This behaves very similarly to SteamOS while remaining fully open
- Debian is still an option but requires more command line use and more steps

---

### TL;DR

If you want:
- Modern Linux gaming
- Minimal setup
- No hacky scripts
- Easy maintenance

Fedora + Steam Autostart is the correct default.
