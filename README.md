# Arch Linux: Post-Install Application Setup Guide

This guide covers application installation and peripheral setup after a base Arch Linux system has been configured (system updates, mirrors, AUR helper, firewall, etc. are assumed complete).

---

## Prerequisites

- A configured Arch Linux system with `yay` installed
- Internet access
- A non-root user with `sudo` privileges

---

## Install Common Applications

Install packages from the official repositories:

```bash
sudo pacman -Syu --needed steam libreoffice-fresh vlc nextcloud-client tailscale ckb-next
```

Install AUR packages via Yay:

```bash
yay -S --needed discord betterbird-bin minecraft-launcher zoom protonup-qt outlook-for-linux ttf-ms-fonts
```

---

## Enable and Start Services

Enable and start the Tailscale daemon:

```bash
sudo systemctl enable --now tailscaled
```

Enable and start the CKB-Next daemon (Corsair peripheral support):

```bash
sudo systemctl enable --now ckb-next-daemon
```

Connect to your Tailnet:

```bash
sudo tailscale up
```

---

## Razer Nari Wireless Pairing

> **Source:** [juanjodarko/razer-nari-pairing](https://github.com/juanjodarko/razer-nari-pairing)
>
> An open-source tool to pair Razer Nari headsets (Ultimate, Regular, Essential) with their USB dongles on Linux — no Razer software required. Supports cross-model pairing.

### Supported Hardware

- Razer Nari Ultimate ✅ (confirmed working)
- Razer Nari Regular ✅ (confirmed working)
- Razer Nari Essential ⚠️ (likely works, needs community verification)

### Installation

Install Python dependencies:

```bash
pip install pyusb
```

> **Note:** On Arch Linux, if pip blocks the global install, use a virtual environment:
>
> ```bash
> python3 -m venv venv
> source venv/bin/activate
> pip install -r requirements.txt
> ```

Clone the repository:

```bash
git clone https://github.com/juanjodarko/razer-nari-pairing.git
cd razer-nari-pairing
pip install -r requirements.txt
```

### Pairing Process

1. Plug the USB dongle into a USB port.
2. Connect the headset via its USB charging cable.
3. Turn the headset **ON** (press the power button).
4. Run the pairing tool:

```bash
./pair.sh
```

Or directly:

```bash
sudo python3 razer_nari_pair.py
```

5. Wait ~5 seconds for the pairing commands to complete.
6. Disconnect the USB charging cable, then turn the headset **OFF** and back **ON**.
7. The headset should now connect wirelessly. A solid blue LED followed by a connection tone confirms success.

> Pairing is permanent — the headset and dongle will remember each other across reboots.

### Audio Profile Setup (Post-Pairing)

The Razer Nari has dual audio outputs (Game stereo + Chat mono). Install the appropriate audio profile for your setup:

**PipeWire:**

```bash
yay -S razer-nari-pipewire-profile
```

Set the Game output as default:

```bash
pactl set-default-sink alsa_output.usb-Razer_Razer_Nari_Ultimate-00.analog-game
```

### Troubleshooting

| Issue | Fix |
|---|---|
| "No dongle found" | Check `lsusb` for device `1532:051a` (or `051c`/`051e`) |
| "No headset found" | Ensure headset is connected via USB cable and powered ON |
| "Permission denied" | Run with `sudo python3 razer_nari_pair.py` |
| Headset keeps blinking blue | Re-run the pairing tool; ensure headset was ON during the process |
| No audio after pairing | Install audio profiles above; verify with `pactl list sinks \| grep Nari` |

---

## AeroThemePlasma (Windows Aero Theme for KDE Plasma)

> **Source:** [AeroShell/AeroThemePlasma](https://gitgud.io/aeroshell/atp/aerothemeplasma)
>
> A full Windows 7 Aero visual style for KDE Plasma 6, including window decorations, KWin effects, plasmoids, SDDM theme, icons, and sounds. X11 is recommended — some Wayland restrictions make certain effects impossible to achieve.

### ⚠️ Important Notes Before Installing

- **Replaces `libplasma`:** The `aeroshell-libplasma` AUR package is a patched fork of `libplasma` required for Aero glass effects. Yay will prompt to remove the stock `libplasma` — answer **yes**, this is intentional and expected.
- **After any future `pacman -Syu`** that upgrades `libplasma`: the stock version will likely be restored, breaking ATP's effects. When this happens, rebuild `aeroshell-libplasma-git` from the AUR to restore them.
- **Upgrading from a source install:** Run `bash uninstall.sh` in your old ATP source directory before proceeding to avoid file conflicts.

### Install Dependencies

```bash
sudo pacman -S git cmake extra-cmake-modules ninja curl unzip \
  qt6-virtualkeyboard qt6-multimedia qt6-5compat qt6-wayland \
  plasma-wayland-protocols plasma5support kvantum \
  sddm sddm-kcm base-devel \
  plasma-nm plasma-pa plasma-workspace plasma-desktop \
  kwin-x11 plasma-x11-session
```

### Install via AUR

This pulls and builds all ATP components — decoration, effects, plasmoids, SDDM theme, icons, and sounds. It will take a while.

```bash
yay -S aerothemeplasma-desktop-x11 uac-polkit-agent aerothemeplasma-desktop \
  aeroshell-libplasma aeroshell-workspace aeroshell-kwin-components \
  aeroshell-smod aeroshell-smodglow-x11 \
  aerothemeplasma-sounds aerothemeplasma-icons
```

When prompted about the `libplasma` conflict, enter **y** to remove it and allow `aeroshell-libplasma` to take its place.

### Switch to the AeroThemePlasma Session

Log out of your current session. At the SDDM login screen, select the **AeroThemePlasma** session from the session menu before logging in. A setup wizard will launch on first login to finish applying the theme.

### Troubleshooting

| Issue | Fix |
|---|---|
| KWin effects stop working after upgrade | Rebuild `aeroshell-libplasma-git` and `aeroshell-kwin-components` from AUR |
| `libplasma` conflict during install | Answer `y` at the conflict prompt — this is expected |
| Aero glass not appearing | Ensure you're on X11, not Wayland |
| File conflicts from old source install | Run `bash uninstall.sh` in old ATP source dir first, then re-run yay |

---
