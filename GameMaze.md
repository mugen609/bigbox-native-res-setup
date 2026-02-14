# GameMaze  
*Native Resolutions • Seamless Gamepad Control • Console-Like Experience for scalers on modern displays*

---

🎮 Imagine launching Street Fighter II at its original 224p arcade resolution, then switching to Elden Ring at 1080p—on the same box, from your couch, using only a gamepad.
This guide is a step-by-step recipe to make that real.

---

# Table of Contents
| Section | Description |
|---------|-------------|
| [Overview](#overview) | Vision, supported systems, why GameMaze |
| [Requirements](#requirements-and-software) | Hardware, GPU compatibility, software list |
| [AMD Blurred Screen](#AMD-GPU-HDMI-Scaler-HDCP-Fix) | Fix DRM on AMD + scaler |
| [SSH for file transfer](#SSH-Setup) | Allow file transfer + config file edit |
| [CRU Timings](#CRU-Timings) | Allow custom resolutions via HDMI |
| [Emulator Config](#emulator-configuration) | RetroArch cores, standalone (MAME/Supermodel) |
| [Steam](#steam) | Recommended Settings |
| [reWASD Setup](#rewasd-setup) | Gamepad Harmonization |
| [LaunchBox](#launchbox-configuration) | Global Unification |
| [Res-O-Matic](#res-o-matic-for-custom-resolutions) | Forcing Resolution |
| [Controller Reorder](#controller-reordering-workaround) | Change Player 1 Controller |
| [Final Touches](#final-touches) | Debloating, BigBox Shell, backups |

Annexes:
 - [Perfect Aspect Ratio Tutorial](PerfectAspectRatio.md)
 - [SSH Detailed Setup](SSH_NoPassword.md) 
 - [GameMaze Philosophy](GameMazeEssence.md)

---

*Dragon Quest 6 on SNES in native resolution*
![DQ6](/images/DQ6.jpg)  


## 👓
## Overview

GameMaze is a Windows configuration framework built around LaunchBox/BigBox that outputs low, console-like resolutions over HDMI/DisplayPort into a modern scaler (and uses standard 480p/720p+ modes for modern systems).

It optimizes for:
- Native-resolution output discipline into the scaler (CRU + resolution forcing; RetroArch CRT SwitchRes where applicable).
- A living-room, gamepad-only flow (boot → browse → play → quit).
- Modern compatibility (current drivers, Vulkan-era emulators, PS3/Switch/PC titles in the same box).

### 📺
> Digital-focus pipeline: HDMI/DP → scaler → modern display. This is not a CRT / analog-output (VGA/SCART) guide.

---

## Why GameMaze?

In many setups, a scaler looks best when it receives a native low-resolution signal, not a pre-upscaled desktop frame.
GameMaze’s approach is to output those low resolutions directly, while keeping the convenience and performance of a modern Windows emulation + PC gaming setup.

**📖** For deeper rationale and design details, also see: **[GameMaze Essence](GameMazeEssence.md)**.

---

## Requirements and Software

### Before You Start

 - **SSH** (Recommended for gamepad-only workflow): Enable on emulation PC for keyboardless file transfers. See [SSH Setup](SSH_NoPassword.md).
 - **Secondary PC** (optional): Helps with initial setup/ROMs.


## ⚡
### GPU Compatibility

GameMaze relies on CRU **super resolutions** (e.g., 2560×240) over HDMI/DP with current drivers 
 
 What this implies:
 
- No analog/VGA requirement (digital chain). 
- No “frozen driver” requirement (in my tests Feb 2026).
- A GPU/driver stack that accepts CRU EDID overrides + custom timings (see below)

---

*Super Mario Land running at native 144p via HDMI → RetroTink 4K with scanlines*
![GameBoy144p](/images/GBMario.jpeg) 

**Tested List (my hardware):**

#### ✅ Confirmed Working (Feb 2026 drivers, super resolution via CRU):
- AMD Radeon HD 7750 (CRT EmuDriver list, legacy VGA test to validate super-res + modern-driver)
- AMD RX 580 Nitro+ (CRT EmuDriver list, HDMI)
- AMD RX 6700 XT (**Not** on CRT EmuDriver list, HDMI)

#### ❌ Not Working In My Tests (Feb 2026 drivers, no legacy drivers testing):
- AMD 780M integrated GPU (Ryzen 9 8945HS)
- NVIDIA RTX 3090 (MSI Suprim)

Note: [CRT EmuDriver-listed GPUs](https://geedorah.com/eiusdemmodi/forum/viewtopic.php?id=295) should work with super resolutions and current drivers. Similar discrete AMD GPUs (e.g., RX 6600) are likely compatible. Integrated graphics and modern NVIDIA cards have shown issues in my limited tests, but treat it as empirical. 

## My Setup & Signal Chain (as example only)

[Mini PC: Ryzen 9 8945HS] ──[eGPU: OCuLink]── [RX 6700 XT] ──[HDMI: CRU super-res]── [RetroTINK 4K] ──[CRT shaders → 1440p]── [LG OLED: 1440p 60Hz, VESA TATE]


**Key components:**
- **GPU**: RX 6700 XT. See [GPU Compatibility](#gpu-compatibility).
- **Scaler**: HDMI scaler accepting 144p–544p inputs (RetroTINK 4K tested).
- **Controllers**: DualSense ×2, Hori Octa Pro, Xbox Series X (4-player).

*60Hz preferred for 2D retro motion clarity.*

---

## 🕹️
## Supported Systems and Resolutions 

| Category                    | Native Res |
|-----------------------------|------------|
| **Handhelds** (GB, DS)      | 144p–256p  |
| **Retro 8/16-bit + Arcade** | 224p–384p  |
| **Oddballs** (PSP, 3DS)     | 272p–544p  |
| **Gen 5** (PS1, N64)        | 240p       |
| **Gen 6** (DC, PS2)         | 480p+      |
| **Gen 7** (PS3, Wii, PC)    | 720p+      |

*All → HDMI → scaler (CRU super-res + RetroTINK 4K)*

---

![OCulink RX580](/images/OCulink_RX580.jpeg)

## 🪟
## Software Requirements

| Priority | Tool | Type | Purpose |
|----------|------|------|---------|
| **Required** | [Windows 11](https://www.microsoft.com/windows/windows-11) | Paid | Base OS |
| **Required** | [LaunchBox/BigBox](https://www.launchbox-app.com/) | Paid | Gamepad frontend |
| **Required** | [reWASD](https://www.rewasd.com/) | Paid | Controller standardization |
| **Required** | [CRU](https://customresolutionutility.net/) | Free | Custom HDMI resolutions |
| **Required** | [RetroArch](https://www.retroarch.com/) | Free | Legacy systems |
| **Required** | [AutoHotkey 1.1](https://www.autohotkey.com/) | Free | Quit scripts, popup fixes |
| **Required** | [Res-O-Matic](https://m.majorgeeks.com/files/details/res_o_matic.html) | Free | Force game resolutions |
| **Recommended** | [WinSCP](https://winscp.net/eng/index.php) | Free | SSH file transfers |
| **Recommended** | Standalone emulators (PCSX2, RPCS3, etc.) | Free | Modern systems |
| **Recommended** | [Steam](https://store.steampowered.com/) | In-Pay | PC games |
| **Optional** | [iRotate](https://www.entechtaiwan.com/util/irotate.shtm) | Free | TATE rotation |
| **Optional** | [7-Zip](https://www.7-zip.org/) | Free | ROM management |

**Troubleshooting (keep handy):**
- [DDU](https://www.wagnardsoft.com/): Deep GPU driver removal when normal uninstall fails  
- [AMD Cleanup](https://www.amd.com/en/resources/support-articles/faqs/GPU-601.html): Official AMD driver removal tool  
- [Show/Hide Updates](https://download.microsoft.com/download/f/2/2/f22d5fdb-59cd-4275-8c95-1be17bf70b21/wushowhide.diagcab): Block specific Windows/driver updates causing regressions  
- [Vulkan SDK](https://vulkan.lunarg.com/sdk/home): Runtime components (rarely needed with modern drivers)

---

## 📚
## System Preparation 

**Quick checklist** (before CRU setup):
- Update Windows + install GPU drivers (latest) + required softwares /emulators
- BIOS: CPU performance mode  
- **Fast Startup**: OFF (preserves emulator save files): 
Control Panel → Hardware and Sound → Power Options → Choose what the power buttons do → Change settings that are currently unavailable → Shutdown settings → uncheck Turn on fast startup (recommended) → Save changes
- System → Power & battery → Power mode → **Best performance**
- **Game Bar**: Disable controller shortcuts (windows Settings → Gaming)
- **Desktop**: 1280×720 *or higher* (Choose based on your scaler/monitor. Example: 720p → clean x2.0 RetroTINK scaling on 1440p.)

## 🖼️
### AMD GPU HDMI Scaler HDCP Fix

**Symptom**: Scaler image blurred/pixelated (*DRM blocking content*).

![Before: DRM blocking](/images/DRM.jpg)

**Fix**: AMD Software → **Settings** (⚙️) → **Display** → **Overrides** → **HDCP: Disabled**
![After: Fixed](/images/amdhdcp.png)

---

## 🌐
## SSH Setup 
*(Gamepad-Only Essential)*

 If your emulation PC is separate from your main machine, **SSH setup will save you hours of hassle** during configuration.

 With SSH, you can:
 - Edit config files from your main PC's comfortable setup
 - Transfer ROMs, installers, and backups seamlessly
 - Move LaunchBox saves and configs between machines
 - Manage everything without hunching over the gaming PC

I am deporting this section as it is a little long. **→ [SSH Remote Access Guide](SSH_NoPassword.md)**

This is **highly recommended if you have two PCs**, but perfectly optional if working locally.

> Better set this up **now**, (before CRU and emulator configuration) so you can do all subsequent work remotely.

---

## 🖥️
## CRU Setup (Custom Resolutions)

**Goal**: Add 144p–544p timings for pixel-perfect scaler input. Standard 480p+ HDMI modes need **NO** CRU.

#### CRU Timings

| Resolution | Active (H×V) | Front Porch (H/V) | Sync Width (H/V) | Blanking (H/V) | Polarity | Refresh Rate |
|:----------:|:------------:|:-----------------:|:----------------:|:--------------:|:--------:|:------------:|
| 144p       | 2560×144     | 16/3              | 32/4             | 80/9           | -/-      | 60.054       |
| 160p       | 2560×160     | 16/3              | 32/4             | 80/12          | -/-      | 60.011       |
| 224p       | 2560×224     | 16/3              | 32/4             | 80/14          | -/-      | 59.985       |
| 240p       | 2560×240     | 16/3              | 32/4             | 80/15          | -/-      | 60.011       |
| 256p       | 2560×256     | 16/3              | 32/4             | 80/16          | -/-      | 59.840       |
| 272p       | 2560×272     | 16/3              | 32/4             | 80/17          | -/-      | 60.003       |
| 384p       | 2560×384     | 16/3              | 32/4             | 80/24          | -/-      | 60.002       |
| 400p       | 2560×400     | 16/3              | 32/4             | 80/25          | -/-      | 60.000       |
| 544p       | 2560×544     | 16/3              | 32/4             | 80/34          | -/-      | 60.000       |

**Notes**:
> - **CRU slot budget**: Since CRU 1.5.3, slot limits have been removed (previously capped at 6). I've tested 9 custom timings successfully; feel free to add more if needed, CRU changelog claims it will work.
>
> - **480p/720p/1080p**: Standard HDMI modes, no CRU needed.
>
> - **Horizontal timings**: All share Active: 2560, Front Porch: 16, Sync Width: 32, Blanking: 80, Polarity: -. *This uniformity is a consequence of HDMI's super resolution requirement (digital connections and modern drivers reject true low-width signals like 320×240). With HDMI, it's OK to have only vertical blanking vary per resolution.*

![240p in CRU](/images/240.PNG)

### CRU Setup Steps

1. **Open CRU**  
   Launch CRU and confirm your scaler/display is selected (top left, marked "active"). If not, choose it from the dropdown.

2. **Clear Defaults**  
   In **Detailed Resolutions** (top right), delete existing resolutions to free up three slots.

3. **Add Resolutions (9-slot layout)**  
   - **Detailed Resolutions** (3 slots): 160p, 224p, 240p  
   - **Extension Blocks → CTA-861 → Edit** (3 more slots): 256p, 272p, 384p 
   - **Extension Blocks → add a new CTA-861 → Edit** (And 3 more slots!): 144p, 400p, 544p  

4. **Save and Restart**  
   Click **OK** twice → Run `restart64.exe` **(CRU folder)** or Reboot

5. **Verify 224p/240p**  
   RetroArch → Load SNES/MD/PS1 game → **Main menu → Settings → Video → CRT SwitchRes**:  
   - **ON**, **15 kHz**, **Super res: 2560**  
   - Confirm game resolution switches to 224p/240p 
   
### CRU Troubleshooting 
 
6. **If game doesn't launch at 240p**:
	- **Check**: Timings exist in CRU + values match table
	- **Run**: `restart64.exe` (CRU folder) → Test RetroArch
	- **Reboot** → Test RetroArch  
	- **Verify**: Custom resolutions appear in Windows Display Settings
	
 > **If all fail** → GPU likely incompatible

🎮

*3DS in original 400p, Dragon Ball Z:Extreme Butoden*
![400p3DS](/images/3DSDBZ.jpeg)
   
---

## 📺
## Emulator Configuration

**Goal**: Native resolution output → leave the upscaling/CRT effects job to the scaler!

**Core rules**:
 - Output at native resolution  
 - Maintain original aspect ratio
 - No smoothing (nearest-neighbor only)  
 - Internal upscale (2-3×) OK for 3D games, native output 
 
 > **Note:** This section mostly treats aspect ratio correction lightly (stretch-to-fit, basic settings). For pixel-perfect geometry with mathematical precision, see the **[Perfect Aspect Ratio Guide](PerfectAspectRatio.md)**.

---

### Resolution Handling

**Emulators handle resolution in 3 ways** (SwitchRes in Retroarch for 224p & 240p, desktop default, config files). For *desktop default* (Emulator launches at desktop resolution), we will need [Res-O-Matic](#res-o-matic-for-custom-resolutions) if we want a different game resolution. The following table shows the method per system:

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
| 3DS                         | Azahar                | 400p/800p      | Res-O-Matic     | Stacked screens,                                   |
| PS3                         | RPCS3                 | 720p           | Res-O-Matic     | Use if desktop ≠ 720p                              |
| PS Vita                     | Vita3K                | 544p           | Res-O-Matic     | Use if desktop ≠ 544p                              |
| Switch                      | Ryujinx               | 720p           | Res-O-Matic     | Use if desktop ≠ 720p                              |
| Arcade (FBNeo/etc. cores)   | RetroArch             | 224p–240p      | CRT SwitchRes   | Automatic, handles TATE                            |
| Arcade (various MAME)       | MAME standalone       | 224p-480p      | Config file     | Set per-system in `.ini` files                     |
| Sega Model 3                | Supermodel            | 384p           | Config file     | Set in `Supermodel.ini`                            |
| Steam/PC games              | N/A                   | 720+           | In-game settings| Configure in graphics options                      |

**Rendering Rules**:
- **Pre-480p (NES→PS1)** (2D): 1× internal, nearest-neighbor only (no xBR/SuperEagle)  
- **Pre-480p (NES→PS1)** (3D): 2-3× internal + Anti-aliasing and anisotropic OK, native output  
- **480p+ (Dreamcast→Switch)**: 2-3× internal + AA/anisotropic OK, native output required

---

##  📺
### RetroArch Configuration

1. **Global Settings**

- **Menu**: Settings → User Interface → **rgui** (Not the sexiest skin but best for low-res)
- **Controllers** (for reWASD harmony):  
  Menu → Settings → Drivers → **Input**: `dinput` + **Controller**: `dinput`; → **Save config**
  
Why `dinput`? `SDL2` = Bluetooth issues, `xinput` in RetroArch steals Home button from reWASD.

2. **Per-Core Setup** (all cores)
- Load game → Main menu → Settings → Video → Scaling → **Aspect Ratio: Core Provided**  
- Quick Menu → Overrides → **Save Core Override**

---

*Chrono Trigger PS1*
![PS1 Example](/images/crono.jpg)  

---

3. **CRT SwitchRes Cores** (224p/240p auto-switching)
   - **PS1 (Mednafen)**: Main menu → Settings → Video → CRT SwitchRes **ON**, 15kHz, 2560 horizontal 
   - Settings → Video → Scaling → Aspect: **Core Provided**
   - Quick Menu → Overrides → **Save Core Override**

4. **Arcade (FBNeo)**:
   - Main menu → Settings → Video → CRT SwitchRes **ON**, 15kHz, 2560 horizontal
   - Quick Menu → **Core Options → Vertical Mode**: **TATE** or **TATE Alternate** (Safe, auto-applies to vertical games only)
   - Main menu → Settings → Video → Scaling → Aspect: **Core Provided** (**Full** if stretched)
   - Core Specific settings = Quick Menu → Overrides → **Save Core Override**, or:
   - Game-specific ratios = Quick Menu → Overrides → **Save Game Override**

5. **[Res-O-Matic Cores](#res-o-matic-for-custom-resolutions)**:
   - **GBA**: Main menu → Settings → Video → CRT SwitchRes **OFF**, 160p via Res-O-Matic
   - **DS (DeSmuME)**: Main menu → Settings → Video → CRT SwitchRes **OFF**, Aspect **Full**, Screen Layout **Bottom/Top** or **Top/Bottom** as preferred, 256p via Res-O-Matic
   - **GameCube (Dolphin)/DreamCast (Flycast)**: Main menu → Settings → Video → CRT SwitchRes **OFF**, Main menu → Settings → Video → Scaling → Aspect **Core Provided**, 480p via Res-O-Matic
 
### Aspect Ratio Troubleshooting For Any Core:
 > For mathematically correct ratios, see [Perfect Aspect Ratio Guide](PerfectAspectRatio.md).
 > **Here, quick fixes for stretched/wrong displays:**
 >  - **If stretched/wrong** (esp. vertical arcades), Aspect Ratio: **Full**  
 >  - **Per-game exception**: **Game Override** (not Core Override) — saves for *specific game only*  
 >  - **Safe with 4:3 profile in the scaler**: RT4K enforces target ratio regardless of "Full" ratio setting.
 
---
 
*Final Fantasy 5 Advance at 160p*
![FF5GBA](/images/GBAFF5.jpeg)  

## 🕹️
### Standalone Emulator Configuration

1. **MAME 0.272 (Per-System Settings, ST-V 224p example.)**

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
   - For vertical games (e.g., Shienryu), create `/MAME/shienryu.ini` with:
     ```ini
     rotate            1
     ```
   - Launch the game, adjust rotation if needed, then save *Game* settings (not system settings).

   For other platforms (Namco, SNK), repeat this process and use a 640×480@60 or desired resolution.

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
  
 > Supermodel renders at 2480-wide (the correct pre-compensated width) but outputs to the closest available EDID timing (2560×384), automatically adding 80 pixels of black bars (40 each side). When your scaler applies 4:3 scaling, the active content scales to the original Model 3 ratio.
 > ⚠️ This relies on Supermodel’s behavior (stretch to INI resolution, output to closest EDID with black bars). Do **not** create 2480×384 in CRU, it must remain “unavailable” for this trick to work.

3. **Azahar (3DS)**

   - Set View → Screen → **Rotate Up Right** for vertical stacked screens.  
   - If the screen flips in the wrong direction for your setup, you will have to use iRotate instead (See below image).
   - By default, Azahar runs at desktop resolution; use [Res-O-Matic](#res-o-matic-for-custom-resolutions) in LaunchBox for native 400p.

4. **PCSX2 (PS2), RPCS3 (PS3), Ryujinx (Switch), Vita3K (PS Vita)**

   - **RPCS3**: On Windows 11 with recent GPU drivers, current RPCS3 builds should work well.  
     - Prefer **Vulkan** (often best-performing), unless per-game settings suggest otherwise.

   - **PCSX2**:  
     - Use the latest builds.  
     - Renderer: Prefer Vulkan (often best-performing).  
     - Scaling: Use 2×–3× internal resolution selectively when it improves image quality. Keep filtering minimal to preserve a faithful low-res look.

   - **Output Resolution**:  
     - Set each emulator to output the native mode (e.g., 480p content at 480p), and only use higher output modes when you have a specific reason (compatibility with your scaler etc.).  
     - For resolutions different from your desktop resolution, use [Res-O-Matic](#res-o-matic-for-custom-resolutions).

5. **PPSSPP (PSP)**

   - Launch the emulator at 272p using [Res-O-Matic](#res-o-matic-for-custom-resolutions) in LaunchBox.  
   - PSP’s native aspect ratio is 30:17 (≈1.7647:1), slightly off from 16:9. To preserve this perfectly:  
     - Set your scaler to 16:9.  
     - Edit `ppsspp.ini` (typically in `C:\Users\<YourAccount>\Documents\PPSSPP\PSP\SYSTEM\`):
       ```ini
       [Graphics]
       DisplayAspectRatio = 5.294000
       DisplayStretch = False
       ```
     This adds *very* slim black bars on the sides, pre-compensating for the scaler’s 16:9 stretch to display the correct PSP aspect ratio.

6. **Vertical Mode (iRotate + LaunchBox Automation)**

   If you need to rotate your display for a specific emulator that doesn't allow it in settings, you can use iRotate in a LaunchBox script:

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

![irotate](/images/rotate.png)
*This script assumes Escape is the emulator shutdown key. Adjust the keybinding if different. Also replace "Azahar.exe" by the executable of your emulator.*

 > ⚠️ When using iRotate automation, **disable all quit keys** (eg. `ESC`) in the emulator's settings (eg. Azahar → Settings → Hotkeys → unbind anything related to `ESC`). 
 > The LaunchBox script must be the ONLY handler for `ESC`; if Azahar also responds to `ESC`, conflicts may occur.

---

## 💻
### Steam

**Controller fixes** (for reWASD harmony):
 - Steam → Settings → Controller:
   - **Disable** Guide Button Focuses Steam
   - **Disable** Enable Guide Button Chord

- (Optional) Steam → Settings → **Notifications**:
  - **Disable** any uneeded notifications

**Local installs only**:
 - Steam → Settings → Remote Play → **OFF** *(Prevents streaming to main PC when both are online)*

---

## 🎮
## reWASD Setup

Connect all controllers and start reWASD.

1. **Profile Creation**  
   - Delete default profiles if you do not need them and create a new one (e.g., “GameMaze”).  
   - Select a gamepad from the bottom icons (e.g., DualSense).

2. **Set Output to Virtual Xbox**  
   - Click the Xbox icon (top center, “Output Devices Settings”).  
   - Enable **Virtual Xbox 360** (or the XInput/SDL variant of your choice) and save.

![reWASD Output Settings](/images/rewasdOUT.png)

3. **Configure Hotkeys**  
   - Click **Home button** on gamepad schematic  
   - **Shift Mode → Jump to Layer 1**  
   - Switch to **Shift Layer 1** (top center)  
   - Map: **South → TAB**, **Start → ESC**  
   - **Save profile**

4. **Apply to All Gamepads**  
   - Select each gamepad and click **Apply to Slot 1**, **Slot 2**, etc.  
  
5. **Configure reWASD settings**   
   - Ensure reWASD starts with Windows: Settings (gear, top right) → General → **Start with Windows**.
   - Ensure keys remapping is always active: Settings → tray agent → and tick "Restore remap on startup".
   - In reWASD settings, you can also tweak **Overlay** options to disable unwanted notifications.

6. **Test in Emulators**  
   - Open RetroArch and each standalone emulator to verify controls.  
   - If needed, rebind in emulator settings, selecting XInput or SDL as input type.

## ⌨️
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

## 📚
## LaunchBox Configuration

This is not a full LaunchBox tutorial; the focus is on unified quitting, custom resolutions, and popup handling.
You can easily find general documentation online, eg. [Just Jamie's Tutorial](https://www.youtube.com/watch?v=Z5HNhp4zQsM).  

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
    - Repeat for other emulators (replacing `rpcs3.exe` with the emulator executable name and `ESC` with your preferred quit key).

![LaunchBox Quit Script](/images/close.png)

- **Optional Extra polish** *(Mostly if popup windows bother you)*:
	- Tools → Manage Emulators → select emulator → Details:  
	  - Enable **Attempt to hide console window on startup/shutdown**.  
	  - Under **Startup Screen**:  
		- Enable **Aggressive Startup Window Hiding** (helps hide transient launcher windows).  
	  - Enable **Hide All Windows that are not in Exclusive Fullscreen Mode** to help keep BigBox in focus.

---
	  
 ### Gamepad-Only Popups Killer (Rare but Critical)

**Problem**: Mouse-only dialogs blocking gamepad flow. (eg. Windows warnings etc.)

If a game triggers such clickable messages:

1. Run **WindowSpy.ahk** (included with AutoHotkey 1.1) → hover popup → copy `ahk_class` + `title`
2. Create **wrapper script** that launches game → waits for popup → auto-closes (Enter/Alt+F4)
3. **LaunchBox**: Point game's Application Path to **wrapper .bat** (not executable)

*Note: Launchbox can't launch .ahk scripts but it can launch .bat scripts. So we use a .bat wrapper in Launchbox that will trigger our .ahk script.*

### Example structure:

**ROMs/SteamGame.bat:**
```batch
@echo off
start "" "SteamGameWrapper.ahk"
```

SteamGameWrapper.ahk (same folder):
```autohotkey
SteamGameWrapper.ahk:
Run, steam://rungameid/12345
Sleep, 5000
Loop {
IfWinExist, ahk_class #32770, Game Updater
Send {Enter}
Sleep, 500
}
```

LaunchBox: Set game/ application path to SteamGame.bat (not Steam launcher).

WindowSpy additional documentation: [WindowSpy Tutorial by TAB NATION](https://www.youtube.com/watch?v=_vnmuQV6_cM)

---

## 📺
## Res-O-Matic for Custom Resolutions

Res-O-Matic forces a specific desktop resolution, useful **When CRT SwitchRes fails or "desktop-res-based standalone emulators" need a different res:**

The approach is to create `.bat` launchers in emulator folders and configure these as alternative “emulators” in LaunchBox.

**For the following examples:**
- Adjust paths and core names to match your setup. 
- Check the exact core naming in `RetroArch\cores\`.  
- `reso.exe` is the Res-O-Matic executable found in your installation directory.
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

![Nintendo DS Emulator](/images/csEmul.png)

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
  - Add `PSP_272p.bat` as a new emulator in LaunchBox for the PSP platform.


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

## 🎮
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

In BigBox, launching the "BT-Toggle" fake ROM triggers the Bluetooth toggle, letting you reconnect controllers in your desired order (Player 1 first, Player 2 second, etc.) **(tested with 4 players)**.

To make it blend into your BigBox theme:

- In LaunchBox, filter by platform → BT-Toggle → right-click → Edit metadata.  
- Add images (e.g., Bluetooth logo, gamepads) as banner / clear logo override / device / fanart, depending on your theme.

![LaunchBox Platforms](/images/platforms.png)  
![BT-Toggle Platform](/images/BTplatform.png)  
![LaunchBox Interface](/images/LB.png)

---

## 💎
## Final Touches

**Goal**: Console-like experience — No distraction, no telemetry, no notifications, fast BigBox boot.

### Debloating Windows (Optional)

**Windows Settings** (Settings app):
- **Startup Apps**: Task Manager (Ctrl+Shift+Esc) → Startup → **disable OneDrive/bloat** *(keep reWASD/LaunchBox)*
- Privacy → **Disable telemetry** 
- System → **Turn off notifications/Action Center**
- Update → **Pause updates** + disable notifications

**Third-party Tools**:
- **O&O ShutUp10**: Recommended settings (telemetry + notifications)
- **Winaero Tweaker**: Lockscreen OFF, Action Center OFF, SmartScreen OFF
- **All-in-One alternative - Chris Titus WinUtil**'s script: Powershell with admin rights → `irm christitus.com/win | iex` → debloat preset

*Reboot after changes.*

---

### Additional Optimizations

 - Check the [Perfect Ratio Guide](PerfectAspectRatio.md) for in-depth correct ratio settings!
 - **BigBox Shell** (optional): 
   - Options → General → **Enable using as the Windows Shell**.
   - *This replaces Explorer (no standard desktop), ideal for pure console-like experience but inconvenient if you frequently use Explorer, browsers, keyboard, or mouse.*
   - *If enabled, do **not** disable Ctrl+Alt+Del: needed for Task Manager/sign out if BigBox crashes.*
 - **Alternative**: Hide desktop icons  
  *Right-click Desktop → View → **Uncheck "Show desktop icons"***  
  *Get a cool wallpaper... Clean console look without replacing Explorer!*
- **Storage**: ROMs/games → put on an SSD, use **BCU** for cleanup
 - **Backup**: Save all your configs! LaunchBox Tools→Backup, reWASD profiles, save CRU `.bin` export; and store them somewhere safe.
 
---

## Conclusion

This project is about more than just running games. GameMaze brings together multiple generations of hardware and software into a single, consistent, console-like experience. Many small configuration choices and workarounds combine to deliver:

- Native-resolution output across decades of systems.  
- Authentic CRT-style visuals via a modern digital chain.  
- Seamless gamepad-only control from boot to shutdown.  
- Unified access to emulation, arcade, and modern PC titles in one library.

If you are facing similar challenges—wanting both authenticity and convenience on modern displays—this approach should give you a solid, repeatable foundation to build on and customize for your own hardware and preferences.
