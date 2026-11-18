# FortniteAimbot

A YOLOv8 + TensorRT aimbot for Fortnite with a DX11 overlay, hardware mouse output via Raspberry Pi Pico, and a full ImGui menu.

---

## Features

- **YOLOv8n TensorRT FP16 inference** — runs at 320x320 input, optimized for low latency
- **FOV crop** — crops only the crosshair region before inference instead of downscaling the full screen, giving near 1:1 pixel ratio for distant targets
- **Sticky Lock** — hard-snaps the crosshair onto the detected head dot with zero smoothing lag
- **Hardware mouse output** — sends `dx,dy` over serial to a Raspberry Pi Pico which moves the mouse at HID level, bypassing software-level anti-cheat
- **DX11 transparent overlay** — renders detection boxes and the ImGui menu without a game capture window
- **Humanizer** — adds configurable Gaussian noise to mouse movement to reduce pattern detection
- **Full ImGui menu** — INSERT to toggle, tabbed layout with Aimbot / Overlay / About sections
- **Config persistence** — settings saved to `config.ini` automatically

---

## Requirements

- Windows 10/11 x64
- NVIDIA GPU (RTX recommended for TensorRT FP16)
- Raspberry Pi Pico with the HID firmware flashed (see below)
- Internet connection during install (for winget packages and ImGui clone)
- NVIDIA developer account (free) for TensorRT download

---

## Installation

### 1. Download the installer

Grab `FortniteAimbot_Setup.exe` from the [Releases](../../releases) page.

### 2. Run as Administrator

Right-click → **Run as administrator**. The wizard will:

- Install Git, CMake, Python 3.11 via winget
- Install VS Build Tools 2022 (MSVC)
- Download and install CUDA 12.4
- Clone ImGui from GitHub
- Install Python packages (ultralytics, onnx, onnxsim)
- Configure and build the project with cmake

### 3. Install TensorRT manually (NVIDIA login required)

The installer will pause and show instructions. At that point:

1. Go to [developer.nvidia.com/tensorrt](https://developer.nvidia.com/tensorrt)
2. Download **TensorRT 10.x** for **CUDA 12.x**, Windows, **ZIP package**
3. Extract the ZIP so the folder structure is:
   ```
   C:\TensorRT\
     include\
     lib\
     bin\
   ```
4. Press **ENTER** in the installer window to continue

### 4. Add your model (optional)

Drop your trained `best.pt` into the install folder before the installer reaches the export step. If none is found, it will download `yolov8n.pt` as a fallback.

### 5. Flash the Raspberry Pi Pico

Flash the HID firmware to your Pico so it appears as a USB mouse. The aimbot sends movement over serial (`COM` port) in the format:

```
dx,dy\n
```

at **115200 baud**. Set the correct `COM` port in the ImGui menu → Aimbot tab.

---

## Running

After install, the exe is at:

```
<install dir>\build\Release\AIToolkitOverlay.exe
```

A desktop shortcut is also created. **Run as Administrator** for overlay and serial access.

---

## Controls

| Key       | Action                        |
|-----------|-------------------------------|
| `INSERT`  | Toggle ImGui menu             |
| Hold aim key (default `RMB`) | Activate aimbot |
| `END`     | Exit                          |

---

## Menu Reference

### Aimbot Tab

| Setting       | Description                                              |
|---------------|----------------------------------------------------------|
| Enable        | Master on/off switch                                     |
| Aim Key       | Mouse button or key that activates aiming                |
| Strength      | How aggressively the crosshair moves toward the target   |
| Smoothing     | Divides movement per frame — higher = slower/smoother    |
| FOV Radius    | Pixel radius around crosshair to search for targets. Lower values (80-120) improve detection at range |
| Confidence    | Minimum YOLO detection confidence (0.05 = very sensitive)|
| Aim Offset Y  | Vertical offset from detected box center (negative = up) |
| Humanize      | Adds random Gaussian noise to movement                   |
| Sticky Lock   | Hard-snaps onto the detected head dot, bypasses smoothing |
| COM Port      | Serial port for the Raspberry Pi Pico (e.g. COM3)        |

### Overlay Tab

| Setting        | Description                             |
|----------------|-----------------------------------------|
| Show Overlay   | Toggle detection box rendering          |
| Box Color      | Color of drawn detection boxes          |
| Show FPS       | Display FPS counter in overlay          |

---

## Project Structure

```
FortniteAimbot/
  src/
    main.cpp              - Entry point, capture + inference loop
    aimbot.cpp            - Target selection, smoothing, serial output
    overlay.cpp           - DX11 window, ImGui menu, rendering
    tensorrt_infer.cpp    - DXGI capture, FOV crop, TensorRT inference
  include/
    aimbot.h              - Config struct, Aimbot class declaration
    overlay.h             - Overlay class declaration
    tensorrt_infer.h      - Inference class declaration
    config.h              - Stub (includes aimbot.h)
  externals/
    imgui/                - Dear ImGui (cloned during install)
  installer/
    setup.nsi             - NSIS installer script
    build_installer.ps1   - Builds FortniteAimbot_Setup.exe
  install.ps1             - Full dependency installer (run by setup wizard)
  CMakeLists.txt          - CMake build config
  README.md
```

---

## Building Manually

If you want to build without the installer:

```powershell
# 1. Install dependencies (Git, CMake, Python, VS Build Tools, CUDA, TensorRT)
# 2. Clone ImGui
git clone --depth=1 https://github.com/ocornut/imgui.git externals\imgui

# 3. Export your model
python -m ultralytics export model=best.pt format=onnx imgsz=320

# 4. Build TensorRT engine
C:\TensorRT\bin\trtexec.exe --onnx=models\best.onnx --saveEngine=models\best.engine --fp16

# 5. Configure and build
cmake -B build -G "Visual Studio 17 2022" -A x64
cmake --build build --config Release
```

---

## Pico Serial Protocol

The aimbot sends plain ASCII over serial at 115200 baud:

```
dx,dy\n
```

- `dx` / `dy` — integer pixel deltas, can be negative
- The Pico firmware reads each line and injects a relative HID mouse report

---

## Disclaimer

This project is for **educational purposes** only. Use of aimbots in online games violates the game's Terms of Service and may result in a ban.
