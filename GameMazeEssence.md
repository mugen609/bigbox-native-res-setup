# GameMaze Design Philosophy

*An annex documenting the project's purpose, evolution, and differentiation from established emulation approaches.*

**← [Back to Main Setup Guide](GameMaze.md)**

---

## Table of Contents

| Section | Description |
|---------|-------------|
| [Project Genesis](#project-genesis-why-this-exists) | Origin, motivation, and quality hypothesis |
| [Design Constraints](#design-constraints) | Non-negotiable requirements that shaped architecture |
| [Evolutionary Path](#changelog) | From v1.0 to v4.1: discovered vs. assumed solutions |
| [Differentiation](#differentiation) | How GameMaze relates to existing solutions |
| [Technical Contributions](#technical-contributions) | Specific advances in timing, hardware, and integration |
| [Resolution Philosophy](#resolution-philosophy) | Native output vs. upscaling trade-offs |
| [Hardware Strategy](#hardware-strategy) | Modern GPU compatibility and driver approach |
| [User Experience Model](#user-experience-model-how-its-achieved) | Gamepad-only operation and interface unification |

---

 *It’s not just an emulation setup... It’s a reimagined console.*
 
---

## Project Genesis (Why this exists)

GameMaze started as a practical quality problem: games running on my physical SNES or PS1 hardware looked *better* than the same games on emulators, despite using the same scaler.

It was first tempting to wonder if original hardware could have any intrinsic qualities that emulator couldn't replicate.
But the missing ingredient wasn’t actually “hardware magic.” It was the *signal* being sent into the scaler. In a word... **Resolution**.

Most PC setups deliver a pre-upscaled desktop-style frame, which prevents the scaler from doing what it does best: treat the input at native resolution and apply its processing (scaling, masks/scanlines, geometry) in the intended domain.

GameMaze is an attempt to let your scaler perform at its best, no matter what you launch on your PC.

---

## The core requirement (two eras, one box)

The idea never was meant to be “a retro PC” *or* “a modern PC.”
It had to be both, at the same time, behind a single couch-friendly UI:

| Era | Quality target | What “success” means in practice |
|---|---|---|
| 1980s–2000s (arcade, 8/16/32‑bit) | Native output timing (144p–480p) into the scaler | Crisp pixels, no smoothing, scanlines/masks behave correctly, correct geometry |
| 2000s–present (PS2/PS3/Switch/PC) | Modern performance and compatibility | Current GPU drivers, Vulkan/modern APIs, good framerates, no “retro-only hardware lock-in” |

That single requirement (“retro authenticity” + “modern performance” + “single UI”) is what forces the unusual architecture choices in the main guide.

## What GameMaze is (and isn’t)

GameMaze is a Windows configuration framework that glues together:
- A gamepad-first frontend/library (BigBox/LaunchBox) with unified art and navigation.
- Emulators and PC games under one launch flow.
- Low-resolution output strategy (CRU timings + resolution forcing) so the scaler receives native-like signals.
- Input standardization and “no-keyboard” ergonomics (unified controls, remote management, scripts/workarounds).

GameMaze is **not**:
- A CRT / analog-output guide (VGA/SCART/component). *In my tests, analogue GPU worked with super resolution.but manufacturers no longer deliver analogue ports on modern GPU, which prevents playing modern games.*

- A claim that FPGA or CRT-focused solutions are bad.
- A promise of universality across every GPU/scaler combo; a working path and its constraints (discovered along the way) are documented.

## Why the usual solutions didn’t fit

Several well-known approaches are excellent—just not for this combined target:
- MiSTer / FPGA: unmatched for certain retro goals, but not a “one box” answer for modern PC/Steam titles.
- Batocera/Lakka-style appliances: great convenience, but commonly treat retro as content to upscale early (fine for TVs, suboptimal for a scaler-first pipeline).
- CRT EmuDriver / analog modelines: authentic output, but often tied to legacy hardware/driver constraints that conflict with modern system performance goals.
- Standard PC emulation: modern performance, but the output signal and UI ergonomics break the “console-like + scaler-native” target.

The rest of this annex documents the design constraints, the project’s evolution, and the specific technical decisions that made the combined target feasible.

---

# The Emulation Landscape at a Glance

```text
I play on emulators
│
└─ Are you OK using mouse + keyboard (desktop workflow and res)?
  ├─ Yes → Standard PC Emulation
  │ (RetroArch + standalone emulators, desktop-first UX)
  └─ No → Couch / gamepad-only workflow
    │
    └─ Is preserving native-like sub-480p output into a scaler important to you?
      ├─ No → Batocera / appliance-style setup
      └─ Yes → Native-ish timing matters
        │
        └─ Do you also need modern systems / modern PC games in the same box?
          ├─ No → MiSTer OR CRT EmuDriver (retro-first solutions)
          └─ Yes → GameMaze
                     └─(CRU super-res timings + aspect compensation + gamepad-only UX patching)
```

*Notes*:
> - “Batocera / appliance-style” optimizes for simplicity, not native scaler input for pre-480p content.
> - “MiSTer / CRT EmuDriver” are excellent retro-first endpoints but don’t target modern PC/Steam titles.
> - **GameMaze** is a Windows-based configuration that attempts to combine the best of both worlds. 

---

## Design Constraints

The following requirements were established at project inception and remained non-negotiable throughout development:

| Constraint | Rationale | Impact on Architecture |
|------------|-----------|------------------------|
| **Single interface** | No switching between frontends for different eras | Required LaunchBox/BigBox integration across all emulators |
| **Gamepad-only operation** | Couch-based usage, no keyboard/mouse dependency | Required Bluetooth reordering workaround, SSH remote management, unified quit keys |
| **Original resolution output** | Visual authenticity as primary quality metric | Required CRU custom timing development, Res-O-Matic integration |
| **Modern system support** | PS3, Switch, Steam titles must run adequately | Required modern GPU compatibility, not legacy-only solutions |
| **Digital HDMI chain¹** | Compatibility with current displays and scalers | Excluded analog VGA/SCART cards, required super-resolution approach |
| **Maximum two scaler profiles** | Practical limit for seamless switching | Required software-based aspect ratio compensation |

*¹In my tests, analogue GPU worked with super resolution. The reason for excluding them is not that analogue is wrong, rather that the project aims at also handling modern AAA games and manufacturers unfortunately no longer deliver analogue ports on modern GPUs.*

---

## Changelog

| Version      | Key Assumption                                                                                   | Discovery                                                                                                                          | Architectural Change                                                                                                   |
| ------------ | ------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| **v1.0–2.x** | CRT EmuDriver + legacy GPU required for 240p; blank password SSH acceptable for gamepad flow     | Super resolutions (2560×H) function on both old and  modern hardware, SSH security issue                                           | Shift from analog-focused to digital-native approach                                                                   |
| **v3.0**     | Windows 10 + frozen drivers necessary; SSH password free still acceptable for gamepad flow       | Windows 11 + current AMD drivers (tested Feb 2026) maintain 240p output; public key achieves same UX with more security            | Removed driver version constraints, enabled modern API support (Vulkan); migrated to Ed25519 public key authentication |
| **v4.0**     | 224p minimum for HDMI signal stability; CRU 1.5.3 removed the 6 slots limit for customs res      | 144p–160p functional via CRU with uniform horizontal timing                                                                        | Extended resolution coverage to original Game Boy/GBA native resolutions & PS Vita                                     |
| **v4.1**     | Multiple scaler profiles required for aspect ratio accuracy                                      | Mathematical compensation enables two-profile system (4:3/16:9)                                                                    | Created aspect ratio annex with derivation methodology; SSH workflow extracted to standalone annex                     |

**Inflection point**: The Windows 11 upgrade (originally feared as system-breaking) inadvertently updated GPU drivers to current versions. Testing confirmed all retro resolutions remained functional, demonstrating that frozen drivers are not strictly required, **when using super resolutions**.

---

## Differentiation

GameMaze is designed to occupy a specific position in the solution space. The following comparison documents **functional overlap and distinction**, not qualitative ranking:

| Solution | Core Strength | GameMaze Relation | Key Difference |
|----------|--------------|---------------------|----------------|
| **MiSTer/FPGA** | Cycle-accurate hardware replication | Different target use case | FPGA limited to PS1-era; GameMaze includes modern systems via software emulation |
| **Batocera/RetroPie** | Plug-and-play appliance experience | Different resolution philosophy | Batocera typically upscales to 480p/720p; GameMaze prioritizes native timing |
| **CRT EmuDriver** | True 320×240 analog output | Alternative technical path | CRT EmuDriver requires legacy GPUs, unsigned drivers; GameMaze uses super resolutions and hardware apt for recent games |
| **GroovyMAME** | Arcade-authentic modeline generation | Partial overlap | GroovyMAME focuses on arcade cabinets; GameMaze extends to consoles and modern PC titles |
| **RetroArch /Lakka** | Unified libretro core interface with excellent internal gamepad harmonization | Complementary component | Limited by core availability (no Vita, limited modern PC support); standalone emulators may exceed core quality (PPSSPP, etc.) |

**Gap addressed**: No existing solution combined (a) native resolution output for pre-480p systems, (b) modern GPU/driver support, (c) unified gamepad-only interface spanning 1980s–2020s gaming, (d) digital HDMI compatibility with consumer scalers.

---

## Technical Contributions

The following elements represent findings or unusual combinations:

### Timing Methodology

| Aspect | Conventional Approach | GameMaze Approach | Origin |
|--------|----------------------|-------------------|--------|
| **Horizontal timing** | Variable with resolution | **Uniform¹**: 2560 active, FP16, SW32, Blank80, Polarity -/- | HDMI constraint (can't accept true low widths); super resolution simplifies configuration |
| **Vertical progression** | Proportional scaling (e.g., 320p=FP16/4) | **Blanking-only variation**: 9,12,14,15,16,17,24,25,34 | Tested stability across 144p–544p range |
| **CRU slot strategy** | 6-resolution limit | **Unlimited slots** (CRU 1.5.3+) enabling 9+ timings | Version exploitation for extended coverage |

> **¹Note**: Super resolution (2560 width) is required because digital connections (HDMI/DisplayPort) and modern drivers reject true low-width signals (like 320×240). However, HDMI doesn't care about the absolute resolution: it only validates timing and refresh rate. This flexibility allows 2560×144 through 2560×544 to work as valid signals with uniform horizontal timings and only vertical blanking varying per resolution, simplifying configuration compared to analog modeline generation.

### Resolution Coverage

| Range | Typical Community Documentation | GameMaze Coverage |
|-------|--------------------------------|-------------------|
| Sub-224p | Rarely documented; usually upscaled to 224p/240p | **144p, 160p** tested and specified |
| 224p–272p | Standard coverage | 224p, 240p, 256p, 272p |
| 320p–384p | Partial (often skipped) | 320p, 384p |
| 400p–544p | Handheld-specific workarounds | **400p (3DS), 544p (PS Vita)** integrated |
| 480p+ | Standard HDMI modes | Acknowledged as native, no CRU required |

### Hardware Compatibility

| GPU Category | CRT EmuDriver Status | GameMaze Tested |
|------------|---------------------|-----------------|
| Legacy (HD 7750, RX 580) | Supported with 2022 drivers | Confirmed with latest driver if using super resolution (2560) |
| **Modern discrete** (RX 6700 XT) | **Unsupported by CRT EmuDriver** | **Confirmed functional** in my setup|
| Integrated (AMD 780M) | Unsupported | Tested non-functional in my setup|
| Modern NVIDIA RTX | Unsupported | RTX 3090 tested non-functional in my setup|

**Implication**: Modern AMD GPU compatibility (RX 6700 XT confirmed, Feb 2026 drivers) for native resolution output via super resolutions—a capability existing retro-resolution frequently documented as requiring legacy GPUs. Additionally, GameMaze empirically demonstrates that GPUs on the CRT EmuDriver compatibility list (HD 7750, RX 580, etc.) function with current drivers when using super resolution timings, removing the driver version constraint typically associated with these cards.

---

## Resolution Philosophy

Two independent decisions govern visual output:

| Decision                | Options                                  | Impact                                                                                   |
| ----------------------- | ---------------------------------------- | ---------------------------------------------------------------------------------------- |
| **Internal resolution** | 1× native, 2×, 3×, etc.                  | Texture clarity, edge smoothing, GPU load; **no effect on output timing or pixel ratio** |
| **Output resolution**   | Original (144p–480p) or upscaled (720p+) | Scaler input quality, authenticity of signal timing, scanline accuracy                   |

---

**Internal Resolution: Quality vs. Purity**

| Setting         | Use Case                                                              | Example                     |
| --------------- | --------------------------------------------------------------------- | --------------------------- |
| **1× (native)** | Maximum authenticity; raw pixel output                                | 2D sprites, pre-PS1 systems |
| **2×–3×**       | Improved geometry, reduced shimmer; still output at native resolution | PS1 3D polygons, N64, PS2   |


Internal scaling is beneficial as it improves visual quality (mostly on 3D games) without compromising output timing. The output remains native resolution (e.g., 240p) regardless of internal multiplier.

---

**Output Resolution: Authenticity vs. Convenience**

| Approach                           | Output Method                                                                          | Trade-off                                                                                   |
| ---------------------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| **Internal upscaling to 1080p/4K** | Emulator/game handles scaling + post-processing; scaler receives a finished frame      | Simple and compatible, but you lose the “native low-res signal into the scaler” pipeline (scanlines/masks/processing happen in the emulator domain, not the scaler’s). |
| **Native output**                  | Emulator outputs original resolution; scaler handles upscaling and processing          | Requires CRU / mode management, but keeps the scaler as the final stage and preserves native-like timing into it. |

---

**GameMaze Position**

| Content type (practical)                          | Internal resolution (guideline)                        | Output resolution (GameMaze goal)                  | Rationale                                                                                                                            |
| ------------------------------------------------- | -------------------------------------------------------| -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| 2D pixel art (8/16-bit, most sprite games)        | 1× (native)                                            | Native low-res (144p–240p etc.)                    | Maximum authenticity; avoid “helpful” filtering.                                                          |
| Early 3D in low-res eras (PS1/N64/PS2-like cases) | 2×–3× when it helps, otherwise 1×                      | Still native output (typically 240p/480p)          | You explicitly allow 2×–3× internal scaling for 3D when it improves visuals without changing output timing.|
| Modern emulators / PC titles                      | “As high as stable,” then back off (framerate/latency) | Use native modes where it makes sense (480p/720p+) | Performance-bound; no specific multiplier prescribed, anyway it preserves the output pipeline discipline.|

---

## Hardware Strategy

| Strategy                               | Rationale                                                                                                                                        |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Super resolutions over true native** | Modern GPUs reject sub-CEA-861 horizontal widths (e.g., 320×240); 2560×H accomodates HDMI timing requirements while maintaining vertical precision |
| **Current drivers over frozen**        | Security updates, modern API support (Vulkan), and new game compatibility; super resolution approach removes driver version dependency           |
| **Digital HDMI over analog**           | Modern GPUs lack analog output (VGA/DVI-A); digital HDMI enables unified hardware for retro and modern systems                                   |


**Validation**: RX 6700 XT (RDNA2) tested with Adrenalin 2026 drivers; all CRU timings stable, Vulkan-based emulators (RPCS3, yuzu/Ryujinx) functional.

---

## User Experience Model (How it’s achieved)

GameMaze’s “gamepad-first” feel comes from a few concrete layers patched together:
- Windows auto-login + BigBox focus (optional shell / hidden desktop).
- Controller order workaround (Bluetooth toggle bound to a LaunchBox “tool” entry).
- Unified controls via reWASD (e.g., Home+Start = ESC, Home+South = TAB).
- Remote management via key-based SSH + WinSCP (no keyboard needed after setup).
- Consistent quitting behavior via LaunchBox scripts / AHK for edge cases.

---

## Conclusion

GameMaze documents a **functional path** for native-resolution emulation on modern PC hardware via digital HDMI output. It does not claim superiority to analog-output or FPGA-based solutions, which remain valid for specific use cases (CRT display, cycle-accuracy requirements). 

The project's value lies in **extending the solution space**: demonstrating that constraints often considered absolute (legacy GPU requirement, 224p minimum resolution, frozen driver necessity) are **context-dependent rather than technical laws**.


---

**← [Back to Main Setup Guide](GameMaze.md)**

---

**Advanced Guides:**
 - [Perfect Aspect Ratio Tutorial](PerfectAspectRatio.md)
 - [SSH Setup For Gamepad Only](SSH_NoPassword.md) 
