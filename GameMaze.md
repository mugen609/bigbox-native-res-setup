# GameMaze  
Native Resolutions • Seamless Gamepad Control • Console-Like Experience for scalers on modern displays

Imagine launching Pac-Man in its original 224p arcade resolution, then switching to The Witcher 3 at 720p—all from your couch, using only a gamepad. No keyboard, no fuss. This project is a passion-driven emulation hub that unifies 40+ years of gaming history, from 1980s arcade classics to modern Switch and Steam titles, in their original resolutions. It delivers a console-like experience that prioritizes visual fidelity, ease of use, and minimal compromises.

# Table of Contents
- [Overview](#overview)  
  - [Key Features](#key-features)
- [Supported Systems and Resolutions](#supported-systems-and-resolutions)
- [Why Build It?](#why-build-it)
- [Requirements and Software](#requirements-and-software)  
  - [Hardware Requirements](#hardware-requirements)  
  - [Software Requirements](#software-requirements)   
- [Video Signal Chain](#video-signal-chain)
- [Setup Process](#setup-process)  
  - [Preparing the PC](#preparing-the-pc)  
  - [GPU and Driver Setup](#gpu-and-driver-setup)  
    - [AMD GPU + HDMI Scaler: HDCP Fix (If Image Looks Wrong)](#amd-gpu--hdmi-scaler-hdcp-fix-if-image-looks-wrong)  
  - [Windows Configuration](#windows-configuration)
- [SSH for File Transfers](#ssh-for-file-transfers)    
- [Resolution and Emulator Configuration](#resolution-and-emulator-configuration)  
  - [Resolution Setup with CRU](#resolution-setup-with-cru)  
  - [Emulator Configuration](#emulator-configuration)  
    - [Resolution Handling](#resolution-handling)  
    - [RetroArch Configuration](#retroarch-configuration)  
    - [Standalone Emulator Configuration](#standalone-emulator-configuration)  
  - [Steam](#steam)
- [reWASD and LaunchBox Setup](#rewasd-and-launchbox-setup)  
  - [reWASD Setup](#rewasd-setup)  
  - [LaunchBox Configuration](#launchbox-configuration)
- [Using Res-O-Matic for Custom Resolutions](#using-res-o-matic-for-custom-resolutions)
- [Dismissing Popups (Optional)](#dismissing-popups-with-windowspy-fix)  
- [Controller Reordering Workaround](#controller-reordering-workaround)
- [Final Touches](#final-touches)  
- [Conclusion](#conclusion)

---
![DQ6](/images/DQ6.jpg)  
*Dragon Quest 6 on SNES in native resolution*

## Overview

GameMaze is a unified emulation platform for Windows that outputs native resolutions (160p–1080p) over HDMI to a modern scaler, preserving the original look of each system. It focuses on an all-digital signal chain, custom timings via CRU, and a low-latency scaler (e.g., RetroTink 4K) rather than CRT EmuDriver and analog outputs.

The key idea is to keep the console-accurate signal (resolution, aspect ratio, no smoothing) while using modern emulators and hardware with a gamepad-only frontend. True low-resolution output over HDMI, combined with standardized controls via reWASD and seamless LaunchBox/BigBox integration, is what makes the setup different from typical 480p+ emulation builds.

 - *Note: This setup assumes you are willing to define custom timings with CRU, rely on an HDMI scaler that accepts low resolutions, and use workarounds for quirks like Windows’ controller ordering. The rest of this guide focuses on providing concrete solutions to those constraints.*


### Key Features

- **Pixel-Perfect Visuals**: Every game runs at its native resolution, upscaled to a modern display with CRT shaders.  
- **Seamless Gamepad Control**: Navigate menus and play games (tested with four controllers), standardized across all systems with consistent hotkeys (e.g., Home + Start to quit).  
- **Unified Frontend**: LaunchBox/BigBox provides a great, gamepad-only interface, integrating standalone emulators and Steam into a visually nice cohesive experience.  
- **Versatile Output**: Supports both horizontal and vertical (TATE) arcade games, with flexible output to modern displays via HDMI.

The system is designed to output to any HDMI-compatible scaler, supporting up to 4K with a suitable monitor.

In my setup, a RetroTINK 4K scaler upscales the PC’s native inputs to 1440p (limited by my LG 27GS95QE-B OLED monitor), applying CRT shaders for a sharp, scanline-rich image. Vertical arcade games are displayed in TATE mode by physically rotating the monitor on a VESA stand.

---

## For Digital Output (Not CRT)

This guide focuses on getting authentic retro resolutions from a Windows emulation PC over digital video (HDMI/DisplayPort) to a modern flat panel. A scaler is required in the chain (e.g., a RetroTINK) before the display.

This is **not** a guide for analog output to CRT TVs/monitors (VGA/SCART/component) and does not cover CRT-specific constraints or setup paths.

### Why That Distinction Matters

Most “240p on PC” legacy guides focus on CRT EmuDriver and specific GPU compatibility, because analog output depends heavily on driver + DAC/output constraints. With an all-digital chain, the system instead relies on EDID overrides, custom timings, and “super resolutions” (e.g., 2560 × original height), which are configured at the display-timing level and work across a wider range of modern GPUs.

This setup has been tested on an AMD RX 6700 XT and should work with many newer cards beyond the usual CRT EmuDriver lists.

---

## Supported Systems and Resolutions

Native resolutions for each platform.

| System                 | Native Resolution        |
|------------------------|--------------------------|
| Game Boy Advance       | 160p                     |
| NES, SNES, Megadrive   | 224p–240p                |
| Arcade (MAME, FBNeo)   | 224p–480p                |
| Nintendo DS            | 256p (dual screen stack) |
| PSP                    | 272p                     |
| Sega Model 3           | 384p                     |
| Dreamcast, PS2, Wii    | 480p                     |
| PS3, Switch, Steam     | 720p                     |

---

## Why Build It?

### The Discovery

This project started with a simple observation: games running on my physical SNES or PS1 hardware through my RetroTINK 4K looked noticeably better than the same games emulated via Batocera. At first, it was tempting to think original hardware had some inherent visual magic that emulation could not replicate.

But the answer was simpler: **resolution**.

Physical consoles output true 240p signals. Batocera and most emulation setups upscale before sending to the scaler—forcing the RetroTINK to process an already-compromised signal. Once emulators were configured to output native resolutions, visual quality matched (and in some cases exceeded) original hardware.

### Beyond Hardware Parity

With proper configuration, emulation can actually improve on original hardware. To keep a clean, authentic signal, the key principles are:

- Preserve original resolution.  
- Preserve original aspect ratio.  
- Avoid smoothing filters (prefer native or nearest-neighbor over xBR, etc.).

Examples:

- **2D/Pixel Art Systems**:  
  Native output resolution with nearest-neighbor scaling produces image quality identical to original hardware (no SuperEagle, 2xBRZ, or smoothing filters). GBA games especially benefit from OLED displays, which are dramatically sharper and more color-accurate than the original dim, washed-out screens.

- **3D Games**:  
  Modern emulators can separate *internal rendering resolution* from *output resolution*. You can render at 2×–3× internally (cleaner textures, reduced polygon shimmer) while outputting native 240p/480p to the scaler. Anti-aliasing is also a solid option. This removes many hardware limitations while preserving authentic signal processing.

### Why Existing Solutions Fall Short

- **MiSTer/FPGA**:  
  Excellent cycle-accurate 240p for retro systems (up to PS1) but no support for modern platforms.

- **Batocera/RetroPie/Lakka**:  
  Very user-friendly but typically upscale retro games to at least 480p, losing native resolution fidelity and struggling to handle a wide range of resolutions (Legacy + modern at the same time).

- **CRT EmuDriver**:  
  Powerful for 240p analog output but limited to older GPUs, outdated drivers, and incompatible with modern APIs (Vulkan, Steam, etc.).

GameMaze aims to unify the authentic visual experience of original hardware with the convenience of modern emulation save states, unified library, TATE support, Steam integration... All from a single, gamepad-controlled interface.

---

## Requirements and Software

### Before You Start

- **Clean Windows install (recommended)**: This guide assumes a clean Windows 11 install (fresh OS), followed by your preferred GPU driver and core tools before emulator configuration. But feel free to adapt it to an existing install, if you prefer.
- **Network access & SSH**: A basic LAN connection is strongly recommended. Enabling SSH on the emulation PC (e.g., via [OpenSSH](#ssh-for-file-transfers) Server) allows file transfers without USB drives. USB or NAS also work, but SSH is most convenient on a gamepad-only machine.
- **Secondary PC (optional)**: Helpful for setup tasks like ROM transfers or scripting, but not absolutely required.


### Hardware Requirements

The system requires:

- A PC with sufficient power for emulation.  
- A compatible GPU/driver combination.  
- An HDMI scaler.  
- A modern display.

The tested configuration can be used as a reference.

- **GPU**: Must support EDID overrides/custom resolutions (CRU) and the required timings/pixel clocks.  
  - Compatibility can vary by driver features (e.g., DSC on NVIDIA) and multi-display setups.  
  - **Tested**: Radeon HD 7750, RX 580 Nitro+, RX 6700 XT (all as eGPU via OCuLink).

- **Scaler/Display**:  
  An HDMI scaler accepting legacy resolution inputs and upscaling to your monitor’s native resolution (1080p or higher), ideally with CRT shader support.  
  - **Tested**: RetroTINK 4K scaler outputting 1440p to a 27" LG ULTRAGEAR OLED monitor on a VESA stand for TATE mode.

- **Controllers**: Gamepads (wired or wireless).  
  - **Tested**: Two Sony DualSense, Hori Octa Pro, Xbox Series X controller (four players).

![OCulink RX580](/images/OCulink_RX580.jpeg)

### Software Requirements

Some paid and some free components are used. Essential software is required for core functionality; optional tools simplify setup or enhance performance.

#### Essential Software

| Software | Type | Purpose |
| --- | --- | --- |
| Windows 11 Pro | Paid | Operating system. |
| [LaunchBox/BigBox](https://www.launchbox-app.com/) | Paid | Gamepad-driven frontend for unified emulator and Steam integration. |
| [reWASD](https://www.rewasd.com/) | Paid | Standardizes controller inputs across all systems and emulators. |
| [Custom Resolution Utility (CRU)](https://customresolutionutility.net/) | Free | Adds custom HDMI resolutions/timings (EDID override) for low/legacy modes. |
| [RetroArch](https://www.retroarch.com/?page=platforms) | Free | Multi-system frontend/emulator (Libretro cores) for many classic platforms. |
| Standalone emulators | Free | For systems where standalone emulators are preferred or no Libretro core is available. |
| [Steam (or similar)](https://store.steampowered.com/) | In-Pay | PC game client. |
| [AutoHotkey 1.1](https://www.autohotkey.com/) | Free | Automation scripts for unified hotkeys and small UI workarounds. |
| [Res-O-Matic](https://m.majorgeeks.com/files/details/res_o_matic.html) | Free | Forces a specific Windows resolution when launching an emulator/game. |
| [Visual C++](https://aka.ms/vs/17/release/vc_redist.x64.exe), [.NET Framework](https://dotnet.microsoft.com/en-us/download/dotnet-framework/net481), [DirectX](https://www.microsoft.com/en-us/download/details.aspx?id=35) | Free | Runtime libraries for emulator compatibility. |

#### Optional Software

- [iRotate](https://www.entechtaiwan.com/util/irotate.shtm): Display rotation utility, required for TATE / stacked handheld layouts.  
- [7-Zip](https://www.7-zip.org/): Manages emulator and ROM archives.  
- [Rufus](https://rufus.ie/en/#download/): Writes bootable Windows install media.  
- Proprietary controller drivers: May be required for some Windows gamepads (e.g., Hori Octa Pro).  
- [WinSCP](https://winscp.net/eng/index.php): Simplifies ROM transfers over a network (requires SSH setup on the PC).  
- Debloating tools ([O&O ShutUp10](https://www.oo-software.com/en/shutup10), [Winaero Tweaker](https://winaerotweaker.com/)): Reduce telemetry/popups and improve couch-only usability.  
- [Notepad++](https://notepad-plus-plus.org/downloads/): Handy for scripts and config editing.  
- [Bulk Crap Uninstaller](https://www.bcuninstaller.com/): Bulk uninstall and startup app management.

#### Troubleshooting Tools

- [Microsoft Show/Hide Updates](https://download.microsoft.com/download/f/2/2/f22d5fdb-59cd-4275-8c95-1be17bf70b21/wushowhide.diagcab): Hide a problematic Windows/driver update if something breaks.  
- [DDU](https://www.wagnardsoft.com/display-driver-uninstaller-ddu): Deep-clean GPU driver removal when normal uninstall/reinstall fails.  
- [AMD Cleanup Utility](https://www.amd.com/en/resources/support-articles/faqs/GPU-601.html): Official AMD tool to remove AMD graphics drivers/software.  
- [Vulkan SDK](https://vulkan.lunarg.com/sdk/home#windows): Only if Vulkan runtime components are missing on your system (unlikely with modern drivers).

---

## Video Signal Chain

```text
Video signal

[Mini PC]
Ryzen 9 8945HS
780M iGPU
│
└─[eGPU Dock via OCuLink]
│
└─[RX 6700 XT GPU]
│
└─[HDMI Output (Custom Resolutions via CRU)]
│
└─[RetroTink 4K]
(Scaler + CRT Shaders)
│
└─[LG OLED Monitor]
(1440p 60Hz*, 4:3 / 16:9 or TATE)
```

The system outputs native resolutions via HDMI, which are then upscaled by the scaler to a modern display. The diagram above shows the signal flow for the reference setup, but it can be adapted to other GPUs, scalers, or monitors.

- Note: In my configuration, I found 60 Hz produces cleaner motion than 120 Hz or higher on 2D/pixel retro systems.

In the example configuration:

- The mini PC (Ryzen 9 8945HS) and RX 6700 XT eGPU generate native resolutions.  
- The RetroTINK 4K scaler upscales to 1440p and applies CRT shaders for retro authenticity.  
- The LG OLED displays at 1440p (its native resolution), but a 4K monitor could replace it for higher output.

---

## Setup Process

Setting up the emulation station involves:

1. Preparing the PC.  
2. Configuring the GPU and drivers.  
3. Optimizing Windows.  
4. Enabling file transfers.

### Preparing the PC

1. **Gather Files**  
   Collect all required software (emulators, drivers, utilities) on a network drive or USB stick for easy access.

2. **BIOS Settings**  
   - Enter the BIOS and set the CPU to performance mode to maximize emulation power.  
   - Disable Secure Boot (or if you don't, remember to try it later if you run into driver installation or boot issues).

3. **Windows Installation**  
   - Prepare the Windows installer on a bootable drive with Rufus. Enable the “Windows User Experience” options to create a local account and skip privacy / data collection prompts.  
   - Install Windows offline first (without connecting to the internet) for a simpler setup and clean local account.  
   - Skip setting a password to enable auto-login, which is critical for gamepad-only operation (see [SSH section](#ssh-for-file-transfers)).

---

### GPU and Driver Setup

A GPU that supports EDID overrides/custom resolutions over HDMI (for CRU) and modern APIs like Vulkan is required for this setup.

**Known-working GPUs (tested):**

- AMD Radeon HD 7750  
- AMD RX 580  
- AMD RX 6700 XT  

(All were used as eGPU via OCuLink in the reference setup. Other recent AMD cards, and many NVIDIA cards, should behave similarly as long as they respect EDID overrides.)

**If custom resolutions don’t show up:**

- Run `restart64.exe` (included in CRU directory) or reboot to reload the graphics driver.  
- If the modes still don’t appear, try a different driver version; CRU depends on driver support for EDID overrides.


1. **Install Drivers**  
   - Download and install your GPU drivers (latest recommended).  
   - Optional (only if switching GPUs or troubleshooting broken installs): use AMD Cleanup Utility or DDU for a clean uninstall before reinstalling drivers.

2. **Driver Updates**  
   - With a stable GameMaze setup, you generally do not need to freeze GPU drivers or block updates.  
   - If Windows Update repeatedly replaces your GPU driver and breaks something, use Microsoft “Show/Hide Updates” to block that specific driver update.

### AMD GPU + HDMI Scaler: HDCP Fix (If Image Looks Wrong)

If your scaler output looks blurred or pixelated, disable HDCP in AMD Software: Adrenalin Edition.

![AMD HDCP issue](/images/DRM.jpg)

- Go to **Settings** (gear icon, top right) → **Display** → **Overrides** (expand) → **HDCP Support**.  
- Disable HDCP and accept the warning.

![AMD HDCP Fix](/images/amdhdcp.png)

---

### Windows Configuration

1. **Activate Windows**

2. **Power Settings**  
   - Win+I → System → Power & battery → Power mode → **Best performance**.  
   - Disable screensaver: Settings → Personalization → Lock screen → Screen saver → **None**.

3. **Disable Fast Startup**  
   Fast Startup is normally enabled in Windows 11 by default and can cause issues with emulator saves on shutdown.  
   - Control Panel → Hardware and Sound → Power Options.  
   - Click **Choose what the power buttons do**.  
   - Click **Change settings that are currently unavailable**.  
   - Under *Shutdown settings*, uncheck **Turn on fast startup (recommended)** → Save changes.

4. **Remove Bloat**  
   - Settings → Apps → uninstall unnecessary built-in apps and clutter.

5. **Game Bar Settings**  
   - Settings → Gaming → Xbox Game Bar → disable controller shortcuts (untick “Open Xbox Game Bar using this button on a controller”).

6. **Install Runtimes**  
   - Install Visual C++, .NET Framework, DirectX (and if needed Vulkan SDK).

7. **Update Windows**  
   - Run Windows Update, install all available updates, and reboot until no updates remain.

8. **Set Desktop Resolution**  
   - Choose a convenient desktop resolution (e.g., 1280×720). Many emulators will launch at desktop resolution by default, so this interacts with Res-O-Matic usage later.

---

## SSH for File Transfers

GameMaze is designed to be gamepad-focused, with no keyboard or mouse attached. This makes local file management (adding ROMs, programs, etc.) inconvenient.

To solve this, everything can be done on a main PC with keyboard and mouse, then transferred to the emulation PC over the network—no cables and no passwords on the emulation PC’s display after initial setup.

### Why Key-Based SSH?

- Preserves gamepad-only operation: You can keep Windows login password-free while still transferring files remotely.  
- Safer than blank passwords: Authentication is done using an SSH keypair (private key on your main PC, public key on the emulation PC).

### SSH Setup Steps

0. **Note the Emulation PC’s IP Address**  
   - Open Terminal/CMD and run:  
     ```cmd
     ipconfig
     ```  
   - Write down the IPv4 address of the active adapter.

1. **Still on the Emulation PC → Enable OpenSSH Server**

   - Open PowerShell as administrator and run:
     ```powershell
     Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
     ```
   - Verify installation:
     ```powershell
     Get-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
     ```

2. **Configure SSH Service**

   - Run `services.msc`.  
   - Find **OpenSSH SSH Server (sshd)**, set Startup type to **Automatic**, and click **Start**.  
   - Reboot.

3. **On Your Main PC → Generate a Key with WinSCP**

   - Open WinSCP → create/edit a session → click **Advanced…**  
   - Go to: SSH → Authentication.  
   - Click **Tools → Generate New Key Pair with PuTTYgen**.  
   - In PuTTYgen, choose **EdDSA (Ed25519)**, generate the key, and save the private key (`.ppk` file) somewhere safe.  
   - Back in WinSCP, click **Display Public Key** and copy the public key text (OpenSSH `authorized_keys` format).

4. **On the Emulation PC → Install the Public Key (Admin)**

   - Create a blank file named `administrators_authorized_keys` in the folder  
     `C:\ProgramData\ssh\`
 
   - Paste the public key as a single line into the file (one key per line) and save.

5. **Fix Permissions**

   - Open PowerShell as Administrator and run:
     ```powershell
     icacls.exe "C:\ProgramData\ssh\administrators_authorized_keys" /inheritance:r /grant "Administrators:F" /grant "SYSTEM:F"
     ```

6. **Configure SSH to Use Keys**

   - Edit `C:\ProgramData\ssh\sshd_config` and ensure:
     ```text
     PubkeyAuthentication yes
     PasswordAuthentication no
     ```
   - Restart the **OpenSSH SSH Server (sshd)** service in `services.msc` or reboot.

7. **On Your Main PC → Set up WinSCP Connection**

   - File protocol: **SFTP**  
   - Hostname: emulation PC IP  
   - Port: 22  
   - Username: emulation PC’s Windows admin user  
   - In **Advanced → SSH → Authentication**, set *Private key file* to the `.ppk` you saved.

### Alternative

Using a USB drive for file transfers is always possible, but is only recommended if SSH is not feasible.

### SSH Notes

- This configuration does not require the emulation PC to show a password prompt on screen; it only affects how WinSCP authenticates over the network.

---

## Resolution and Emulator Configuration

This section covers:

- Setting up native low/legacy resolutions (160p–384p) using Custom Resolution Utility (CRU).  
- Configuring emulators to output them correctly, targeting pixel-perfect visuals.

Even with EDID entries created via CRU, most emulators will still launch at desktop resolution. For those, [Res-O-Matic](#ssh-for-file-transfers) is used in LaunchBox or BigBox to force the desired resolution.

---

### Resolution Setup with CRU

The CRU resolutions below are the *custom* low modes used for retro systems. Standard HDMI modes like 480p and 720p **do not require** CRU and can be selected normally per emulator/game as needed.

#### Resolution Table (CRU)

| Resolution | Active (H×V) | Front Porch (H/V) | Sync Width (H/V) | Blanking (H/V) | Polarity (H/V) | Refresh Rate (Hz) |
|:----------:|:------------:|:-----------------:|:----------------:|:--------------:|:--------------:|:-----------------:|
| 160p       | 2560×160     | 16/3              | 32/4             | 80/12          | -/-            | 60.011            |
| 224p       | 2560×224     | 16/3              | 32/4             | 80/14          | -/-            | 59.985            |
| 240p       | 2560×240     | 16/3              | 32/4             | 80/15          | -/-            | 60.011            |
| 256p       | 2560×256     | 16/3              | 32/4             | 80/16          | -/-            | 59.840            |
| 272p       | 2560×272     | 16/3              | 32/4             | 80/17          | -/-            | 60.003            |
| 384p       | 2560×384     | 16/5              | 32/6             | 80/24          | -/-            | 60.002            |

#### Resolution Notes

- **CRU slot budget**:  
  CRU slots are limited, so this list prioritizes the modes actually used (160p/224p/240p/256p/272p/384p). If you want pixel-perfect 144p for Game Boy, you need to sacrifice another mode; otherwise RetroArch CRT SwitchRes will pick the closest available resolution.

- **Refresh-rate variations (optional)**:  
  Slight differences (e.g., 59.985 vs 60.000 Hz) *seems* to help RetroArch CRT SwitchRes consistently select the intended resolution when multiple modes are close (e.g., 224p vs 240p). 

- **480p/720p/1080p are not CRU topics**:  
  480p and above are standard HDMI modes. Use them normally for systems/emulators that are already 480p+ (e.g., PS2/Dreamcast/3DS stacked layouts) without using CRU slots.

- **Horizontal timings**:  
  All resolutions share the same horizontal settings (Active: 2560, Front Porch: 16, Sync Width: 32, Blanking: 80, Polarity: -).

![240p Resolution](/images/240.PNG)

---

### CRU Setup Steps

1. **Open CRU**  
   - Launch CRU and confirm your scaler/display is selected (top left, marked “active”). If not, choose it from the dropdown.

2. **Clear Defaults**  
   - In **Detailed Resolutions** (top right), delete existing resolutions to free up three slots.

3. **Add Resolutions (6-slot layout)**  
   - Click **Add → Manual timing**, and enter settings from the table above (e.g., 2560×224 for 224p).  
   - In **Detailed Resolutions** (up to three slots): add **160p, 224p, 240p**.  
   - In **Extension Blocks → CTA-861 → Edit**: add **256p, 272p, 384p**.

4. **Save and Restart**  
   - Click **OK** twice to save changes.  
   - Run `restart64.exe` (in the CRU folder) to apply the new resolutions.

5. **Verify**  
   - Test 224p and 240p resolution in RetroArch with CRT SwitchRes active:  
     Settings → Video → CRT SwitchRes → 15 kHz / Super resolution 2560.
   - Other legacy resolution will likely not be recognized by CRT SwitchRes and require [Res-O-Matic](#ssh-for-file-transfers).

---

### Emulator Configuration

The goal is to preserve each system’s original signal and let the scaler handle all upscaling and CRT-style processing. Emulators should:

- Output at native resolution wherever possible.  
- Avoid internal smoothing or filtering.  
- Use internal resolution scaling for 3D content.

---

### Resolution Handling

Emulators handle resolution in three main ways:

1. **RetroArch with CRT SwitchRes**  
   - Automatically switches to 224p/240p for NES/SNES/Megadrive/PS1/arcade.  
   - No Res-O-Matic needed.

2. **Desktop Resolution Default (most common)**  
   - Launches at Windows desktop resolution.  
   - Requires Res-O-Matic to force native resolutions (RetroArch without SwitchRes, most standalone emulators).

3. **Internal Configuration**  
   - Resolution set via config files or in-game settings.  
   - No Res-O-Matic needed.
   
*The following table summurizes the required method to get the right resolutuion per system:*
| System                      | Emulator              | Native Res     | Method          | Notes                                              |
|-----------------------------|-----------------------|----------------|-----------------|----------------------------------------------------|
| NES, SNES, Megadrive        | RetroArch             | 224p-240p      | CRT SwitchRes   | Automatic resolution switching                     |
| PS1                         | RetroArch             | 240p           | CRT SwitchRes   | Automatic resolution switching                     |
| Game Boy                    | RetroArch             | 144p           | Res-O-Matic     | SwitchRes disabled                                 |
| GBA                         | RetroArch             | 160p           | Res-O-Matic     | SwitchRes disabled                                 |
| Nintendo DS                 | RetroArch (DeSmuME)   | 256p           | Res-O-Matic     | Stacked screens, SwitchRes disabled                |
| PSP                         | PPSSPP                | 272p           | Res-O-Matic     | Use if desktop ≠ 272p                              |
| Dreamcast                   | RetroArch (Flycast)   | 480p           | Res-O-Matic     | SwitchRes disabled                                 |
| GameCube                    | RetroArch (Dolphin)   | 480p           | Res-O-Matic     | SwitchRes disabled                                 |
| PS2                         | PCSX2                 | 480p           | Res-O-Matic     | Use if desktop ≠ 480p                              |
| 3DS                         | Azahar                | 400p/800p      | Res-O-Matic     | Or use desktop resolution                          |
| PS3                         | RPCS3                 | 720p           | Res-O-Matic     | Use if desktop ≠ 720p                              |
| PS Vita                     | Vita3K                | 544p           | Res-O-Matic     | Or use desktop resolution                          |
| Switch                      | Ryujinx               | 720p           | Res-O-Matic     | Use if desktop ≠ 720p                              |
| Arcade (FBNeo/etc. cores)   | RetroArch             | 224p–240p      | CRT SwitchRes   | Automatic, handles TATE                            |
| Arcade (various MAME)       | MAME standalone       | 224p-480p      | Config file     | Set per-system in `.ini` files                     |
| Sega Model 3                | Supermodel            | 384p           | Config file     | Set in `Supermodel.ini`                            |
| Steam/PC games              | N/A                   | 720+           | In-game settings| Configure in graphics options                      |

**For legacy systems (pre-480p, before Dreamcast)**:  
Use nearest-neighbor filtering with no smoothing (avoid SuperEagle, 2xBRZ, etc.). Keep internal resolution at 1× for 2D/pixel art to preserve accuracy. For 3D titles (PS1, N64), internal resolution scaling (2×–3×) with anti-aliasing can improve visuals while maintaining native *output* resolution.

**For 480p+ systems (Dreamcast onwards)**:  
Internal resolution scaling (2×–3×) and anti-aliasing are safe and look great, especially for 3D content. Always output at native resolution (480p, 720p, etc.) using Res-O-Matic where necessary.

RetroArch is used for most retro systems, while standalone emulators are preferred for newer platforms. When RetroArch’s CRT SwitchRes is not suitable (e.g., vertically stacked DS or 480p+ content), [Res-O-Matic](#using-res-o-matic-for-custom-resolutions) is used to launch games at the desired resolution.

---

#### RetroArch Configuration

1. **General Settings**

   - Go to Settings → User Interface → Menu → select **rgui**.  
     It may not be the sexiest skin, but works best with low resolutions.

   - **Set Controller Drivers (Gamepad Harmonization)**:  
     - Main Menu → Settings → Drivers:  
       - **Input**: `dinput`  
       - **Controller**: `dinput`  
     - Main Menu → Configuration File → **Save Current Configuration**

   **Why dinput?**

   - **Reconnection reliability**: `SDL2` has known Bluetooth reconnection issues.  
   - **Unified hotkeys**: `xinput` binds the Home button to toggle RetroArch’s menu, intercepting it before reWASD can process Home+button combos. This breaks harmonized hotkeys (e.g., Home+Start to quit) in RetroArch while they work in other emulators. `dinput` treats Home as a regular button, enabling consistent hotkey behavior across the system.

2. **Core Options and Aspect Ratio**

For each core:

- Load a game, open the menu, and navigate to:  
  - Video → Scaling → Aspect Ratio: **Core Provided**
- Save core overrides:  
  - Quick Menu → Overrides → **Save Core Overrides**

**When to use "Full" instead or "Core Provided":**

If a game looks stretched or incorrect (especially vertical arcade games), switch Aspect Ratio to **Full**. If a specific game needs a different ratio from other games on the same platform, save its ratio as a **Game Override** instead of a Core Override.

Using "Full" is safe because the RetroTINK 4K profile is set to 4:3, preventing unintended stretching for legacy systems. Standard HD resolutions (720p, 1080p) are hardcoded in the RT4K firmware to force 16:9 regardless of profile settings—this is why Vita (if running at desktop 720p or 1080p) and modern systems display correctly in 16:9, even with a 4:3 scaler profile.

4. **PS1 (Mednafen PSX Core)**

   - Settings → Video → CRT SwitchRes:  
     - Enable, set to 15 kHz, 2560 horizontal.  
   - Aspect Ratio: **Core Provided**.  
   - Save core overrides.

![PS1 Example](/images/crono.jpg)  
*Chrono Trigger PS1*

5. **Arcade (FBNeo Core)**

   - Same CRT SwitchRes settings as PS1.  
   - Quick Menu → Core Options → Vertical Mode: **TATE** or **TATE Alternate** (affects only vertical games).  
   - Aspect Ratio: **Core Provided** (or **Full** if stretched).  
   - Save core overrides; if a specific game requires a unique ratio, save it as a game override.

6. **Nintendo DS (DeSmuME Core)**

   - Disable CRT SwitchRes (2560×256 is not properly detected by SwitchRes).  
   - Aspect Ratio: **Full**.  
   - Core Options → Screen Layout: **Bottom/Top** or **Top/Bottom** as preferred.  
   - Save core overrides.  
   - Use [Res-O-Matic](#ssh-for-file-transfers) in LaunchBox to force 2560×256.

7. **GameCube (Dolphin Core), Dreamcast (Flycast Core)**

   - Disable CRT SwitchRes (RetroArch defaults to desktop resolution).  
   - Aspect Ratio: **Core Provided** (or adjusted as needed).  
   - Use [Res-O-Matic](#using-res-o-matic-for-custom-resolutions) to force 640×480 (480p) in LaunchBox.

![FF5GBA](/images/FF5.jpeg)  
*Final Fantasy 5 on GBA at 160p*

---

#### Standalone Emulator Configuration

Standalone emulators handle modern systems (PS2, Switch, etc.) and some retro platforms. Configure them to output native resolutions.

1. **MAME 0.272 (Per-System Settings, e.g., Namco, ST-V, SNK Hyper 64)**

   Example for ST-V (224p):

   - Run a game in MAME once and close it.  
   - Create `/MAME/ini/stvbios.ini` containing:
     ```ini
     [stvbios]
     resolution        2560x224@60
     aspect            4:3
     rotate            0
     unevenstretch     1
     switchres         1
     ```
   - For vertical games (e.g., Shienryu), create `/MAME/shienryu.ini` containing:
     ```ini
     rotate            1
     ```
   - Launch the game, adjust rotation if needed, then save *Game* settings (not system settings).

   For other platforms (Namco, SNK), you can repeat this process and use a 640×480@60 resolution where appropriate.

2. **Supermodel (Sega Model 3)**

   - Obtain the Supermodel.ini (often configured for XInput) and NVRAM files from the excellent [Warped Polygon thread](https://forums.launchbox-app.com/files/file/3857-sega-model-3-supermodel-git-everything-pre-configured-inc-controls-for-pc-controller-mouse-light-guns-test-menus-configured-free-play-all-games-in-english-2-player-mouse-support-audio-adjusted-layout-imagesthe-whole-9-yards).  
   - Follow Warped Polygon’s setup instructions (But no GUI needed, to preserve 384p).  
   - Edit `/supermodel/Config/Supermodel.ini`:
     ```ini
     FullScreen=1
     Throttle=1
     XResolution=2480
     YResolution=384
     Stretch=True
     ```

   *Note*: Model 3 games natively render at 496×384, which is not 4:3 but a slimmer 1.2917:1 aspect ratio. To preserve this ratio when your scaler forces 4:3:

   - In CRU: create **2560×384** (not 2480×384).  
   - In Supermodel.ini: set `2480×384` with `Stretch=True`.  
   - On the scaler: set aspect mode to **4:3**.
  
   Supermodel renders at 2480-wide (the correct pre-compensated width) but outputs to the closest available EDID timing (2560×384), automatically adding 80 pixels of black bars (40 each side). When your scaler applies 4:3 scaling, the active content scales to the original Model 3 ratio.

   ⚠️ This relies on Supermodel’s behavior (stretch to INI resolution, output to closest EDID with black bars). Do **not** create 2480×384 in CRU, it must remain “unavailable” for this trick to work.

3. **Azahar (3DS)**

   - Set View → Screen → **Rotate Up Right** for vertical stacked screens.  
   - If the screen flips in the wrong direction for your setup, you will have to use iRotate instead (See below image).
   - By default, Azahar runs at desktop resolution; use [Res-O-Matic](#using-res-o-matic-for-custom-resolutions) in LaunchBox for custom resolutions.

4. **PCSX2 (PS2), RPCS3 (PS3), Ryujinx (Switch), Vita3K (PS Vita)**

   - **RPCS3**: On Windows 11 with recent GPU drivers, current RPCS3 builds should work well.  
     - Prefer **Vulkan** when available (often best-performing), unless per-game settings suggest otherwise.

   - **PCSX2**:  
     - Use the latest builds.  
     - Renderer: Vulkan (if supported by your GPU/driver; otherwise D3D11/D3D12 or OpenGL).  
     - Scaling: Use 2×–3× internal resolution selectively when it improves image quality. Keep filtering minimal to preserve a faithful low-res look.

   - **Output Resolution**:  
     - Set each emulator to output the native mode you want (e.g., 480p content at 480p), and only use higher output modes when you have a specific reason (compatibility or CRU slot limit reached).  
     - For resolutions different from your desktop resolution, use [Res-O-Matic](#using-res-o-matic-for-custom-resolutions).

5. **PPSSPP (PSP)**

   - Launch the emulator at 272p using Res-O-Matic in LaunchBox (see [Using Res-O-Matic](#using-res-o-matic-for-custom-resolutions)).  
   - PSP’s native aspect ratio is 30:17 (≈1.7647:1), slightly off from 16:9. To preserve this perfectly:  
     - Set your scaler to 16:9.  
     - Edit `ppsspp.ini` (typically in `C:\Users\<YourAccount>\Documents\PPSSPP\PSP\SYSTEM\`):
       ```ini
       [Graphics]
       DisplayAspectRatio = 5.294000
       DisplayStretch = False
       ```
     This adds *very* slim black bars on the sides, pre-compensating for the scaler’s 16:9 stretch to display the correct PSP aspect ratio.

6. **Vertical Mode (iRotate Example)**

   If you need external tools to rotate your display for a specific emulator, you can use iRotate in a LaunchBox script:

![irotate](/images/rotate.png)

```autohotkey
; Rotate to portrait when emulator starts
Send ^!{Left}   ; Ctrl + Alt + Left for portrait rotation
Sleep, 500

; Override Escape behavior while Azahar is running
#IfWinActive ahk_exe Azahar.exe
$Esc::
    Send ^!{Up}                      ; Rotate back to landscape
    Sleep, 200                       ; Small buffer
    WinClose, ahk_exe Azahar.exe     ; Ask Azahar to close
    Sleep, 500
    if WinExist("ahk_exe Azahar.exe")
        Process, Close, Azahar.exe   ; Force kill if needed
return
#IfWinActive
```

*This script assumes Escape is the emulator shutdown key. Adjust the keybinding if different. Also replace "Azahar.exe" by the executable of your emulator.*

---

### Steam

After installing Steam and logging into your account:

- Steam → Settings → Controller:  
  - Disable **Guide Button Focuses Steam**.  
  - Disable **Enable Guide Button Chord for Controller**.

If you want games to install locally on the emulation PC (so they are available without network connectivity):

- Steam → Settings → Remote Play → turn **OFF** Remote Play.  
  Otherwise, if your main PC is on, Steam may launch games via streaming instead of local installs, mirroring the game on both machines.

---

## reWASD and LaunchBox Setup

This section configures reWASD to standardize gamepad inputs across all emulators and LaunchBox/BigBox to unify emulators, Steam, and scripts into a seamless, gamepad-only experience.

The reference setup uses a mix of two Sony DualSense, a Hori Octa Pro, and an Xbox Series X controller, but any wired or wireless gamepads will work.

### reWASD Setup

Connect all controllers and start reWASD.

- **Profile Creation**  
  - Delete default profiles if you do not need them and create a new one (e.g., “BigBox”).  
  - Select a gamepad from the bottom icons (e.g., DualSense).

- **Set Output to Virtual Xbox**  
  - Click the Xbox icon (top center, “Output Devices Settings”).  
  - Enable **Virtual Xbox 360** (or the XInput/SDL variant of your choice) and save.

![reWASD Output Settings](/images/rewasdOUT.png)

- **Configure Hotkeys**  
  - Click the Home button (PS or Xbox button) on the gamepad schematic.  
  - Select **Shift Mode → Jump to Layer 1**.  
  - Switch to **Shift Layer 1** (top center) and map:
    - South (X on PlayStation, A on Xbox) → `TAB` (opens emulator menus).  
    - Start → `ESC` (quits emulators).  
    - Adjust to your preferred keys if desired.  
  - Save the profile.

- **Apply to All Gamepads**  
  - Select each gamepad and click **Apply to Slot 1**, **Slot 2**, etc.  
  
- **Configure reWASD settings**   
  - Ensure reWASD starts with Windows: Settings (gear, top right) → General → **Start with Windows**.
  - Ensure keys remapping is always active: Settings → tray agent → and tick "Restore remap on startup".
  - In reWASD settings, you can also tweak **Overlay** options to disable unwanted notifications.

- **Test in Emulators**  
  - Open RetroArch and each standalone emulator to verify controls.  
  - If needed, rebind in emulator settings, selecting XInput or SDL as input type.

### Proposed Controller Mapping

To ensure consistent controls across all emulators and BigBox:

| Button Combo     | Action | Purpose                                         |
|------------------|--------|-------------------------------------------------|
| Home + South     | TAB    | Opens emulator configuration/menu.              |
| Home + Start     | ESC    | Quits the emulator or game.                     |
| Standard buttons | Native | Bound to in-game controls (per emulator).       |

You can customize these combos in reWASD (e.g., Home + Select = TAB).  
Ensure all emulators respond to TAB (menu) and ESC (quit), or use LaunchBox scripts to enforce this where native binding is not available. (See the next part)

---

### LaunchBox Configuration

This is not a full LaunchBox tutorial; the focus is on unified quitting, custom resolutions, and popup handling.

- **Install LaunchBox/BigBox**  
  - Download and install LaunchBox with BigBox for gamepad-only navigation.  
  - Import ROMs and configure emulators (see LaunchBox documentation for details).

- **Unified Quit Control**

  - For emulators that support custom quit keys (e.g., PPSSPP):  
    - Set `ESC` as the quit key in emulator settings.  
    - Disable “Confirm Quit” to ensure instant closure.

  - For emulators without custom quit keys (e.g., RPCS3):  
    - In LaunchBox: Tools → Manage → Emulators → **RPCS3** → Edit → **Running Script**.  
    - Add:
      ```autohotkey
      $Esc::
      {
          WinClose, ahk_exe rpcs3.exe
          Sleep 100
          If WinExist("ahk_exe rpcs3.exe")
              Process, Close, rpcs3.exe
      }
      Return
      ```
    - Repeat for other emulators (replacing `rpcs3.exe` with the emulator executable name).

![LaunchBox Quit Script](/images/close.png)

---

## Using Res-O-Matic for Custom Resolutions

Res-O-Matic forces a specific launch resolution outside RetroArch’s CRT SwitchRes workflow.

It is especially useful when:

- CRT SwitchRes picks an unexpected mode, or  
- You need to launch at a non-desktop resolution (e.g., 480p) with standalone emulators.

The approach is to create `.bat` launchers in emulator folders and configure these as alternative “emulators” in LaunchBox.

**For the following examples:**
- Adjust paths and core names to match your setup. 
- Check the exact core naming in `RetroArch\cores\`.  
- `reso.exe` is the executable of Res-O-Matic found in your installation directory.
- Create similar scripts for any system needing a forced resolution.


- **Example: Nintendo DS (2560×256)**

  - Create `C:\RetroArch\retroarch_ds_256p.bat` containing:
    ```batch
    @echo off
    echo start /wait "" "C:\Emulators\RetroArch\retroarch.exe" -L "cores\desmume_libretro.dll" %* > "%TEMP%\launch_ds.bat"
    start /wait "" "C:\YourPath\reso.exe" "%TEMP%\launch_ds.bat" 2560 256 32 60
    del "%TEMP%\launch_ds.bat"
    ```
  - In LaunchBox, add as a new emulator (e.g., “RetroArch DS 256p”), with:
    - Application path: `retroarch_ds_256p.bat`  
    - Associate it with the Nintendo DS platform.

- **Example: PPSSPP (2560×272)**

  - Create `PSP_272p.bat`:
    ```batch
    @echo off
    echo start /wait "" "C:\YourPath\PPSSPPWindows64.exe" %* > "%TEMP%\launch_psp_new.bat"
    :: Use reso.exe to run at 2560x272
    start /wait "" "C:\YourPath\reso.exe" "%TEMP%\launch_psp_new.bat" 2560 272 32 60
    :: Clean up
    del "%TEMP%\launch_psp_new.bat"
    ```
  - Add `PSP_272p.bat` as a new emulator in LaunchBox for the GameCube platform.

![Nintendo DS Emulator](/images/csEmul.png)

- **Example: GameCube (640×480)**

  - Create `C:\RetroArch\retroarch_dolphin_480p.bat`:
    ```batch
    @echo off
    echo start /wait "" "C:\Emulators\RetroArch\retroarch.exe" -L "cores\dolphin_libretro.dll" %* > "%TEMP%\launch_dl.bat"
    start /wait "" "C:\YourPath\reso.exe" "%TEMP%\launch_dl.bat" 640 480 32 60
    del "%TEMP%\launch_dl.bat"
    ```
  - Add `retroarch_dolphin_480p.bat` as a new emulator in LaunchBox for the GameCube platform.


---

## Dismissing Popups with WindowSpy Fix

Any dialog that requires a mouse click when launching a game breaks the gamepad-only experience.

On a fully updated system (Windows, drivers, games), this should be rare. If it does happen, try LaunchBox options first, then AutoHotkey.

### Try LaunchBox Options First

In LaunchBox:

- Tools → Manage Emulators → select emulator → Details:  
  - Enable **Attempt to hide console window on startup/shutdown**.  
  - Under **Startup Screen**:  
    - Enable **Aggressive Startup Window Hiding** (helps hide transient launcher windows).  
  - Enable **Hide All Windows that are not in Exclusive Fullscreen Mode** to keep BigBox in focus.

These options can handle some cases, but modal Windows dialogs will still require scripting.

### If LaunchBox Options Don’t Work: AutoHotkey + WindowSpy

1. Install AutoHotkey 1.1 and run `WindowSpy.ahk` (included in the same folder) to identify the popup’s window title and `ahk_class`.  
2. Create a wrapper script that launches the game, waits for the popup, and auto-closes it (Alt+F4, Enter, or a coordinate click).  
3. In LaunchBox, point the game’s Application Path to the wrapper script instead of the executable.

**Example structure:**

Because Launchbox cannot accept .ahk script as emulators, but can accept .bat files:
- Create `ROMs/SteamYourGame.bat`:
  ```batch
  @echo off
  start "" "YourGamePopupKiller.ahk"
  ```

- Create `YourGamePopupKiller.ahk` in the same folder:
  ```autohotkey
  Run, steam://rungameid/YOUR_GAME_ID
  Sleep, 5000
  Loop {
      IfWinExist, Popup Window Title ahk_class YOUR_CLASS
          WinClose
      Sleep, 500
  }
  ```

In LaunchBox, set the game’s Application Path to `SteamyourGame.bat`.

- *Notes*:  

- **Script Paths**:  
  Adjust all script paths and window titles/classes to match your actual system and target popups (e.g., Forza Horizon 5 vs another game).

- **Popup Workarounds**:  
  Use `WindowSpy.ahk` to confirm the correct `ahk_class` and title if a script doesn’t behave as expected.

---

## Controller Reordering Workaround

Windows does not offer native controller reordering. To avoid needing a keyboard/mouse, a Bluetooth toggle script can be used to reset controller order from BigBox.

1. **Create Toggle Script**

   Create `C:\Emulators\Bluetooth\ToggleBluetooth.bat`:

   ```batch
   @echo off
   net session >nul 2>&1
   if %errorlevel% neq 0 (
       powershell -Command "Start-Process cmd -ArgumentList '/c %~f0' -Verb RunAs" >nul 2>&1
       exit
   )
   start /min powershell -NoProfile -ExecutionPolicy Bypass -Command "$ProgressPreference = 'SilentlyContinue'; $bt = Get-PnpDevice -FriendlyName '*Bluetooth*' | Where-Object {$_.HardwareID -like '*BTH\\MS_BTHBRB*'}; $bt | Disable-PnpDevice -Confirm:$false; Start-Sleep -Milliseconds 500; $bt | Enable-PnpDevice -Confirm:$false; Start-Sleep -Milliseconds 500" >nul 2>&1
   exit
   ```

2. **Create a Dummy ROM File**

   - Create an empty file: `C:\Emulators\Bluetooth\BT.bin`.

3. **Configure in LaunchBox**

   - Create a new platform (e.g., **BT-Toggle**).  
   - Add a new emulator: Tools → Manage → Emulators → Add.  
     - Name: “Gamepad Reorder”.  
     - Application Path: `ToggleBluetooth.bat`.  
     - Platform: “BT-Toggle”.  
   - Add a ROM:  
     - Platform: “BT-Toggle”.  
     - File: `BT.bin`.

In BigBox, launching the “BT-Toggle” fake ROM triggers the Bluetooth toggle, letting you reconnect controllers in your desired order (Player 1 first, Player 2 second, etc.).

To make it blend into your BigBox theme:

- In LaunchBox, filter by platform → BT-Toggle → right-click → Edit metadata.  
- Add images (e.g., Bluetooth logo, gamepads) as banner / clear logo override / device / fanart, depending on your theme.

![LaunchBox Platforms](/images/platforms.png)  
![BT-Toggle Platform](/images/BTplatform.png)  
![LaunchBox Interface](/images/LB.png)

---

## Final Touches

With emulators, resolutions, and gamepad controls configured, the last step is to optimize Windows for maximum performance and a distraction-free, gamepad-only experience, as close as possible to a dedicated gaming console.

### Debloating Windows

The goal is to reduce background noise (telemetry, notifications, bundled apps) and eliminate UI interruptions that break the gamepad-only flow.

#### Option A, Manual

Tools like O&O ShutUp10 and Winaero Tweaker simplify manual debloating.

- **Install Debloating Tools**  
  - Download and run **O&O ShutUp10** and **Winaero Tweaker** as administrator.

- **O&O ShutUp10 Configuration**  
  - Select “Recommended Settings” for a balanced profile, or customize:  
    - Disable telemetry (Privacy → Telemetry) to reduce background data exchange with Microsoft.  
    - Turn off Windows Update notifications to avoid surprise prompts.  
    - Disable Cortana and unnecessary background apps to reduce CPU usage.  
  - Apply changes and reboot.

- **Winaero Tweaker Configuration**  
  - Common tweaks:
    - Disable lockscreen (so you can always return to BigBox with only a controller).  
    - Disable Action Center to avoid notification popups during gameplay.  
    - Disable SmartScreen for app launches (reasonable on a dedicated emulation PC).  
  - Apply changes and reboot.

#### Option B, Automated

**Chris Titus Tech WinUtil (all-in-one)**

1. Open PowerShell **as Administrator** and run:
   ```powershell
   irm "https://christitus.com/win" | iex
   ```
2. Use the interface to apply a debloat/tweak preset and optionally install common apps via WinGet.

---

### Additional Optimizations

These tweaks polish the experience, ensuring fast boot into BigBox and clean performance.

- **Disable Startup Apps**  
  - Task Manager (Ctrl+Shift+Esc) → Startup tab.  
  - Disable all non-essential apps (e.g., OneDrive, browser updaters), keeping reWASD and LaunchBox if they are set to auto-start.

- **BigBox as Shell (Optional)**  
  - In BigBox: Options → General → **Enable using as the Windows Shell**.  
  - This replaces Explorer (no standard desktop), which is ideal for a pure console-like experience but inconvenient if you frequently use Explorer, browsers, keyboard, or mouse.  
  - If you enable shell mode, do **not** disable the Ctrl+Alt+Del shortcut: you may need it to open Task Manager or sign out if BigBox ever crashes.
  
- **Optimize Storage**  
  - Use **Bulk Crap Uninstaller** (BCU) for bulk uninstalls and leftover cleanup.   
  - Put ROMs and games on a SSD for reduced load times.

- **Backup Configuration**  
  - Back up LaunchBox settings (Tools → Backup) and reWASD profiles to a USB drive or network share.  
  - Save CRU `.bin` congif (export) and emulator config folders for quick restoration.

### Optimization Notes

- **Security Context**  
  This setup targets a dedicated emulation PC on a home network. Disabling telemetry is mainly a privacy and noise reduction choice.  
  However, disabling SmartScreen does reduce actual protection against malicious or unknown downloads. Do this if you understand and accept the trade-off.

---

## Conclusion

This project is about more than just running games. GameMaze brings together multiple generations of hardware and software into a single, consistent, console-like experience. Many small configuration choices and workarounds combine to deliver:

- Native-resolution output across decades of systems.  
- Authentic CRT-style visuals via a modern digital chain.  
- Seamless gamepad-only control from boot to shutdown.  
- Unified access to emulation, arcade, and modern PC titles in one library.

If you are facing similar challenges—wanting both authenticity and convenience on modern displays—this approach should give you a solid, repeatable foundation to build on and customize for your own hardware and preferences.
