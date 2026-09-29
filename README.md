# Super Shift G - Mint Deck Mode

Install video

<p align="center">
  <a href="https://youtu.be/lwleCvIMuCw">
    <img src="https://img.youtube.com/vi/lwleCvIMuCw/hqdefault.jpg" width="700">
  </a>
</p>

**Version 13.00-mint**

Turn a Linux Mint Cinnamon desktop into a Steam Deck-like gaming console with a single keybind. Press `Super+Shift+G` to enter Gaming Mode (Steam Big Picture in Gamescope), and use Steam's "Exit to Desktop" to return to Cinnamon.

Built for [Linux Mint](https://linuxmint.com/) running Cinnamon on LightDM.

## What It Does

This installer transforms your desktop into a dual-mode system:

- **Desktop Mode** - Your normal Cinnamon (X11) session
- **Gaming Mode** - Full-screen Steam Big Picture running inside Gamescope (the same compositor used by the Steam Deck), with automatic performance tuning, controller support, and external drive mounting

Switching between modes is seamless — LightDM handles session transitions, and all your network, audio, and peripherals carry over automatically.

## Requirements

- **OS**: [Linux Mint](https://linuxmint.com/) (Ubuntu-based, tested on Mint 22)
- **Desktop**: Cinnamon (X11)
- **Display Manager**: LightDM
- **GPU**: AMD (discrete or APU) or NVIDIA (discrete)
  - Intel-only systems are **not supported**
  - Intel iGPU + AMD/NVIDIA dGPU configurations work fine

> **Note**: This script is designed specifically for Linux Mint and its stack (Cinnamon, LightDM, NetworkManager, PipeWire). It uses `apt` for package management and builds Gamescope from source as a recent build is not available in Mint repos. It is not intended for Arch-based or Fedora-based distributions — see [Super-Shift-S-Omarchy-Deck-Mode](https://github.com/28allday/Super-Shift-S-Omarchy-Deck-Mode) for Omarchy/Arch, or [Super-Shift-G-Nobara-Deck-Mode](https://github.com/28allday/Super-Shift-G-Nobara-Deck-Mode) for Nobara/Fedora.

## Quick Start

```bash
git clone https://github.com/28allday/Super-Shift-G-Mint-Deck-Mode.git
cd Super-Shift-G-Mint-Deck-Mode
chmod +x super_shift_g_mint.sh
./super_shift_g_mint.sh
```

The installer is fully interactive and will walk you through each step.

## Usage

| Action | Keybind |
|--------|---------|
| Enter Gaming Mode | `Super + Shift + G` |
| Return to Desktop | Steam > Power > Exit to Desktop |

### Command-Line Options

```
./super_shift_g_mint.sh              # Full installation
./super_shift_g_mint.sh --verify     # Verify installation only
./super_shift_g_mint.sh --version    # Show version
./super_shift_g_mint.sh --help       # Show help
```

## What Gets Installed

### Packages (via apt)

The installer checks for and offers to install:

**Core Steam Dependencies**
- `steam-installer`, `gamescope` (built from source), `mangohud` (built from source), `gamemode`
- Vulkan loaders and Mesa libraries (32-bit and 64-bit)
- Audio libraries (PipeWire / pulseaudio compatibility)
- Fonts (`fonts-liberation`)

**GPU-Specific Drivers**
- **NVIDIA**: proprietary driver packages, `libnvidia-egl-wayland1`, VA-API driver
- **AMD**: `libvulkan1`, `mesa-vulkan-drivers`, `libva-mesa-driver`

**Build Dependencies** (for Gamescope from source)
- Meson, ninja, CMake, `libwayland-dev`, `libdrm-dev`, and others

**ChimeraOS Session Scripts**
- Cloned from GitHub — provides the gamescope-session framework

**Apt Resilience**
- If a package install hits a broken dpkg state or mirror failure, the installer automatically cleans the cache, retries, and falls back through multiple Ubuntu mirrors before failing.

### Files Created

#### Session Scripts
| Path | Purpose |
|------|---------|
| `/usr/local/bin/switch-to-gaming` | Switches from Cinnamon to Gaming Mode |
| `/usr/local/bin/switch-to-desktop` | Switches from Gaming Mode back to Cinnamon |
| `/usr/local/bin/gamescope-session-nm-wrapper` | Main session wrapper (performance mode, NM, drive mounting) |
| `/usr/local/bin/gaming-session-switch` | Helper to toggle LightDM session config between modes |
| `/usr/local/bin/gaming-keybind-monitor` | Python daemon monitoring keyboard in Gaming Mode |
| `/usr/lib/os-session-select` | Handler for Steam's "Exit to Desktop" button |
| `/usr/local/lib/gamescope-nvidia/gamescope` | NVIDIA wrapper adding `--force-composition` flag |

#### NetworkManager Integration
| Path | Purpose |
|------|---------|
| `/usr/local/bin/gamescope-nm-start` | Ensures NetworkManager is active on gaming session entry |
| `/usr/local/bin/gamescope-nm-stop` | No-op on Mint (NetworkManager always runs) — kept for compatibility |

#### External Drive Support
| Path | Purpose |
|------|---------|
| `/usr/local/bin/steam-library-mount` | Auto-detects and mounts drives with Steam libraries |

#### Session & Display Manager
| Path | Purpose |
|------|---------|
| `/usr/share/xsessions/gamescope-session-steam-nm.desktop` | LightDM session entry for Gaming Mode |

#### Permissions & Security
| Path | Purpose |
|------|---------|
| `/etc/sudoers.d/gaming-session-switch` | Passwordless sudo for session switching, NM, bluetooth |
| `/etc/sudoers.d/gaming-mode-sysctl` | Passwordless sudo for performance sysctl tuning |
| `/etc/polkit-1/rules.d/50-gamescope-networkmanager.rules` | Polkit rules for NM D-Bus access |
| `/etc/polkit-1/rules.d/50-udisks-gaming.rules` | Polkit rules for external drive mounting |
| `/etc/udev/rules.d/99-gaming-performance.rules` | Udev rules for CPU/GPU performance control |
| `/etc/security/limits.d/99-gaming-memlock.conf` | Memory lock limits (2GB) for gaming |

#### Performance & Environment
| Path | Purpose |
|------|---------|
| `/etc/environment.d/99-shader-cache.conf` | Shader cache optimisation (12GB Mesa/DXVK cache) |
| `/etc/environment.d/90-nvidia-gamescope.conf` | NVIDIA Gamescope environment variables |
| `/etc/pipewire/pipewire.conf.d/10-gaming-latency.conf` | PipeWire low-latency audio config |

#### User Config
| Path | Purpose |
|------|---------|
| `~/.config/environment.d/gamescope-session-plus.conf` | Gamescope session config (resolution, refresh rate, GPU) |

## How It Works

### Session Switching Flow

```
Desktop Mode (Cinnamon)
    |
    +- Super+Shift+G pressed
    |   +- switch-to-gaming runs:
    |       +- Masks suspend targets (prevents sleep during switch)
    |       +- Updates LightDM config to gaming session
    |       +- Restarts LightDM -> boots into Gaming Mode
    |
Gaming Mode (Gamescope + Steam Big Picture)
    |
    +- On session start (gamescope-session-nm-wrapper):
    |   +- Enables performance mode (CPU governor, GPU tuning)
    |   +- Ensures NetworkManager is active (Mint never stops NM)
    |   +- Launches steam-library-mount (external drive detection)
    |   +- Starts gaming-keybind-monitor
    |   +- Launches gamescope-session-plus with Steam
    |
    +- Steam > Power > Exit to Desktop
    |   +- switch-to-desktop runs:
    |       +- Unmasks suspend targets
    |       +- Restores Bluetooth
    |       +- Shuts down Steam gracefully
    |       +- Kills gamescope (releases DRM master — critical for AMD)
    |       +- Updates LightDM config to Cinnamon session
    |       +- Restarts LightDM -> boots into Desktop Mode
    |
    +- On session cleanup (trap handler):
        +- Kills steam-library-mount and keybind-monitor
        +- Restores balanced power mode
```

### Performance Mode

When Gaming Mode starts, the session wrapper automatically:

- Sets CPU governor to `performance` on all cores
- **NVIDIA**: Enables persistence mode, sets power limit to maximum, disables runtime suspend
- **AMD**: Sets GPU to high performance via `power_dpm_force_performance_level`
- Applies kernel sysctl tuning (scheduler, VM, inotify, network buffers)
- Sets power profile to `performance` (if power-profiles-daemon is available)

On exit, everything is restored to balanced/powersave defaults.

### GPU Detection

The installer automatically detects your GPU configuration:

- **AMD dGPU**: Detected via PCI device names (Navi, RDNA, Vega discrete cards)
- **AMD APU**: Detected via integrated GPU codenames (Phoenix, Rembrandt, Van Gogh, etc.)
- **NVIDIA**: Detected via lspci, configures `nvidia-drm.modeset=1` if missing
- **Multi-GPU**: Correctly identifies discrete vs integrated, selects dGPU for gaming

### Monitor Detection

The installer scans DRM connectors on your gaming GPU to find connected displays. If multiple monitors are connected to the dGPU, you can choose which one to use for Gaming Mode. Resolution and refresh rate are auto-detected from EDID data.

### External Drive Auto-Mount

The `steam-library-mount` daemon runs during Gaming Mode and:

1. Scans all connected drives for Steam library folders
2. Mounts drives containing `steamapps/` directories via udisks2
3. Monitors udev for hot-plugged drives
4. Unmounts non-Steam drives to avoid clutter

Supports ext4, NTFS, btrfs, xfs, exfat, f2fs, and vfat filesystems.

## Configuration

### Config File

The installer reads from `/etc/gaming-mode.conf` (or `~/.gaming-mode.conf` if it exists):

```bash
PERFORMANCE_MODE=enabled   # Set to "disabled" to skip performance tuning
```

### Gamescope Session Config

After installation, you can edit `~/.config/environment.d/gamescope-session-plus.conf`:

```bash
SCREEN_WIDTH=2560
SCREEN_HEIGHT=1440
CUSTOM_REFRESH_RATES=165
OUTPUT_CONNECTOR=DP-1
ADAPTIVE_SYNC=1            # AMD only
ENABLE_GAMESCOPE_HDR=1     # AMD only
```

**NVIDIA note**: Resolution is capped at 2560x1440 due to Gamescope limitations with NVIDIA GPUs.

### Shader Cache

The installer configures a 12GB shader cache by default in `/etc/environment.d/99-shader-cache.conf`. This reduces stutter in games by caching compiled shaders. Values can be adjusted:

```bash
MESA_SHADER_CACHE_MAX_SIZE=12G
__GL_SHADER_DISK_CACHE_SIZE=12884901888
DXVK_STATE_CACHE=1
```

## Key Differences from Other Versions

| | Omarchy (Arch) | Nobara (Fedora) | **Mint (Ubuntu)** |
|---|---|---|---|
| **Package manager** | pacman / yay | dnf | **apt** |
| **Desktop** | Hyprland | KDE Plasma | **Cinnamon (X11)** |
| **Display manager** | SDDM | plasmalogin | **LightDM** |
| **Networking** | iwd | NetworkManager | **NetworkManager (never stopped)** |
| **Gamescope** | pacman package | Built from source | **Built from source** |
| **MangoHud** | pacman package | dnf package | **Built from source** |
| **ChimeraOS session** | AUR packages | Cloned from GitHub | **Cloned from GitHub** |
| **Gaming keybind** | `Super+Shift+S` | `Super+Alt+G` | **`Super+Shift+G`** |
| **Return keybind** | `Super+Shift+R` | Steam > Exit to Desktop | **Steam > Exit to Desktop** |

## NVIDIA-Specific Notes

- **Kernel parameter**: `nvidia-drm.modeset=1` is required. The installer can configure this for GRUB.
- **Resolution cap**: Gamescope on NVIDIA is limited to 2560x1440 maximum.
- **Force composition**: The NVIDIA wrapper automatically adds `--force-composition` if supported by your Gamescope version.
- **Environment**: `GBM_BACKEND=nvidia-drm` and related vars are set automatically.
- **Persistence mode**: Enabled during gaming to keep the GPU initialised, disabled on exit.

## Troubleshooting

### Verify Installation

Run the built-in verification to check all files, permissions, packages, and services:

```bash
./super_shift_g_mint.sh --verify
```

### Common Issues

**Gaming Mode doesn't start / black screen**
- Check NVIDIA kernel params: `cat /proc/cmdline | grep nvidia`
- Verify gamescope works: `gamescope -- steam`
- Check session logs: `journalctl --user -u gamescope-session -n 50`

**Apt install failures**
- The installer retries with fallback mirrors automatically
- Manual recovery: `sudo dpkg --configure -a && sudo apt clean && sudo apt install -y --fix-broken`

**Super+Shift+G doesn't work**
- Log out and back in after installation
- Check Cinnamon keyboard shortcuts for "Switch to Gaming Mode"
- Verify the desktop file exists: `ls /usr/share/applications/switch-to-gaming.desktop`

**External drives not mounting**
- Ensure `udisks2` is installed: `dpkg -l udisks2`
- Check polkit rules exist: `ls /etc/polkit-1/rules.d/50-udisks-gaming.rules`
- Check mount logs: `journalctl -t steam-library-mount -n 20`

**Audio stuttering in Gaming Mode**
- Check PipeWire config exists: `cat /etc/pipewire/pipewire.conf.d/10-gaming-latency.conf`
- Try lower quantum: edit the config and set `default.clock.min-quantum = 128`

**"NO DICE - INTEL ONLY DETECTED"**
- This system has only Intel graphics. Gaming Mode requires AMD or NVIDIA.
- If you have a discrete GPU, check that its driver is loaded: `lspci -k | grep -A2 VGA`

### Log Locations

| Component | Command |
|-----------|---------|
| Gaming session | `journalctl --user -u gamescope-session` |
| NetworkManager | `journalctl -t gamescope-nm` |
| Drive mounting | `journalctl -t steam-library-mount` |
| Keybind monitor | `journalctl -t gaming-keybind-monitor` |
| Session wrapper | `journalctl -t gamescope-wrapper` |
| Installation | `journalctl -t gaming-mode` |

## Credits

- [Linux Mint](https://linuxmint.com/) - The Ubuntu-based distribution this was adapted for
- [Omarchy](https://omarchy.com) - Original script built for Omarchy (Arch Linux)
- [ChimeraOS](https://chimeraos.org/) - gamescope-session packages
- [Valve](https://store.steampowered.com/) - Steam, Gamescope, and the Steam Deck inspiration

## License

This project is provided as-is for the Linux Mint gaming community.
