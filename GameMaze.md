# GameMaze
Native Resolutions • Seamless Gamepad Control • Console-Like Experience for scalers on modern displays

Imagine launching Pac-Man in its original 224p arcade resolution, then switching to The Witcher 3 at 720; all from your couch, using only a gamepad… No keyboard, no fuss. This project is a passion-driven emulation hub that unifies 40+ years of gaming history, from 1980s arcade classics to modern Switch and Steam titles, in their original resolutions. It’s a console-like experience that prioritizes visual fidelity, ease of use, and minimum compromises.


# Table of Contents
- [Overview](#overview)
  - [Key Features](#key-features)
- [For Digital output (not CRT)](#for-digital-output-not-crt)
- [Supported Systems and Resolutions](#supported-systems-and-resolutions)
- [Why Build It?](#why-build-it)
- [Challenges & Constraints](#challenges--constraints)
- [Requirements and Software](#requirements-and-software)
  - [Hardware Requirements](#hardware-requirements)
  - [Software Requirements](#software-requirements)
    - [Essential Software](#essential-software)
    - [Optional Software](#optional-software)
    - [Troubleshooting Tools](#troubleshooting-tools)
    - [Additional Recommendations](#additional-recommendations)
- [Video Signal Chain](#video-signal-chain)
- [Setup Process](#setup-process)
  - [Preparing the PC](#preparing-the-pc)
  - [GPU and Driver Setup](#gpu-and-driver-setup)
    - [AMD GPU + HDMI Scaler: HDCP fix (if image looks wrong)](#amd-gpu--hdmi-scaler-hdcp-fix-if-image-looks-wrong)
  - [Windows Configuration](#windows-configuration)
- [SSH for File Transfers](#ssh-for-file-transfers)
  - [Why key-based SSH?](#why-key-based-ssh)
  - [SSH Setup Steps](#ssh-setup-steps)
  - [Alternative](#alternative)
  - [SSH Notes](#ssh-notes)
- [Resolution and Emulator Configuration](#resolution-and-emulator-configuration)
  - [Resolution Setup with CRU](#resolution-setup-with-cru)
    - [Resolution Table (CRU)](#resolution-table-cru)
    - [Resolution Notes](#resolution-notes)
  - [CRU Setup Steps](#cru-setup-steps)
  - [Emulator Configuration](#emulator-configuration)
    - [RetroArch Configuration](#retroarch-configuration)
    - [Standalone Emulator Configuration](#standalone-emulator-configuration)
  - [Steam](#steam)
- [REWASD and LaunchBox Setup](#rewasd-and-launchbox-setup)
  - [REWASD Setup](#rewasd-setup)
  - [Proposed Controller Mapping](#proposed-controller-mapping)
  - [LaunchBox Configuration](#launchbox-configuration)
- [Using Res-O-Matic for Custom Resolutions](#using-res-o-matic-for-custom-resolutions)
- [Dismissing Popups (Optional)](#dismissing-popups-optional)
  - [Try LaunchBox options first](#try-launchbox-options-first)
  - [Example: closing a stubborn warning (AutoHotkey)](#example-closing-a-stubborn-warning-autohotkey)
  - [LaunchBox Notes](#launchbox-notes)
- [Controller Reordering Workaround](#controller-reordering-workaround)
- [Final Touches](#final-touches)
  - [Debloating Windows](#debloating-windows)
    - [Option A, Manual:](#option-a-manual)
    - [Option B, Automated:](#option-b-automated)
  - [Additional Optimizations](#additional-optimizations)
  - [Optimization Notes](#optimization-notes)
- [Conclusion](#conclusion)


## Overview
The idea is to create a unified emulation platform that preserves the authentic visual experience of retro gaming while providing modern convenience: Instead of separate devices for different eras or accepting visual compromises, you get one system that handles everything from 1980s arcade cabinets to current-generation Switch games - all displaying at their original resolutions before being properly upscaled by dedicated hardware. The key breakthrough is outputting true native resolutions (tested from 160p up to 1080p) over HDMI to modern scalers, something most PCs and emulation distributions can't achieve. Combined with gamepad-only operation through BigBox and standardized controls via REWASD, the result feels like using a premium retro console rather than managing a collection of emulators on a computer. The system aims at removing the usual drawbacks of PC emulation: no keyboard/mouse dependency, no resolution mismatches, no juggling between different interfaces. You boot directly into a unified game library, select any title from four decades of gaming, and it launches at the correct resolution with consistent controls - exactly as these games were meant to be experienced, but on modern displays.


![DQ6](/images/DQ6.jpg)
*Dragon Quest 6 on SNES in native resolution*

### Key Features 
- Pixel-Perfect Visuals: Every game runs at its native resolution, upscaled to a modern display with CRT shaders. 
- Seamless Gamepad Control: Navigate menus and play games (tested with four controllers), standardized across all systems with consistent hotkeys (e.g., Home + Start to quit). 
- Unified Frontend: LaunchBox/BigBox offers a polished, gamepad-only interface, integrating standalone emulators and Steam for a cohesive experience. 
- Versatile Output: Supports both horizontal and vertical (TATE) arcade games, with flexible output to modern displays via HDMI.

The system is thought to output to any HDMI-compatible scaler, supporting up to 4K with a suitable monitor.

In my setup, I use a RetroTink 4K scaler to upscale the PC’s native inputs to 1440p (limited by my LG 27GS95QE-B OLED monitor), applying CRT shaders for a crisp, scanline-rich image. Vertical arcade games are displayed in TATE mode by rotating the monitor on a VESA stand.


## For Digital output (not CRT)
This guide is for getting authentic retro resolutions from a Windows emulation PC over digital video (HDMI/DisplayPort) to a modern flat panel. For best results, a low-latency scaler is required in the chain (e.g., a RetroTINK) before the display.

This is not a guide for analog output to CRT TVs/monitors (VGA/SCART/component) and it won’t cover CRT-specific constraints or setup paths.

Why that distinction matters
Most “240p on PC” legacy guides focus on CRT EmuDriver and specific GPU compatibility, because analog output depends heavily on driver + DAC/output constraints. With an all-digital chain, the system relies on EDID + custom timings and “super resolutions” (e.g., 2560 × original height), which are configured at the display-timing level and work across a wider range of modern GPUs.

I’ve tested this setup on an AMD RX 6700 XT, and it should work with many newer cards beyond the usual CRT EmuDriver lists.



## Supported Systems and Resolutions
Native resolutions for each platform (4:3 and 16:9 depending on the system running).

| System                | Native Resolution        |
|-----------------------|--------------------------|
| Game Boy Advance      | 160p                     |
| NES, SNES, Megadrive  | 224p–240p                |
| Arcade (MAME, FBNeo)  | 224p–384p                |
| Nintendo DS           | 256p (dual screen stack) |
| PSP                   | 272p                     |
| Sega Model 3          | 384p                     |
| Dreamcast, PS2, Wii   | 480p                     |
| PS Vita, Switch, Steam| 720p                     |


## Why Build It?
This idea first came when I noticed how better my RetroTink was when fed with original resolution from my physical consoles vs upscaled signals from PC or Batocera systems. This guide focuses on a digital HDMI workflow (PC → HDMI → scaler → display), not analog CRT output.

Existing emulation solutions often fall short of this unified vision:
- MiSTer/FPGA: Excellent for cycle-accurate 240p retro systems (up to PS1) but lacks support for more modern to this day. 
- Batocera/RetroPie: User-friendly but typically upscale retro games, losing native resolution fidelity, and struggles with flexible HDMI output. 
- CRT EmuDriver: Powerful for 240p on older GPUs but outdated, unsigned, and incompatible with modern APIs like Vulkan or Steam. 

## Challenges & Constraints
Building a unified, gamepad-only emulation system came with a few key challenges. I spent a full month configuring the setup and invested in several hardware components I didn’t already own. Most notably, Windows lacks native support for consistent controller order, so we'll use a custom Bluetooth toggle workaround. Achieving true native resolutions also meant manually defining custom timings with CRU, validating which resolutions the scaler accepts reliably, and ensuring emulator settings don’t silently override the intended output resolution. 

All this in order to preserve resolution accuracy, controller consistency, and the seamless couch-only experience.

## Requirements and Software
### Hardware Requirements
The system requires a PC with sufficient power for emulation, a compatible GPU/driver, an HDMI scaler, and a monitor. My tested setup is provided as an example.

- GPU: Must support EDID overrides/custom resolutions (CRU) and the required timings/pixel clocks.
  - Notes: Compatibility can vary by driver features (e.g., DSC on NVIDIA) and multi-display setups.
  - Tested: Radeon HD 7750, RX 580 Nitro+, RX 6700 XT (eGPU via OCuLink).
  
- Scaler/Display: An HDMI scaler accepting legacy resolution inputs, upscaling to your monitor capacity (1080p or higher) with CRT shader support.
  - Tested: RetroTink 4K scaler outputting to a 27” LG OLED ULTRAGEAR monitor on a VESA stand for TATE mode.
  
- Controllers: Gamepads (wired or wireless).
  - Tested: Sony DualSense x2, Hori Octa Pro, Xbox Series X controller (four players).



  ![OCulink RX580](/images/OCulink_RX580.jpeg)


### Software Requirements 
Some paid and free components are used. Essential software is required for core functionality, while optional tools simplify setup or enhance performance.  

#### Essential Software

| Software | Type | Purpose |
| --- | --- | --- |
| Windows 11 Pro | Paid | OS |
| [LaunchBox/BigBox](https://www.launchbox-app.com/) | Paid | Gamepad-driven frontend for unified emulator and Steam integration. |
| [reWASD](https://www.rewasd.com/) | Paid | Standardizes controller inputs across all systems and emulators. |
| [Custom Resolution Utility (CRU)](https://customresolutionutility.net/) | Free | Adds custom HDMI resolutions/timings (EDID override) for low/legacy modes. |
| [RetroArch](https://www.retroarch.com/?page=platforms) | Free | Multi-system frontend/emulator (Libretro cores) for many classic platforms. |
| Standalone emulators | Free | For systems where standalone emulators are preferred or no Libretro core is available. |
| [Steam](https://store.steampowered.com/) | In-Pay | PC game client (or similar). |
| [AutoHotkey 1.1](https://www.autohotkey.com/) | Free | Automation scripts for unified hotkeys and small UI workarounds. |
| [Res-O-Matic](https://m.majorgeeks.com/files/details/res_o_matic.html) | Free | Forces a specific Windows resolution when launching an emulator/game. |
| [Visual C++](https://aka.ms/vs/17/release/vc_redist.x64.exe), [.NET Framework](https://dotnet.microsoft.com/en-us/download/dotnet-framework/net481), [DirectX](https://www.microsoft.com/en-us/download/details.aspx?id=35) | Free | Runtime libraries for emulator compatibility. |

#### Optional Software 
- [iRotate](https://www.entechtaiwan.com/util/irotate.shtm): Display rotation utility, Required for TATE / stacked handheld layouts. 
- [7-Zip](https://www.7-zip.org/): Manages emulator and ROM archives. 
– [Rufus](https://rufus.ie/en/#download/): Writes bootable Windows install. 
- Proprietary controller drivers: Possibly required for some Windows gamepads (e.g., Hori Octa Pro). 
- [WinSCP](https://winscp.net/eng/index.php): Simplifies ROM transfers over a network (requires SSH setup on the PC). 
- Debloating tools ([O&O ShutUp10](https://www.oo-software.com/en/shutup10), [Winaero Tweaker](https://winaerotweaker.com/)): Reduce telemetry/popups and improve couch-only usability. 
- [Notepad++](https://notepad-plus-plus.org/downloads/): Handy for scripts and config editing. 
- [Bulk Crap Uninstaller]( https://www.bcuninstaller.com/): Deinstallation /startup app management. 

#### Troubleshooting Tools
 - [Microsoft Show/Hide Updates](https://download.microsoft.com/download/f/2/2/f22d5fdb-59cd-4275-8c95-1be17bf70b21/wushowhide.diagcab): Hide a problematic Windows/driver update if something ever breaks.
 - [DDU](https://www.wagnardsoft.com/display-driver-uninstaller-ddu): Deep-clean GPU driver removal when normal uninstall/reinstall fails.
 - [AMD Cleanup Utility](https://www.amd.com/en/resources/support-articles/faqs/GPU-601.html): Official AMD tool to remove AMD graphics drivers/software.
 - [Vulkan SDK](https://vulkan.lunarg.com/sdk/home#windows): Only if Vulkan runtime components are missing on your system (unlikely on modern driver installs).

#### Additional Recommendations
- Secondary PC: Useful for setup tasks like ROM transfers or scripting, but not required.
- Network Setup: Enabling SSH on the emulation PC (e.g., via OpenSSH Server) allows file transfers without USB drives. USB or NAS work too but SSH is best with a Gamepad only machine.
- Clean Windows install: This guide starfts from a clean Windows 11 install (fresh OS, then install your preferred GPU driver and core tools before emulator configuration).


## Video Signal Chain


```Video signal
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
(1440p 60Hz*, 4:3 /16:9 or TATE)
```
The system outputs native resolutions via HDMI, upscaled by a scaler to a modern display. Above is the signal flow for my setup, adaptable to other GPUs, scalers, or monitors.
-	Note: On my setup, I find that 60 Hz produces cleaner motion than 120 Hz or higher on 2D/ pixel retro systems.



In my configuration:
- The mini PC (Ryzen 9 8945HS) and RX 6700 XT eGPU generate native resolutions.
- The RetroTink 4K scaler upscales to 1440p, applying CRT shaders for retro authenticity.
- The LG OLED displays at 1440p, limited by its resolution, but a 4K monitor could replace it for higher output.

## Setup Process
Setting up the emulation station involves preparing the PC, configuring the GPU and drivers, optimizing Windows, and enabling file transfers.

### Preparing the PC
1. Gather Files: Collect all required software (emulators, drivers, utilities) on a network drive or USB stick for easy access.
2. BIOS Settings: Enter the BIOS and set the CPU to performance mode to maximize emulation power. Disable unnecessary features like secure boot if they interfere with drivers.
3. Windows Installation:
   - It is recommended to prepare the Windows installation on a bootable drive with Rufus and then enable the “Windows User Experience” options that creates a local account to skip privacy questions / disable data collection prompts.
   - Install Windows offline first (without setting internet yet) to keep setup simple and allow a clean local-account.
   - Skip setting a password to enable auto-login, critical for gamepad-only operation (see [SSH section below](#ssh-for-file-transfers)).



### GPU and Driver Setup 
A compatible GPU is essential for outputting low/legacy resolutions over HDMI and for running modern emulators (Vulkan).
GameMaze uses Custom Resolution Utility (CRU), which defines them by creating EDID overrides directly in the Windows registry.

**Known-working (tested by me):**
 - AMD Radeon HD 7750
 - AMD RX 580
 - AMD RX 6700 XT
 (all eGPU via OCuLink in my setup) 

**Important notes about compatibility:**
 - Results depend on the full chain (GPU + driver version + HDMI/DP output path + the scaler/EDID + the timing you’re trying to add).
 - If CRU settings don’t apply, start by running CRU’s restart64.exe (in the CRU folder) to restart the graphics driver or reboot.
 - If it still doesn’t apply, try a different driver version (CRU relies on driver support for EDID overrides).



1. Install Drivers:
   - Download and install your video drivers (latest recommended).
   - Optional (only if you are switching GPUs or troubleshooting broken installs): use AMD Cleanup Utility (AMD) or DDU for a clean uninstall before reinstalling drivers. 

2. Driver updates:
 - With GameMaze setup, you generally do not need to freeze GPU drivers or block driver updates.
 - Troubleshooting only: if Windows Update keeps replacing your GPU driver and it breaks something, use Microsoft “Show/Hide Updates” to block that specific driver update.


### AMD GPU + HDMI Scaler: HDCP fix (if image looks wrong) 
If your scaler output looks blurred/pixelated, disable HDCP in AMD Software: Adrenalin Edition.
![AMD HDCP issue](/images/DRM.jpg)

Settings (the gear, top right)> Display > Overrides (expand) > HDCP Support (Disable and accept the warning)


![AMD HDCP Fix](/images/amdhdcp.png)

### Windows Configuration
1. Activate Windows

2. Power Settings:
 - Win+I → System → Power & battery → Power mode → Best performance.
 - Disable screensaver: Settings → Personalization → Lock screen → Screen saver → None.

3. Disable Fast Startup:
Fast boot is normally enabled in Windows 11 by default but in short, it can cause issues with emulator saves when shutting down.
 - Open Control Panel → Hardware and Sound → Power Options.
 - Click Choose what the power buttons do.
 - Click Change settings that are currently unavailable.
 - Under Shutdown settings, uncheck Turn on fast startup (recommended) → Save changes.


4. Remove Bloat: In Settings → Apps and Optional Features, uninstall unnecessary apps

5. Game Bar Settings → Untick Allow your controller to open Game Bar.

6. Install Runtimes:
   - Install Visual C++, .NET Framework, DirectX, and Vulkan SDK.

7. Update Windows:
   - Run Windows Update, install all updates, and reboot until no updates remain.

8. Set Your desired Desktop Resolution (I am using 1280 x 720)


## SSH for File Transfers
A core goal of this setup is to boot directly into BigBox with no keyboard or mouse interaction. 
SSH/SFTP lets you transfer files from another PC (e.g., WinSCP) without touching the emulation system’s UI. 

### Why key-based SSH?
- Preserves gamepad-only operation: You can keep windows-login password-free while still transferring files remotely. 
- Safer than blank passwords: You authenticate using an SSH keypair (private key on your main PC, public key on the emulation PC).

### SSH Setup Steps
0. Note the emulation PC’s IP address:
   - Open Terminal/CMD and run: `ipconfig`
   - Write down the IPv4 Address of the active adapter for later.

1. On the emulation PC → Enable OpenSSH Server:
   - Open PowerShell as admin and run:
     ```powershell
     Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
     ```
   - Verify installation:
     ```powershell
     Get-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
     ```

2. Configure SSH Service:
   - Run `services.msc`, find **OpenSSH SSH Server (sshd)**, set Startup Type to Automatic, and click Start. Reboot.

3. On your main PC → Generate a key with WinSCP:
   - Open WinSCP → create/edit your session → click Advanced…. 
   - Go to SSH → Authentication. 
   - Click Tools → Generate New Key Pair with PuTTYgen. 
   - In PuTTYgen, choose EdDSA (Ed25519), generate the key, and save the private key (a .ppk file) somewhere safe. 
   - Back in WinSCP (same page), click Display Public Key and copy the public key text (OpenSSH authorized_keys format). 

4. On the emulation PC → Install the public key (Admin):
   - Create a blank file named "administrators_authorized_keys" in: `C:\ProgramData\ssh\administrators_authorized_keys`.
   - Paste the public key as one single line into the file (one key per line), then save.

5. Fix permissions:
   - Open PowerShell as Administrator and run: 
     ```powershell
     icacls.exe "C:\ProgramData\ssh\administrators_authorized_keys" /inheritance:r /grant "Administrators:F" /grant "SYSTEM:F"
     ```

6. Configure SSH to use keys:
   - Edit `C:\ProgramData\ssh\sshd_config` and set: 
     ```text
     PubkeyAuthentication yes
     PasswordAuthentication no
     ```
   - Restart the OpenSSH SSH Server (sshd) service in `services.msc` or reboot the PC.

7. On your main PC → Set up WinSCP connection:
   - File protocol: SFTP, Hostname: emulation PC IP, Port: 22, Username: your emulation PC’s Windows admin user. 
   - Advanced → SSH → Authentication → set Private key file to the .ppk you saved. 

### Alternative
Use USB drives for file transfers, only recommended if SSH is not feasible. 

### SSH Notes
- This does not require the emulation PC to show a password prompt on screen; it only changes how WinSCP authenticates over the network. 



## Resolution and Emulator Configuration
This section covers setting up native low/legacy resolutions (160p–384p) using Custom Resolution Utility (CRU) and configuring emulators to output them correctly, targeting pixel-perfect visuals.

### Resolution Setup with CRU
The CRU resolutions below are the *custom* low modes used for retro systems. Standard HDMI modes like 480p and 720p do not require CRU and can be selected normally per emulator/game as needed.

#### Resolution Table (CRU)
| Resolution | Active (H×V) | Front Porch (H/V) | Sync Width (H/V) | Blanking (H/V) | Polarity (H/V) | Refresh Rate (Hz) |
|:----------:|:------------:|:-----------------:|:----------------:|:--------------:|:--------------:|:-----------------:|
| 160p       | 2560×160     | 16/3              | 32/4             | 80/12          | -/-            | (set in CRU)      |
| 224p       | 2560×224     | 16/3              | 32/4             | 80/14          | -/-            | 59.985            |
| 240p       | 2560×240     | 16/3              | 32/4             | 80/15          | -/-            | 60.011            |
| 256p       | 2560×256     | 16/3              | 32/4             | 80/16          | -/-            | 59.840            |
| 272p       | 2560×272     | 16/3              | 32/4             | 80/17          | -/-            | 60.003            |
| 384p       | 2560×384     | 16/5              | 32/6             | 80/24          | -/-            | 60.002            |

#### Resolution Notes
- **CRU slot budget:** CRU slots are limited, so this list prioritizes the modes I actually use (160p/224p/240p/256p/272p/384p). If you want pixel-perfect 144p for Game Boy, you may need to sacrifice another mode; otherwise RetroArch CRT SwitchRes will pick the closest available one.

- **Refresh-rate variations (optional):** Slight differences (e.g., 59.985 vs. 60.000 Hz) *may* help RetroArch CRT SwitchRes consistently select the intended resolution when multiple modes are close (e.g., 224p vs. 240p). If you prefer simplicity, you can try 60.000 Hz across the board; if mode selection becomes unreliable, revert to the tested values.

- **480p/720p are not CRU topics:** 480p and 720p are standard HDMI modes. Use them normally for systems/emulators that are already 480p+ (e.g., PS2/Dreamcast/3DS stacked layouts) without spending CRU slots.

- **Horizontal timings:** All resolutions use the same horizontal settings (Active: 2560, Front Porch: 16, Sync Width: 32, Blanking: 80, Polarity: -).



![240p Resolution](/images/240.PNG)

### CRU Setup Steps
1. Open CRU: Launch CRU and ensure your scaler/display is selected (top left, marked “active”). If not, choose it from the dropdown.

2. Clear Defaults: In Detailed Resolutions (top right), delete existing resolutions to make space for yours.

3. Add Resolutions (my 6-slot layout):
   - Click Add → Manual timing, and enter settings from the table above (e.g., 2560×224 for 224p).
   - In **Detailed Resolutions** (up to three slots): add **160p, 224p, 240p**.
   - In **Extension Blocks → CTA-861 → Edit**: add **256p, 272p, 384p**.

4. Save and Restart:
   - Click OK twice to save changes.
   - Run `restart64.exe` (in the CRU folder) to apply the new resolutions.

5. To Verify:
   - You can test each resolution in RetroArch when CRT SwitchRes is active (Settings → Video → CRT SwitchRes → 15 kHz / Super resolution 2560).



### Emulator Configuration
The goal is to preserve each system’s original signal and let the scaler handle all upscaling and CRT-style processing. Emulators should output native resolutions wherever possible, with no internal filtering or smoothing.
Use nearest-neighbor scaling and x1 resolution for anything up to the PS1 era to maintain pixel accuracy. For PS2 and newer systems, internal resolution scaling (x2 or x3) can be used selectively when it looks better.
RetroArch is used for most retro systems, while standalone emulators are preferred for newer platforms. When RetroArch’s CRT SwitchRes isn't suitable (e.g., for vertically stacked DS or 480p+ content), Res-O-Matic is used to launch games at the desired resolution.

#### RetroArch Configuration
1. General Settings:
   - Go to Settings → User Interface → Menu, select rgui.
     This might not be the sexiest skin, but it works best with low resolutions.
   - For each core, load a game, open the menu, and set:
     - Video → Scaling → Aspect Ratio: Core Provided (or Full for vertical games if stretched).
     - Save core overrides: Quick Menu → Overrides → Save Core Overrides.
     - However if a game is strectched with ratio Core Provided and needs Full to look fine, save it as Game Override instead of Core Overrride.

2. Note on Ratio:
   “Core Provided” usually works, but if a game looks stretched—especially vertical ones—switch to “Full.” This is safe because the RetroTink 4K enforces 4:3 output, so nothing gets stretched to 16:9. It also smartly adapts when 16:9 content (like Vita or desktop) is detected.

3. PS1 (Mednafen PSX Core):
   - Menu → Settings → Video → CRT SwitchRes: Enable, set to 15kHz, 2560 horizontal.
   - Aspect Ratio: Core Provided.
   - Save core overrides.

![PS1 Example](/images/crono.jpg)
*Chrono Trigger PS1* 

4. Arcade (FBNeo Core):
   - Same as PS1, plus Quick Menu → Core Options → Vertical Mode: TATE or TATE Alternate (Will only affect vertical games).
   - Aspect Ratio: Core Provided (or Full if stretched).
   - Save core overrides. If a game needs aspect ratio Full, save as game override.

5. Nintendo DS (Desmume Core):
   - CRT SwitchRes: Disable (desired 2560×256 is not properly detected by SwitchRes).
   - Aspect Ratio: Full.
   - Core Options → Screen Layout: Bottom/Top (or Top/Bottom per preference).
   - Save core overrides.
   - Use Res-O-Matic in LaunchBox (see below) to force 2560×256.

6. GameCube (Dolphin Core), Dreamcast (Flycast Core):
   - CRT SwitchRes: Disable (RetroArch defaults to desktop resolution).
   - Aspect Ratio: Core Provided (or whatever works best is not good).
   - Use Res-O-Matic to force 640×480 (480p) in LaunchBox.

![FF5GBA](/images/FF5.jpeg)
*Final Fantasy 5 on GBA at 160p*

#### Standalone Emulator Configuration
Standalone emulators handle modern systems (PS2, Switch, etc.) and some retro systems. Set them to native resolutions or 720p, depending on your scaler’s performance.
1. MAME 0.272 (per core settings, eg Namco, ST-V, SNK Hyper 64):
   - Ex. for ST-V (224p):
     - Run a game in Mame and close it.
     - Create /MAME/ini/stvbios.ini, containing:
       ```
       [stvbios]
       resolution        2560x224@60
       aspect            4:3
       rotate            0
       unevenstretch     1
       switchres         1
       ```
     - For vertical games (e.g., Shienryu), create /MAME/shienryu.ini, containing:
       ```
       rotate            1
       ```
     - Launch the game, if needed, adjust rotation, save Game settings (Not System settings).
   - Namco/SNK: repeat the ST-V process with resolution 640x480@60 if desired.

2. Supermodel (Sega Model 3):
   - Get the Supermodel.ini (likely for Xinput) and NVRAM files from the great [thread of Warped Polygon](https://forums.launchbox-app.com/files/file/3857-sega-model-3-supermodel-git-everything-pre-configured-inc-controls-for-pc-controller-mouse-light-guns-test-menus-configured-free-play-all-games-in-english-2-player-mouse-support-audio-adjusted-layout-imagesthe-whole-9-yards)
   - Follow additional setup from Warped Polygon’s guide (But no GUI needed to preserve 384p).
   - Edit /supermodel/Config/Supermodel.ini:
     ```
     FullScreen=1
     Throttle=1
     XResolution=2560
     YResolution=384
     Stretch=True
     ```

3. Citra (3DS):
   - Set View → Screen → Rotate Up Right for vertical stacked screens.
   - Default runs at desktop resolution so use ResOMatic in LB for customs resolution if preferred.

4. PCSX2 (PS2), RPCS3 (PS3), Ryujinx (Switch), Vita3K (PS Vita):
   - **RPCS3:** On Windows 11 with current GPU drivers, the latest RPCS3 builds should normally work fine. 
     - Prefer Vulkan when available (it’s usually the best-performing backend, unless per game settings say otherwise). 

   - **PCSX2:** Use the latest builds unless you have a specific regression.
     - Renderer: Vulkan is supported if your GPU/driver meets PCSX2’s Vulkan requirements; otherwise use D3D11/D3D12 (or OpenGL) as needed. 
     - Scaling: For PS2, use internal resolution x2/x3 selectively when it looks better; keep nearest-neighbor / minimal filtering to preserve a faithful low-res look.

   - **Output resolution:** Set each emulator to output the *native* mode you want (e.g., 480p content at 480p), and only use higher output modes when you have a specific reason (display/scaler preference, performance, or compatibilit)y.
Note: For resolutions different from your desktop resolution, you need to use Res-O-Matic.

   - **Aspect ratios:** PCSX2 can respect 4:3 for 4:3 titles; PS3/Switch/Vita titles are typically 16:9.



5. PPSSPP (PSP):
   - Set to 272p (native) or desktop based on preference.
  
 - Note: If for any reasons you need extra tools to rotate your display for a specific emulator, use iRotate in a LB script:

![irotate](/images/rotate.png)
```
; Rotate to portrait when emulator starts
Send ^!{Left}   ; Ctrl + Alt + Right for portrait rotation
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
  *This script is based on ESC key being the emulator shut down key, edit with your own key if different.*

### Steam
Upon installing Steam and connecting to your account, go to:
- Steam/ Settings/ Controller => Guide Button Focuses Steam = OFF
- And Enable Guide Button Chord for Controller = OFF
- Also, it is up to you, but if you want the games to install on the emulation PC locally and be available without network, better go to Steam/ Settings/ Remote Play/ and turn it OFF. (Otherwise, if your main PC is connected, games might not install locally and rather use the install from the other computer and trigger the mirror screen on both machines).



## REWASD and LaunchBox Setup
This section configures REWASD to standardize gamepad inputs across all emulators and LaunchBox/BigBox to unify emulators, Steam, and scripts for a seamless, gamepad-only experience.
My setup uses a mix of two Sony DualSense, Hori Octa Pro, and an Xbox Series X controller, but any wired or wireless gamepads work.

### REWASD Setup
Connect all your controllers and start REWASD.

- Profile Creation:
  - Delete default profiles if you don’t need them and create a new one (e.g., “BigBox”).
  - Select a gamepad from the bottom icons (e.g., DualSense).
- Set Output to Virtual Xbox:
  - Click the Xbox icon (top center, “Output Devices Settings”).
  - Enable Virtual Xbox 360 (or XInput /SDL of your choice) on the left and save.

![REWASD Output Settings](/images/rewasdOUT.png)

- Configure Hotkeys:
  - Click the Home button (e.g., PS or Xbox button) on the gamepad schematic.
  - Select Shift Mode → Jump to Layer 1.
  - Switch to Shift Layer 1 (top center) and map:
    - South (X on PlayStation, A on Xbox) = TAB (opens emulator menus).
    - Start = ESC (quits emulators).
    - Or your preferred keys.
  - Save the profile.
- Apply to All Gamepads:
  - Select each gamepad and click Apply to Slot 1, Slot 2, etc.
  - Ensure REWASD starts with Windows (Settings (top right, gear icon) → General → Start with Windows).
  - Additionally, while in REWASD settings, you can tweak → Overlay → and there deactivate unwanted notifications.
- Test in Emulators:
  - Open RetroArch and every standalone emulator to verify controls work. If needed, rebind in emulator settings, selecting Xinput or SDL as the input type.

### Proposed Controller Mapping
To ensure consistent controls across all emulators and BigBox:

| Button Combo   | Action | Purpose                              |
|----------------|--------|--------------------------------------|
| Home + South   | TAB    | Opens emulator configuration menus.  |
| Home + Start   | ESC    | Quits the emulator or game.          |
| Standard Buttons | Native | Mapped to in-game controls (configured per emulator). |

Note: You can customize these combos in REWASD (e.g., Home + Select = TAB).
Ensure all emulators are set to respond to TAB (menu) and ESC (quit), or use LaunchBox scripts for those that don’t support remapping.

### LaunchBox Configuration
I am not doing a Launchbox tutorial here and will only focus on the steps useful for unified quitting, custom resolutions, and popup dismissal.

- Install LaunchBox/BigBox:
  - Download and install LaunchBox with BigBox for gamepad-only navigation.
  - Import ROMs and configure emulators (cf. LaunchBox documentation).
- Unified Quit Control:
  - For emulators supporting custom quit keys (e.g., PPSSPP):
    - Set ESC as the quit key in emulator settings.
    - Disable “Confirm Quit” to ensure instant closure.
  - For emulators without custom quit keys (e.g., RPCS3):
    - In LaunchBox, go to Tools → Manage → Emulators → RPCS3 → Edit → Running Script.
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
    - Repeat for other emulators (replace rpcs3.exe with the emulator’s executable).

![LaunchBox Quit Script](/images/close.png)

## Using Res-O-Matic for Custom Resolutions
Res-O-Matic forces a specific launch resolution outside of RetroArch’s CRT SwitchRes workflow.
This is useful when CRT SwitchRes picks an unexpected mode, or when you need to launch a non-desktop resolution (e.g., 480p) with standalone emulators.

We will create `.bat` launchers in the emulators’ folders and configure these alternative “emulators” in LaunchBox.

- Example: Nintendo DS (2560×256):
  - Create C:/RetroArch/launch_retroarch_ds_256x2560.bat, containing:
    ```batch
    @echo off
    echo start /wait "" "C:\Emulators\RetroArch\retroarch.exe" -L "cores\desmume_libretro.dll" %* > "%TEMP%\launch_ds.bat"
    start /wait "" "C:\YourPath\reso.exe" "%TEMP%\launch_ds.bat" 2560 256 32 60
    del "%TEMP%\launch_ds.bat"
    ```
  - In LaunchBox, add as a new emulator (e.g., “RetroArch DS 256p”), target Application path = launch_retroarch_ds_256x2560.bat and associate with Nintendo DS.

![Nintendo DS Emulator](/images/csEmul.png)

- Example: GameCube (640×480):
  - Create /RetroArch/launch_retroarch_dolphin_640x480.bat, containing:
    ```batch
    @echo off
    echo start /wait "" "C:\Emulators\RetroArch\retroarch.exe" -L "cores\dolphin_libretro.dll" %* > "%TEMP%\launch_dl.bat"
    start /wait "" "C:\YourPath\reso.exe" "%TEMP%\launch_dl.bat" 640 480 32 60
    del "%TEMP%\launch_dl.bat"
    ```
  - Add as a new emulator in LaunchBox for GameCube.

Tip: Adjust paths and core names to match your setup. Create similar scripts for other systems needing custom resolutions.
- Note: check the core spelling going to /RetroArch/Cores/ directory


## Dismissing Popups (Optional)
Most setups won’t need this if drivers and games are configured cleanly, but it’s useful to have a fallback when you’re running a gamepad-only box and a dialog steals focus.

### Try LaunchBox options first
In LaunchBox, go to **Tools → Manage Emulators → (select emulator) → 
 - Details > You can tick Attempt to hide console window on startup/shutdown
You also have: 
 - Startup Screen**:
- **Aggressive Startup Window Hiding** (helps hide/handle transient launcher windows).
- **Hide All Windows that are not in Exclusive Fullscreen Mode** (can help keep Big Box in focus).

These options can help for some cases, but modal Windows dialogs may still require automation. 

### Example: closing a stubborn warning (AutoHotkey)
This example targets a specific popup I used to get (it should not happen on a properly updated system), where Forza Horizon was showing a warning of obsolete drivers where I had to click OK to dismiss before the game would launch.  
Use Window Spy to identify the popup’s window title/class and adjust the script for other games. (AutoHotkey supports targeting windows by `ahk_class`, as shown by Window Spy.) 

- Create `ROMs/Steam/LaunchForza.bat`:
  ```bat
  @echo off
  start "" "ForzaPopupKiller.ahk"
  ```

Create ForzaPopupKiller.ahk in the same folder:
#SingleInstance Force
SetTitleMatchMode, 2
Run, steam://rungameid/1551360
Sleep, 10000
timeout := 90000
elapsed := 0
interval := 500
Loop
{
    IfWinExist, Forza Horizon ahk_class #32770
    {
        WinActivate
        Sleep, 300
        SendInput {Alt down}{F4 down}
        Sleep, 50
        SendInput {F4 up}{Alt up}
        Sleep, 1000
        IfWinExist, Forza Horizon ahk_class #32770
        {
            WinGetPos, X, Y, W, H, Forza Horizon ahk_class #32770
            XButtonX := X + W - 20
            XButtonY := Y + 20
            Click, %XButtonX%, %XButtonY%
            Sleep, 500
            IfWinExist, Forza Horizon ahk_class #32770
            {
                WinKill
            }
        }
        break
    }
    Sleep, %interval%
    elapsed += interval
    if (elapsed >= timeout)
        break
}

In LaunchBox, set the game’s Application Path to LaunchForza.bat.



### LaunchBox Notes
- BigBox Shell Mode: Once configured, set BigBox to launch at startup. You even have the possibility to run it as shell (replacing Windows Explorer) for a true console-like experience. Don’t make BB a shell until you’re done setting up everything on your PC though.
- Script Paths: Adjust file paths in scripts to match your directory structure. My example was targeting a Window named Forza Horizon 5. When targeting a different popup, adjust the key words.
- Popup Workarounds: Test popup scripts with WindowSpy.ahk if the window class differs.

# Controller Reordering Workaround
- Windows lacks native controller reordering, so we will use a Bluetooth toggle script to reorder gamepads without keyboard/mouse.
- Create C:/Emulators/Bluetooth/ToggleBluetooth.bat, containing:
  ```batch
  @echo off
  net session >nul 2>&1
  if %errorlevel% neq 0 (
      powershell -Command "Start-Process cmd -ArgumentList '/c %~f0' -Verb RunAs" >nul 2>&1
      exit
  )
  start /min powershell -NoProfile -ExecutionPolicy Bypass -Command "$ProgressPreference = 'SilentlyContinue'; $bt = Get-PnpDevice -FriendlyName '*Bluetooth*' | Where-Object {$_.HardwareID -like '*BTH\MS_BTHBRB*'}; $bt | Disable-PnpDevice -Confirm:$false; Start-Sleep -Milliseconds 500; $bt | Enable-PnpDevice -Confirm:$false; Start-Sleep -Milliseconds 500" >nul 2>&1
  exit
  ```
  
- Create an empty file as: C:/Emulators/Bluetooth/BT.bin
- In LaunchBox:
  - Create a new platform (eg. BT-Toggle).
  - Add a new emulator: Tools → Manage → Emulators → Add.
  - Name: “Gamepad Reorder”.
  - Application Path: ToggleBluetooth.bat.
  - Platform: “BT-Toggle”.
  - Add a ROM: Platform “BT-Toggle”, file BT.bin.
- In BigBox, launch the “BT-Toggle” fake rom to toggle Bluetooth, then reconnect controllers in desired order (Player 1 first, Player 2 second, etc.).
- Enhance visuals: In LaunchBox, filter by platform, find BT-Toggle and right click on it: edit metadata, and add images (e.g., Bluetooth logo, gamepads) as banner / clear logo override / device / fanart. Whatever works best with your BB theme.

![LaunchBox Platforms](/images/platforms.png)

![BT-Toggle Platform](/images/BTplatform.png)

![LaunchBox Interface](/images/LB.png)

## Final Touches
With emulators, resolutions, and gamepad controls configured, the final step is to optimize Windows for maximum performance and a distraction-free, gamepad-only experience as close as possible to a gaming console.

### Debloating Windows
The goal is to reduce background noise (telemetry, notifications, bundled apps) and eliminate UI interruptions that break a gamepad-only flow.

#### Option A, Manual:
Tools like O&O ShutUp10 and Winaero Tweaker simplify this process.
- Install Debloating Tools:
  - We will use O&O ShutUp10 and Winaero Tweaker, run as administrator.
- O&O ShutUp10 Configuration:
  - Launch O&O ShutUp10 and select “Recommended Settings” for a balanced approach, or customize:
    - Disable telemetry (under Privacy → Telemetry) to stop background data exchanges with Microsoft.
    - Turn off Windows Update notifications to prevent driver update prompts.
    - Disable Cortana and other background apps to reduce CPU usage.
  - Apply changes and reboot.
- Winaero Tweaker Configuration:
  - Open Winaero Tweaker and navigate to key settings:
    - Disable lockscreen (use the search bar if needed) as we can’t return in BigBox from Lockscreen with only a game controller.
    - Disable Action Center: Eliminates popups during gameplay.
    - Disable SmartScreen: Speeds up app launches (safe for a dedicated emulation PC).
  - Apply changes and reboot.

#### Option B, Automated:
Chris Titus Tech WinUtil (all-in-one) 
1.	Open PowerShell **as Administrator** and run: 
```powershell irm "https://christitus.com/win" | iex```

2.	Use it to apply your preferred debloat/tweak preset and install common apps (WinGet) as needed.


### Additional Optimizations
These tweaks polish the system, ensuring it boots directly into BigBox and runs efficiently.
- Disable Startup Apps:
  - Open Task Manager (Ctrl+Shift+Esc) → Startup tab.
  - Disable all non-essential apps (e.g., OneDrive, browser updaters), keeping REWASD and LaunchBox if set to auto-start.
-Optionally, Set BigBox as Shell:
  - In BigBox, go to Options → General → Enable using as the Windows Shell. (replaces Explorer, no Dekstop, don’t do it if you often use explorer / web-browsers /keyboard or mouse etc.).
  - If you do not want to use BigBox as shell, you can hide the icons on desktop (Right click on the desktop, view, uncheck Show desktop icons)
- Optimize Storage:
  - Use Bulk Crap Uninstaller (BCU, optionally previously installed) to uninstall programs in bulk and clean leftovers.
- Startup apps: Settings → Apps → Startup, or Task Manager → Startup Apps, to disable anything non-essential. 
Optional: BCU also includes a Startup manager
  - Put ROMs and games to a fast SSD for quicker load times.
- Backup Configuration:
  - Back up LaunchBox settings (Tools → Backup) and REWASD profiles to a USB drive or network folder.
  - Save CRU settings and emulator configs for easy restoration.

### Optimization Notes
- Security Context: This is a dedicated emulation PC on a home network. Disabling telemetry is mainly a privacy/noise reduction choice, but disabling SmartScreen (reputation-based protection) reduces real protection against malicious or unknown downloads—only do it if you understand the trade-off.

## Conclusion
We’re done! This project was about more than just running games but rather unifying different generations of hardware and software into a consistent, console-like experience. It’s the result of many small choices and workarounds coming together, and I hope it might be useful to others facing similar challenges.







