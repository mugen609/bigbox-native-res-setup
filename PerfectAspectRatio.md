# 🎯 Perfect Aspect Ratios

*A GameMaze annexe covering mathematically correct geometry for non-standard aspect ratios.*

**← [Back to Main Setup Guide](GameMaze.md)**

---

# Table of Contents
| Section | Description |
|---------|-------------|
| [Compensation chart](#correct-ratio-strategy) | When to use 4:3 vs 16:9, compensation requirements |
| [General Logic](#general-logic) | Output resolution, stretch mechanics, compensation formula |
| [PSP (30:17)](#psp-3017--ppsspp-at-272p) | PPSSPP config file aspect ratio adjustment |
| [Sega Model 3 (1.29:1)](#sega-model-3-1291--supermodel-at-384p) | Supermodel letterbox trick with CRU fallback |
| [Game Boy (10:9)](#game-boy-109-retroarch-sameboy-core-at-144p) | RetroArch custom viewport for 144p |
| [GBA & DS (3:2)](#game-boy-advance--nintendo-ds---32-ratio-at-169) | RetroArch viewport for 160p and 256p |
| [Nintendo 3DS (5:3)](#nintendo-3ds-53---azahar-at-400p) | Custom layout coordinates for dual screens |
| [PS Vita (30:17)](#ps-vita-3017-vita3k-544p) | ReShade aspect correction shader setup |

---

## The Two-Profile Approach

The RetroTINK 4K can trim images to exact ratios and save it per profiles, which is fine for physical consoles (eg 5 consoles → 5 RT4K profiles). But for a single emulation box like GameMaze outputting everything over one HDMI connection, I feel that constantly switching scaler profiles between systems would break the seamless experience.

**Solution**: Use only **two scaler profiles** (4:3 and 16:9) and handle aspect compensation in software.

## Correct Ratio Strategy

 Which profile and whether compensation is needed depends on the system's native ratio:
 
> - **Native ratio = 4:3** → Use 4:3 scaler profile, no compensation needed (PS1, SNES)
> - **Native ratio < 4:3** → Use 4:3 scaler profile, compensation required (Game Boy 10:9)
> - **Native ratio = 16:9** → Use 16:9 scaler profile, no compensation needed (PS3, Switch)
> - **Native ratio between 4:3 and 16:9** → Use 16:9 scaler profile, compensation required (GBA 3:2, PSP 30:17)

What makes this harder is that every system handles ratio compensation differently. RetroArch viewport configs, standalone emulator INI values, or even ReShade scripts. The following sections provide exact settings for several platform. I can't test all existing systems but this should give you a framework.

*Compensation chart*:
| System                                | Native Res | Ratio | Scaler ratio | Compensation       |
|---------------------------------------|------------|-------|--------------|--------------------|
| **8-bit** (NES, Master System)        | 256×240    | 8:7   | 4:3          | No (4:3 CRT design)|
| **16-bit** (SNES, Megadrive)          | 256×224    | 8:7   | 4:3          | No (4:3 CRT design)|
| **32-bit** (PS1, Saturn)              | 320×240    | 4:3   | 4:3          | No                 |
| **Gen 6** (PS2, Dreamcast)            | 640×480    | 4:3   | 4:3          | No                 |
| **Modern 720p+** (PS3, Switch, Steam) | 720p+      | 16:9  | 16:9         | No                 |
| **Sega Model 3**                      | 496×384    | 1.29:1| 4:3          | Supermodel ini     |
| **Game Boy**                          | 160×144    | 10:9  | 4:3          | RA core cfg        |
| **Nintendo DS** (vertical, on top)    | 384×256    | 4:3   | 4:3          | RA core cfg        | 
| **PSP**                               | 480×272    | 30:17 | 16:9         | PPSSPP cfg         |
| **GBA**                               | 240×160    | 3:2   | 16:9         | RA core cfg        |
| **3DS** (vertical, on top)            | 480×400    | 5:3   | 16:9         | Custom Layout      |
| **PS Vita**                           | 960×544    | 30:17 | 16:9         | ReShade cfg        |

### **Important Note**: examples below use a 2560-wide super-resolution canvas; if your width differs, recompute the active width / black bars with the same ratios.

## General Logic

 - **Output resolution**: 2560 (super resolution width) × native height (e.g., 2560×240 for SNES)
 
 - **Emulators stretch to fill**: When set to "Full" or "Stretch," the original square-pixel content (e.g., 320×240) is stretched horizontally to fill 2560×240, creating a severely deformed image
 
 - **Scaler corrects the stretch**: The scaler enforces 4:3 or 16:9, which reverses the horizontal stretch and displays the correct proportions on your display. For true 4:3 or 16:9 systems, this works perfectly: 2560×240 stretched and forced to 4:3 by the scaler looks identical to original hardware 320×240 output
 
 - **The problem**: When your target ratio is *not* 4:3 or 16:9 (e.g., Game Boy's 10:9), the scaler over-corrects. You must add **letterbox bars** (black bars on the sides) to pre-compensate for the scaler's forced ratio

 > **Example:** Blue Reflection (PS Vita) outputs at 2560×544. With the scaler set to 1:1 (Square Pixel), the image appears stretched horizontally. Setting the scaler to 16:9 + applying ratio compensation fixes this.
 >
 > The slim black bars visible on the image's sides are generated by the scaler's **Left Trim: +19px** to enforce correct geometry. But we will rather achieve the same result mathematically at the emulator level, keeping scaler profiles minimal.

  *Blue Reflection stretched at 1:1 (Square Pixel) - fixed via 16:9 profile + compensation*
  ![BlueReflectionVita](/images/VitaBR.jpeg)  

---

**Compensation Formula (the modifier $m$)**:

$$
m = \frac{\text{original ratio}}{\text{scaler ratio}}
$$

---

The compensation factor $m$ tells you what percentage of the 2560-pixel width should contain active image vs. black bars.

**Example**: 4:3 content on a 16:9 scaler

1. Calculate $m$:

$$
m = \frac{4/3}{16/9} = \frac{4}{3} \times \frac{9}{16} = \frac{3}{4} = 0.75
$$

---

2. Active width:

$$
2560 \times 0.75 = 1920 \text{ pixels}
$$

---

3. Black bars (total):

$$
2560 - 1920 = 640 \text{ pixels} \quad (320 \text{ left} + 320 \text{ right})
$$

---

**Result**: Configure the emulator to output 2560×H letterboxed, with 1920×H active content centered and 320 pixels of black bars on each side.


The following sections detail how to achieve this for different systems, as emulators handle letterboxing differently.


# 🕹️ 
## PSP (30:17) — PPSSPP at 272p

I shared the magic numbers in [Standalone Emulator Configuration](GameMaze.md#standalone-emulator-configuration), but here's the derivation.

> **Tested with**: Vulkan renderer

### The PPSSPP Challenge

Most emulators stretch content to fullscreen (2560×272), then you add letterbox bars to compensate. PPSSPP works differently:

- Its **Stretch** option locks output to fullscreen and doesn't allow ratio adjustment
- Its **Aspect Ratio** slider lets you scale the image horizontally, but can't be used with Stretch
- The slider maxes out at 2.0 in the UI, but we need a higher value

**Solution**: Edit the config file to unlock higher aspect ratio values.

### The Math (Two Steps)

**Step 1: Expand from squished to 16:9**

PPSSPP renders at 480×272 but outputs it in a 2560×272 canvas, leaving the image horizontally severely compressed, with massive black bars. 

---

First, we need to stretch it horizontally to fill 16:9:

$$
\frac{2560}{480} = 5.333333
$$

---

Setting `DisplayAspectRatio = 5.333333` would give us proper 16:9.

**Step 2: Compensate from 16:9 down to 30:17**

Now apply our compensation formula:

$$
m = \frac{30/17}{16/9} = \frac{30}{17} \times \frac{9}{16} = 0.9926
$$

---

Multiply the 16:9 value by the compensation factor:

$$
5.333333 \times 0.9926 = 5.294
$$

---

### Configuration

Edit `ppsspp.ini` (typically in `C:\Users\YourAccount\Documents\PPSSPP\PSP\SYSTEM\`):

```ini
DisplayScale = 1.000000
DisplayIntegerScale = False
DisplayAspectRatio = 5.294000
DisplayStretch = False
```

Result: With your scaler set to 16:9, PSP content displays at its native 30:17 ratio with slim pillarbox bars.

# 🕹 ️
## Sega Model 3 (1.29:1) — Supermodel at 384p

This was briefly covered in [Standalone Emulator Configuration](GameMaze.md#standalone-emulator-configuration); here's the full derivation.

### The Supermodel Trick

Supermodel doesn't let you set aspect ratios directly. Instead, you specify a target resolution, and it letterboxes that image into the **closest available EDID timing** (your CRU custom resolutions). This quirk becomes our solution.

### The Math

Model 3 games render at **496×384** (1.29:1), which is narrower than 4:3. We'll use a 4:3 scaler profile.

If we set Supermodel to output 2560×384 and force 4:3 in the scaler, we'd get perfect 4:3—slightly wider than the original 1.29:1. We need to pre-narrow the image.

---

Apply the compensation formula to the output width:

$$
m = \frac{496/384}{4/3} = 0.96875
$$

$$
2560 \times 0.96875 = 2480 \text{ pixels}
$$

---

So the active content should be **2480×384**, which will generate 80 pixels of black bars (40 per side) when the nearest working resolution is **2560×384**.

### Configuration

Edit `Supermodel.ini` in `/supermodel/Config/`:

```ini
FullScreen = 1
XResolution = 2480
YResolution = 384
Stretch = 1
```

**How It Works**

Important: Do not create a 2480×384 timing in CRU. Only 2560×384 should exist.

When you launch a game:

 1. Supermodel renders at 2480×384 and tries to output that resolution

 2. No 2480×384 timing exists, so it falls back to the nearest available: 2560×384

 3. Supermodel letterboxes the 2480 image inside the 2560 canvas (40-pixel black bars on each side)

 4. Your scaler receives 2560×384 and applies 4:3, revealing the perfect 1.29:1 ratio

 *Result:* Authentic Model 3 aspect ratio using only standard 4:3 scaler settings.
 
# 🕹️
## Game Boy (10:9) - RetroArch SameBoy Core at 144p

*Successfully tested with: SameBoy libretro core*

### The RetroArch Viewport Challenge

With RetroArch, the challenge is entirely different. We can't manipulate aspect ratios with simple presets and must use custom viewport configuration instead.

### The Math

Game Boy displays at **160×144** (10:9 = 1.111:1), which is narrower than 4:3. We'll use a 4:3 scaler profile.

---

Apply the compensation formula:

$$
m = \frac{10/9}{4/3} = \frac{10}{9} \times \frac{3}{4} = 0.8333
$$

$$
2560 \times 0.8333 = 2133.33 \approx 2134 \text{ pixels}
$$

> **Rounded up to 2134 (not down to 2133)** to get an even number: this ensures symmetrical centering with whole-pixel black bars on each side.

So the active Game Boy image should be **2134×144**, centered in a 2560×144 canvas with 426 pixels of black bars (213 per side)..

### Configuration

Find or create `SameBoy.cfg` in `RetroArch/config/SameBoy/` and edit with Notepad++ or any text editor:

```ini
video_fullscreen_x = "2560"
video_fullscreen_y = "144"

video_scale_integer = "false"
video_force_aspect = "false"

aspect_ratio_index = "23"

custom_viewport_width = "2134"
custom_viewport_height = "144"
custom_viewport_x = "0"
custom_viewport_y = "0"
```

**Key settings:**

aspect_ratio_index = "23" enables custom viewport mode

custom_viewport_width = "2134" sets the active image width (the compensated value)

x = "0" and y = "0" center the viewport automatically when smaller than fullscreen

Result: With your scaler set to 4:3, Game Boy games display at their native 10:9 ratio with slim lateral pillarbox bars.

# 🕹️
## Game Boy Advance & Nintendo DS - 3:2 Ratio at 16:9

Both GBA and DS require custom viewport configuration in RetroArch to achieve proper geometry when using a 16:9 scaler profile.

> **Successfully tested with:**  
> - **GBA**: mGBA libretro core  
> - **DS**: DeSmuME libretro core

### The RetroArch Viewport Challenge

RetroArch's custom viewport system lets you define the active image area within the output canvas. While it is possible to adjust it through the UI (Quick Menu → Settings → Video → Scaling), it's really tedious and direct config file editing is much easier!

---

### The Math

Both systems display at **3:2 aspect ratio**, which is narrower than 16:9 but wider than 4:3. We'll use a 16:9 scaler profile.

---

Apply the compensation formula:

$$
m = \frac{3/2}{16/9} = \frac{3}{2} \times \frac{9}{16} = 0.84375
$$

$$
2560 \times 0.84375 = 2160 \text{ pixels}
$$

---

So the active image should be **2160 pixels wide**, centered in a 2560-wide canvas with 400 pixels of black bars (200 per side).

*Final Fantasy VI Advance at native 160p with correct 3:2 geometry*
![FF6GBA](/images/GBAFF6.jpeg)

---

### Configuration: Game Boy Advance

**GBA native resolution**: 240×160  
**Output**: 2560×160 (160p)

Find or create `mgba.cfg` in `RetroArch/config/mGBA/`:

```ini
video_fullscreen_x = "2560"
video_fullscreen_y = "160"

video_scale_integer = "false"
video_force_aspect = "false"

aspect_ratio_index = "23"

custom_viewport_width = "2160"
custom_viewport_height = "160"
custom_viewport_x = "0"
custom_viewport_y = "0"
```

*Black Sigil (Nintendo DS) at 256p with TATE rotation*
![BlackSigilDS](/images/DSBS.jpeg)  

### Configuration: Nintendo DS 
**DS native resolution**: 256×192 per screen 
**Output**: 2560×256 (stacked vertical layout with rotation) 

Find or create `desmume.cfg` in `RetroArch/config/DeSmuME/`:

```ini
video_fullscreen_x = "2560"
video_fullscreen_y = "256"

video_scale_integer = "false"
video_force_aspect = "false"

aspect_ratio_index = "23"

custom_viewport_width = "2160"
custom_viewport_height = "256"
custom_viewport_x = "0"
custom_viewport_y = "0" 
 
video_rotation = "1"
```
 > Note: `video_rotation = "1"` enables 90° rotation for TATE display orientation.
 
### Key Settings Explained
 - aspect_ratio_index = "23" enables custom viewport mode

 - custom_viewport_width = "2160" sets the active image width (the compensated value)

 - x = "0" and y = "0" center the viewport automatically when smaller than fullscreen

 - **Result**: With your scaler set to 16:9, both GBA and DS games display at their native 3:2 ratio with slim lateral pillarbox bars.

# 🕹️ 
## Nintendo 3DS (5:3) - Azahar at 400p

Once you've created the 2560×400 timing in CRU, Azahar is surprisingly straightforward! It skips most formulas and uses direct coordinate positioning, but gets a dedicated section because the settings aren't exactly obvious.

> **Configuration location**: While you *can* edit `qt-config.ini` (typically in `C:\Users\YourAccount\AppData\Roaming\Azahar\config\`), the entire setup is easier through the UI. The config file is really heavy with about 800 lines...

### The 3DS Layout Challenge

The 3DS has dual screens with different dimensions:
- **Top screen**: 400×240 (5:3 ratio)
- **Bottom screen**: 320×240 (4:3 ratio, but here treated as 400×240 with 40px black bars on each side)

When stacked vertically, the combined output is **400×480** (5:6 ratio), which is narrower than 4:3. We'll use a 4:3 scaler profile.

### The Math (Simpler here)

Unlike previous systems, Azahar works with **pixel coordinates** rather than ratio modifiers, so we calculate the final stretched dimensions directly.

The 3DS content at 400×480 needs to fit in a 2560×400 canvas and display correctly when the scaler applies 4:3.

**Finding the correct vertical stretch:**

1. The horizontal super resolution stretch: $\frac{2560}{400} = 6.4 \times$
2. After the scaler applies 4:3, the effective horizontal scale becomes: $6.4 \times \frac{3}{4} = 4.8 \times$
3. To maintain the correct aspect ratio, the vertical dimension must match this scale: $240 \times 4.8 = 1152 \text{ pixels per screen}$


So each 240px-tall screen must be stretched to **1152px** in Azahar's coordinate system to display correctly after the scaler's 4:3 compression.

### Configuration

Navigate to **Emulation → Configure → Graphics → Layout** and follow the screenshot below:
![Azahar Configuration](/images/azahar.png)

**General Settings**:
- **Screen Layout**: Custom Layout
- **Rotate screens upright**: Checked (for vertical stacking)
- **Screen Gap**: 0.00 (seamless transition between screens)
- **Large Screen Proportion**: 4.00 (maintains proper sizing)

**Custom Layout Coordinates**:

| Screen | Setting | Value | Purpose |
|--------|---------|-------|---------|
| **Top Screen** | X Position | 0px | Aligns screen horizontally |
| | Y Position | 128px | Top border offset |
| | Width | 400px | Native resolution |
| | Height | 1152px | Compensated for 4:3 scaler |
| **Bottom Screen** | X Position | 40px | Centers 320px width (use 0px for 400px width*) |
| | Y Position | 1280px | Positions directly below top screen |
| | Width | 320px | Native resolution (or 400px*) |
| | Height | 1152px | Compensated for 4:3 scaler |

**\*Bottom Screen Width Options**:
- **320px width** (X=40px): Authentic aspect ratio, smaller than top screen
- **400px width** (X=0px): Matches top screen size but stretches bottom screen content horizontally

**Result**: With your scaler set to 4:3, 3DS games display in vertical orientation with both screens at their correct aspect ratios (or optionally uniform sizes).

# 🕹️ 
## PS Vita 30:17 Vita3K 544p

Vita3K is quite the pain for aspect ratio perfection. Despite real efforts, I couldn't find any settings to correct the Vita's 30:17 aspect ratio (960×544 native). 

Fortunately, Vita3K is compatible with **ReShade**, which we will exploit to inject a custom aspect correction shader that adds thin black bars and restores perfect 30:17 aspect.

---

### **OpenGL Only**

This guide uses OpenGL for compatibility and simplicity. Vulkan + ReShade is possible but requires manual DLL deployment and Windows Registry editing; may also be unstable on some Windows 11 + AMD configurations (exactly GameMaze expected configuration...).

---

## What is ReShade?

I'm not doing a full ReShade tutorial here, but in short, ReShade is a post-processing injector that lets you apply custom shaders to games via configuration files (in our case, aspect ratio correction).

- **Official site:** https://reshade.me (scroll to bottom for download)
- **Tutorial video:** [ReShade Setup Guide](https://www.youtube.com/watch?v=2ne-yGZA5Ew)

---

## Step-by-Step Setup

### **1. Install ReShade**

1. **Run setup as Administrator**

2. Click **"Browse..."** and navigate to your emulator's folder and select **`Vita3K.exe`**

3. When asked to select rendering API → Choose **OpenGL**

4. When asked about shader packages → **Check "Standard effects"**

5. Click **Finish**

6. **Verify installation:** Check your emulator's folder (where you have Vita3K.exe) for a new file called **`opengl32.dll`** (reShade won't work without it)

---

### **2. Configure Vita3K**

**Edit `config.yml`** (located in your Vita3K root folder, same directory as `Vita3k.exe`):

```yaml
backend-renderer: OpenGL
stretch_the_display_area: true
```

Set your scaler to 16:9

### **3. Configure ReShade Settings**

Edit `ReShade.ini` (ReShade created this in your Vita folder):

Find these sections and ensure they match:
```ini
[GENERAL]
NoReloadOnInit=0

[INPUT]
KeyOverlay=36,0,0,0
```
   - *NoReloadOnInit=0 → Allows ReShade to remember enabled shaders across sessions*

   - *KeyOverlay=36,0,0,0 → Home key opens/closes ReShade menu*
   
### **4. Create Shader Folders**

In your Vita3K folder (alongside Vita3k.exe), create this folder structure if it doesn't exist:

```text
Vita/
├── Vita3k.exe
├── opengl32.dll
├── ReShade.ini
└── reshade-shaders/
    ├── Shaders/       ← Create this folder
    └── Textures/      ← Create this folder
```

### **5. Create Custom Aspect Correction Shader**

Inside `reshade-shaders/Shaders/`, create a new text file named `VitaAspect.fx` and containing:

```
#include "ReShade.fxh"

uniform float HScale <
    ui_type = "slider";
    ui_min = 0.5;
    ui_max = 1.5;
    ui_step = 0.001;
    ui_label = "Horizontal Scale";
> = 1.0074;  // Vita 30:17 ratio correction (1 / 0.9926)

sampler BackBufferBorder
{
    Texture = ReShade::BackBufferTex;
    AddressU = BORDER;
    AddressV = CLAMP;
    BorderColor = 0x00000000;  // Black bars
};

float4 PS_AspectCorrect(float4 vpos : SV_Position, float2 texcoord : TEXCOORD) : SV_Target
{
    texcoord.x = (texcoord.x - 0.5) * HScale + 0.5;
    return tex2D(BackBufferBorder, texcoord);
}

technique VitaAspectFix
{
    pass
    {
        VertexShader = PostProcessVS;
        PixelShader = PS_AspectCorrect;
    }
}
```

 - What this shader does:

    - Narrows the displayed image horizontally by 0.74% (from stretched 16:9, back to native 30:17)

    - Adds thin black bars on left/right to fill the gap

    - The HScale = 1.0074 value is the inverse of 0.9926 (Vita's ratio relative to 16:9)

    - Save as VitaAspect.fx
	
### **6. Enable the Shader (One-Time Setup)**	

 1. Launch a game with Vita3K

     - You should see a brief "ReShade" banner at the top of the screen for ~1 second, confirming reShade is on.

 2. Press the Home key on your keyboard (yes, needs a keyboard for this first-time setup)

     -  The ReShade overlay menu appears

 3. In the "Home" tab, find VitaAspectFix in the techniques list

 4. Click the checkbox next to VitaAspectFix to enable it

     - The image should immediately narrow slightly with thin black bars appearing on the sides

 5. Press Home again to close the ReShade interface

 6. Done! ReShade will remember this setting and the shader auto-enables on every future launch

### Result
Vita games now display in perfect native 30:17 aspect ratio with imperceptible black bars compensating for the difference from 16:9. Circles are perfectly round, UI elements match hardware proportions, etc..
From now on: Just launch Vita3K normally and the correction applies silently every time.

**← [Back to Main Setup Guide](GameMaze.md)**
