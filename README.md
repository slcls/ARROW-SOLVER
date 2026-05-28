# <h2><p align="center">**ARROW SOLVER (EXPERT)**</p></h2>

This Python program is an automated solver for the **Arrow Puzzle** minigame (***Expert difficulty***) for the game **"Exponential Idle"**. It implements **OpenCV** (`cv2`) library, utilizing *Canny Edge Detection* to detect and identify numerical shapes accurately, designed to bypass UI background noise and feed color variations.

For android device I/O, I have added two configuration modes. By default, it compiles and pushes batched Unix shell commands via an **ADB** interface for execution stability. But I have also added a low latency mode (unstable on restrictive firmware / unrooted devices), that features an experimental **Scrcpy Socket Bridge** that bypasses the Android virtual machine. It injects touch commands directly into the device's hardware dispatcher to reduce input latency to ~30ms. This framework features a fully automated loops, a persistent statistics logger, and a *validation gate* that autonomously re-scans the screen to validate board completion before claiming rewards.

<p align="center">
  <img src="https://github.com/user-attachments/assets/b38fe21a-c53e-4e40-9d8a-1366725f5820" alt="Arrow Solver Demo" width="300"/>
</p>

## Program Structure
```text
Arrow-Solver-2/
├── core/
│   ├── board.py           # Core logic for board state
│   ├── solver.py          # Math logic to calculate sequences/solution
│   └── validator.py       # Verification gate
├── docs/                  # Assorted documentation/reference images
│   ├── debug_logs.md      # Comprehensive Development Logs
│   ├── solver_demo.gif    # Demonstration GIF
├── io_utils/
│   └── adb_ctrl.py        # ADB batch execution and screen capture
│   └── scrcpy_ctrl.py     # Experimental scrcpy I/O controller
│   └── SCRCPY.txt         # Specific dependencies needed for MODE 2.
├── vision/
│   ├── calibration/       # Contains screen.png (baseline screenshot)
│   ├── templates/         # 1:1 pixel templates (1.PNG - 6.PNG)
│   ├── auto_template.py   # Script to extract templates from screen
│   ├── calibrate.py       # Script to calculate screen coordinates
│   └── scanner.py         # OpenCV logic to read the current board state
├── config.py              # Stores the screen_map (from calibrate.py)
├── logs.txt               # Persistent session statistics
├── main.py                # Master execution loop
└── requirements.txt       # Python dependencies
```

## Prerequisites
Before installing the Python dependencies, your host machine and Android device must be properly configured for hardware bridging.

**1. Install ADB (Android Debug Bridge):**
- Download the [SDK Platform Tools](https://developer.android.com/tools/releases/platform-tools) for your operating system. Extract the folder and add its path to your system's **Environment Variables** (PATH).
- Verify installation by opening a terminal and running: `adb version`.

**2. Enable USB Debugging (Android):**
- Go to your device **Settings > About Phone** and tap the **Build Number** *7 times* to unlock Developer Options.
- Go to ***Developer Options*** and enable **USB Debugging**.
- (Optional but recommended) Enable **Stay Awake** to prevent the screen from locking while the bot runs.

**3. Connect Device:**
- Plug your phone into your PC, run **adb devices** in your terminal, and accept the RSA fingerprint prompt on your phone's screen.

- *You may also opt to use wireless debugging instead, though it may introduce some extra latency.*

## Installation & Setup
**1. Install Dependencies**
```bash
pip install -r requirements.txt
```
**2. First-Time Calibration (Crucial)**
> Because every mobile device has a different resolution, you must map the puzzle's physical coordinates to your screen. **This only needs to be done once.**
- ***Step A: Capture Baseline Screen*:** Take a screenshot of the puzzle on your device and save it exactly as `vision/calibration/screen.png`.
- **Step B: Map the Grid Coordinates:** Run the calibrator:
    ```bash
    python vision/calibrate.py
    ```
    1. Click the **exact center of the MIDDLE tile**.
    2. Click the **exact center of the UPPER-RIGHT tile (referencing from the middle tile)**.
    3. The script will project a grid over the board. If perfectly aligned, press any key to close.
    4. Copy the generated *SCREEN_MAP = {...}* dictionary from the terminal and paste it into `config.py`.
- **Step C: Extract Templates:** The framework needs mathematically perfect reference images to parse the numbers.
    ```bash
    python vision/auto_template.py
    ```
    This automatically crops `1.PNG` through `6.PNG` from your calibration image and saves them to `vision/templates/`.

## Usage (Autonomous Mode)
Once calibrated, the framework operates entirely on its own.
1. Open the **Arrow Puzzle** section on your Android device.
2. Ensure the device is awake and plugged in.
3. Run the master script:
    ```bash
    python main.py
    ```

## Execution Logic
1. **Scan:** Captures `live_screen.png` and uses edge detection to read the numbers.
2. **Solve:** Computes the exact taps required to turn all tiles to `1`.
3. **Execute:** Compiles the taps into a single batched ADB shell command for instant execution.
4. **Verify:** Waits for animations to settle, takes a new screenshot, and verifies all tiles are `1`. If residual numbers remain, it dynamically re-solves.
5. **Claim & Loop:** Once verified, it taps the "Claim Stars" button and repeats the process.

## Kill Switch
The bot runs indefinitely. To stop it, click your terminal window and press `Ctrl + C`. The framework will safely shut down and write your final session statistics to `logs.txt`.

## Statistics & Monitoring
- **`logs.txt`:** Records every successful run, the time taken, and total stars earned. It dynamically reads past sessions so your lifetime total persists across restarts.

- **Live Monitoring (Recommended):** Use `scrcpy` to watch the bot work on your monitor with zero latency. Open a separate terminal and run:
    ```bash
    scrcpy --no-control --no-audio -m 1024 -b 4M --always-on-top
    ```

## Optimizations (v.1.5.0 onwards)
The solver now calculates the absolute shortest physical execution path rather than just a valid path.
- **Logic:** Simulates all 1,296 possible configurations of the board's  top 4 tiles in roughly 30ms.
- **Result:** Reduces USB injection time by 20% to 50% by mathematically eliminating wasted taps. (from ~250ms down to ~130ms)

## Scrcpy Socket Bridge (Experimental I/O)
An ultra-low latency alternative to the standard ADB controller, pushing inputs directly to the hardware dispatcher at 30ms intervals. Includes a "Thumb Squish" micro-nudge to bypass game-engine dead-zone filters.
- Strict Dependencies: The framework autonomously forces downgrades for `adbutils==0.14.1` and `setuptools==69.5.1` to bypass a known Windows PyAV C++ compilation wall.
- ROM Warning: To use this mode on these devices, you must toggle Disable Permission Monitoring (or equivalent) in Developer Options, otherwise some taps won't register.
- I recommend using a rooted device to use `sendevent` command to write directly to `/dev/input/event2`.

## Development & Debug Logs
Building a program compatible for android devices supporting ADB and optimizing latency requires extensive research and testing of Android UI mechanics, OpenCV limitations, and OS thread schedulers.

To ensure the dozens of hours spent diagnosing issues and optimizing the program's performance does not go uncounted, the entire development lifecycle and experimental observations have been documented.

You can read the full development history here:
**[journal.md](https://github.com/slcls/arrow-solver/blob/main/docs/journal.md)**

## Author
[**SLCLS**](https://github.com/SLCLS) - A First-Year **Computer Science Student** at *FEU Institute of Technology*.
- **My Github:** [@SLCLS](https://github.com/SLCLS)
- **See more projects:** [WORKSPACE](https://github.com/SLCLS/WORKSPACE)