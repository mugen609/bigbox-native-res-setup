# GameMaze - Emulation Station

![BigBox Powered Station](/images/BB.png)

**One gamepad. Four decades of games. Zero compromises.**
- **📖 [Jump to the full setup guide](GameMaze.md)** 

This project turns a Windows PC into a unified emulation console—boot straight into a gamepad-only interface spanning everything from 1980s arcade machines to modern Switch and Steam titles. No stretched pixels. No desktop clutter. Just the games, the way they were meant to be seen and played.

> **Digital-only video chain**  
> HDMI/DisplayPort → scaler → modern flat panel.  
> This is *not* a CRT / analog-output (VGA/SCART/component) guide.

## Why It Exists

High-quality HDMI scalers with CRT shader support achieve their best results when fed **native** signals, not pre-upscaled output. Matching the original console resolutions has a huge impact on how games look and feel.

Most emulation boxes cut corners, either sacrificing visual accuracy or ignoring modern systems. This build aims higher:

- **Native resolution output** (160p–720p) via HDMI for pixel-perfect fidelity  
- **Gamepad-only operation** from boot to gameplay  
- **Support for rotated (TATE) and horizontal displays**  
- **Integration of standalone emulators, RetroArch, Steam etc.**, in one artwork-rich library that feels like a jukebox!

It’s not just an emulation setup. It’s a reimagined console.

## At a Glance

| Feature               | Details                                             |
|-----------------------|-----------------------------------------------------|
| **Frontend**          | LaunchBox / BigBox (fully gamepad-navigable)       |
| **Supported Systems** | NES, SNES, PS1, Dreamcast, Switch, Steam & more    |
| **Display Output**    | True native 160p to 720p over HDMI to a scaler     |
| **Scaler Used**       | Tested with RetroTink 4K for CRT-style upscaling   |
| **Controllers**       | Tested with DualSense, Xbox, Hori — up to 4 players|

From hardware to hotkeys—everything you need to turn Windows into a console.

## Screenshots

![SNES Example](/images/SOM.jpg)  
*Secret of Mana running at original resolution with CRT shader via RetroTink 4K*

## License

Feel free to use, adapt, or share this project. All referenced tools and emulators remain under their original licenses.

**📖 [Full guide here!](GameMaze.md)**
