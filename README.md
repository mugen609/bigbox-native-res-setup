# GameMaze - Emulation Station

![BigBox Powered Station](/images/BB.png)

**One gamepad. Four decades of games. Zero compromises.**
- **📖 [Jump to the full setup guide](GameMaze.md)** 

This project turns a Windows PC into a unified emulation console—boot straight into a gamepad-only interface, spanning everything from 1980s arcade machines to modern Switch and Steam titles. No stretched pixels. No desktop clutter. Just the games, the way they were meant to be seen and played... But on modern flat panel!

> **works with digital video chain / Relies on HDMI scaler**  
 

## Why It Exists

Most emulation setups “work,” but over HDMI they can still look a bit off -- Soft scaling, weird stretching, and a pre-upscaled image going into a scaler that actually prefers native-like signals. 

Some players don’t notice; if you do, GameMaze focuses on fixing that digital pipeline without giving up modern PC compatibility. 
​
GameMaze aims to:

​ - Feed your scaler low, native output (144p–720p+) over HDMI/DP. 

​ - Keep a couch-based, gamepad-only flow from boot → play → quit. 

​ - Support both horizontal and TATE setups. 

​ - Unify RetroArch, standalone emulators, and Steam/PC games into one library. 

​
Deeper rationale and history **[explained here!](GameMazeEssence.md)**

## At a Glance

| Feature               | Details                                             |
|-----------------------|-----------------------------------------------------|
| **Frontend**          | LaunchBox / BigBox (fully gamepad-navigable)       |
| **Supported Systems** | NES, SNES, PS1, Dreamcast, Switch, Steam & more    |
| **Display Output**    | True native 144p to 720p+ over HDMI to a scaler     |
| **Scaler Used**       | Tested with RetroTink 4K for CRT-style upscaling   |
| **Controllers**       | Tested with DualSense, Xbox, Hori — up to 4 players|

From hardware to hotkeys—everything you need to turn Windows into a console.

*It’s not just an emulation setup. It’s a reimagined console.*

## Screenshots

![SNES Example](/images/SOM.jpg)  
*Secret of Mana running at original resolution with CRT shader via RetroTink 4K*

## License

Feel free to use, adapt, or share this project. All referenced tools and emulators remain under their original licenses.

---

**📖 [Full guide here!](GameMaze.md)**

---

**Advanced Guides:**
 - [Perfect Aspect Ratio Tutorial](PerfectAspectRatio.md)
 - [SSH Setup For Gamepad Only](SSH_NoPassword.md) 

 
