# InMotion CNC Machine Documentation

## Summary

This document provides an overview of the InMotion CNC machine's hardware and software, based on a series of instructional videos. It covers the main electronic components, how to interact with the machine via the console, and the specific commands for machine operation, including motion, spindle control, and the automatic tool changer.

### Key Hardware Components

*   **Power System:** Protected by thermal breakers and a differential for safety. Can be controlled from a front switch. An emergency button (24V) will kill all power.
*   **Logic System:** Includes a custom controller for motor motion, a single-board computer running a Linux-based OS, and power supplies (12V and 24V).
*   **Connectivity:** Features Wi-Fi for network access and a remote hotspot, plus USB ports.
*   **Spindle Control:** Managed by a VFD (Variable Frequency Drive).
*   **Motor Drivers:** AC-operated servo motor drivers for the Z, Y, and X axes.

### Machine Operation via Console

*   The console accepts standard G-code, GRBL commands, and custom tool changer commands.
*   **GRBL Settings:** View with `$$`, and modify with `$<setting>=<value>` (e.g., `$20=1`).
*   **Homing:** `$H` for all axes, or `$Hx`, `$Hy`, `$Hz` for individual axes.
*   **Spindle Control:** `M3 S<speed>` to start, `M5` to stop.
*   **Motion:**
    *   `G0`: Rapid linear move at maximum speed.
    *   `G1 F<feedrate>`: Linear move at a specified speed.
    *   `G92 X0 Y0 Z0`: Set the current position as the work zero.
    *   `G53`: Move in machine coordinate system.
*   **Coolant:** `M7` for air, `M8` for mist, `M9` to turn off.

### User Interface and Workflow

- **Tabs Overview**
  - Jobs Manager, Controls, and Preview are the main tabs for day-to-day use.

- **Jobs Manager**
  - Lists jobs stored on the machine’s SSD. Supports search, sort (e.g., by size, upload date, name), download, and delete.
  - Renaming opens a custom on-screen keyboard automatically. It supports special characters, case toggle, delete, space, and close.
  - USB auto-import: when a USB is plugged in, a file picker pops up to upload selected files directly into the Jobs list.

- **Preview**
  - Shows toolpaths for the selected job. You can pan, zoom, and tilt to inspect paths. It does not display machine position.

- **Controls Tab**
  - Jogging: set step size per-axis; X and Y steps can be locked together or edited independently. Speed modes: slow, normal, fast.
  - Return to Zero returns in X/Y only (keeps current Z for safety).
  - Z moves require caution because the controller doesn’t know actual tool length unless properly set.
  - Many controls (zeroing, tool pick, job management) are disabled while a job is running; only Pause/Stop and overrides remain.

- **Coordinates Panel**
  - Machine coordinates are absolute and repeatable across power cycles (after homing).
  - Job coordinates are relative to the current work zero. You can manually set offsets via the coordinate fields.
  - Tip: take a photo of machine coordinates for later recovery of a physical position.

- **Homing, Alarms, and Resets**
  - On startup, enable the motor drivers (green button), wait a few seconds for drivers/fans to come up, then retry homing.
  - If an alarm persists, use Unlock; combine with Software Reset when needed.
  - Soft limits prevent out-of-range moves; X travel limit is 1000 mm. Max rapid is 7000 mm/min.

- **Overrides During a Job**
  - Feed rate and spindle RPM can be adjusted live via overrides. Feed override up to 130% for safety limits; you can restore defaults anytime.
  - Pausing a job keeps the spindle ON; lower the brushes only when paused and after visually confirming conditions.

- **Tool Change Behavior and Safety**
  - Brushes automatically raise before a tool change to avoid collisions.
  - Motion order during tool change: move to a safe Y first, then X, to avoid colliding with adjacent tools.
  - Tool change is a critical sequence; you cannot modify speed mid-change. You may Stop/Pause, but the tool is only committed at the last position.
  - The machine does not sense tool presence; if you manually insert/remove a tool, update the controller using `GFT T<number>` (use `GFT T0` for no tool) to prevent incorrect assumptions.
  - Starting a toolchanger job with no tool will trigger automatic pickup of the first required tool per the program.

- **Spindle and Timing**
  - Spindle max: 24,000 RPM; spin-up from 0 to max takes ~12 seconds.
  - Toolchange sequences budget time to stop the spindle before picking/placing tools. CAM post-processors may not wait for spin-up at job start; add dwell/startup logic if needed.

- **Logs and Debugger**
  - On errors, an Open Debugger link can show recent controller commands and machine status. Logs can be downloaded for diagnostics.

### Automatic Tool Changer

*   **Pneumatic System:** Enable with `GP1`, disable with `GP0`. Check status with `GPC`.
*   **Tool Release:** `GA1` to release, `GA0` to clamp.
*   **Tool Library:**
    *   `GPAT`: Print all tool information.
    *   `GPT <tool_number>`: Print info for a specific tool.
    *   `STDE T<tool_number>=<description>`: Set tool description.
    *   `STDI T<tool_number>=<diameter>`: Set tool diameter.
*   **Tool Management:**
    *   `GFT <tool_number>`: Force set the tool number in the spindle.
    *   `GRT`: Read the tool number currently in the spindle.
*   **Tool Length Measurement:** Crucial for accurate Z-axis positioning after a tool change. It is recommended to use tools of the same body length to simplify this process, as the automatic touch plate is not currently accurate enough for precise measurements.

---

## Full Transcript

### Hardware Overview

The machine's electrical cabinet is organized for safety and functionality.

*   **Thermal Breakers:** Three breakers protect the logic PCB, motor drivers, and the spindle from short circuits.
*   **Differential:** In series with the breakers, this protects from current dispersion.
*   **Relays:** Two main relays power the machine's systems on and off, controlled by a switch on the front panel. This allows the machine's logic to remain on while the power systems are off for safety.
*   **Emergency Stop:** A dedicated relay and 24V power supply are connected to the emergency button. Pressing this button cuts all power to the machine, including the logic.
*   **Wiring:** Push-fix multi-connectors are used for easy and secure wire connections.
*   **Power Supplies:** Two DC power supplies provide 24V and 12V. The 12V supply powers the screen and parts of the PCB, while the 24V supply powers relays and other systems, including the 24V valves for the pneumatic system.
*   **Logic System:**
    *   **Motion Controller:** A custom controller board is responsible for the motor motion. It connects to motor drivers and limit switches.
    *   **Onboard Computer:** A Ryzen 64-bit CPU with 8 or 16 GB of RAM runs a Linux-based operating system (OpenLab OS) that is displayed on the screen. The software runs in full screen, but Linux is accessible.
    *   **Connectivity:** The computer has two Wi-Fi adapters: one for connecting to a network for internet access (the machine can operate offline) and another that creates a hotspot for remote management (use with caution).
*   **Spindle Control:** A VFD (Variable Frequency Drive) controls the spindle's RPM based on signals from the motion controller. It is pre-configured.
*   **Motor Drivers:** The machine uses AC servo motors, and their corresponding drivers are also operated by AC. The configuration can be changed via buttons on the drivers or by connecting a computer via USB.
*   **Main Switch:** A main switch controls power to the entire machine.

### Console and Basic Commands

The `Console` tab provides a terminal-like interface for sending commands to the machine. It accepts three types of commands:
1.  Standard G-code
2.  GRBL-specific commands
3.  Custom commands for the tool changer

#### GRBL Commands

*   **View Settings:** `$$`
    *   This command lists all the GRBL settings, which control machine parameters like acceleration and speed. These are pre-configured and should not be changed without a good reason.
*   **Change a Setting:** `$<number>=<value>`
    *   Example: `$20=1` sets setting number 20 to the value 1.
*   **Homing:**
    *   `$H`: Homes all axes.
    *   `$Hx`, `$Hy`, `$Hz`: Homes a single, specified axis.

#### G-Code Commands

*   **Spindle Control:**
    *   `M3 S<speed>`: Starts the spindle at a specified speed (e.g., `M3 S10000`).
    *   `M5`: Stops the spindle.
*   **Motion Commands:**
    *   `G0 X<pos> Y<pos>`: Rapid linear move. The machine moves at its maximum configured speed.
    *   `G1 X<pos> Y<pos> F<feedrate>`: Linear move. The machine moves at the specified feed rate (e.g., `G1 X100 F5000`).
    *   `G92 X<pos> Y<pos>`: Sets the work coordinates. `G92 X0 Y0` sets the current position as the zero point for the work offset.
    *   `G53`: Move in absolute machine coordinates, ignoring work offsets. This is useful for moving to a specific physical location on the machine.

### Tool Changer Commands

These are custom commands for operating the automatic tool changer.

#### Pneumatic System

*   `GP1`: Enables the pneumatic system.
*   `GP0`: Disables the pneumatic system.
*   `GPC`: Checks the status of the pneumatic system.
*   `GD1`: Lowers the brushes.
*   `GD0`: Raises the brushes.
*   `GDC`: Checks the status of the brushes.
*   `GB1`: Enables the tool blower.
*   `GB0`: Disables the tool blower.

#### Tool Management

*   `GA1`: Releases the tool from the spindle.
*   `GA0`: Clamps the tool in the spindle.
*   `GFT T<number>`: Force Tool. Manually tells the machine which tool is in the spindle. Use `GFT T0` if no tool is present.
*   `GRT`: Read Tool. Displays the tool number currently in the spindle.

#### Tool Library

The machine has a permanent tool library.

*   `GPAT`: Prints all data for all tools in the library.
*   `GPT <number>`: Prints the data for a specific tool.
*   **Setting Tool Properties:**
    *   `STDE T<number>=<description>`: Sets the description for a tool (e.g., `STDE T3=Best Tool`).
    *   `STDI T<number>=<diameter>`: Sets the diameter for a tool (e.g., `STDI T3=6.5`).
    *   Other properties like tool offset and length can be set with similar commands.

**Important Note on Tool Length:** For the tool changer to work correctly without crashing, the machine must know the precise length of each tool. The automatic touch plate in the back is not accurate enough for this.
*   **Recommended Method:** Use tools that all have the same body length. This way, no software offsets are needed.
*   **Manual Method:** Manually measure each tool's length very precisely and input the offset into the tool library. Any error in this measurement can cause the machine to crash.

#### Measuring Tool Length

*   `MTL T<number>=<estimated_length>`: Initiates the automatic tool length measurement sequence.
    *   Example: `MTL T6=32`
    *   This command will move the machine to the touch probe, clean the tool with air, and then touch the probe to measure the length.
    *   **CRITICAL:** Ensure a tool is actually in the spindle before running this command. Failure to do so will cause a crash.
    *   **Note:** This feature can be unreliable due to electrical noise. It may fail to trigger or give inconsistent results.

### Coolant Commands

*   `M7`: Enables air coolant.
*   `M8`: Enables mist coolant.
*   `M9`: Disables all coolant.

If you try to use a coolant command when the pneumatic system is disabled (`GP0`), you will receive a warning. You must enable it first with `GP1`.

### Troubleshooting

*   **UI Freeze:** If the user interface freezes, you can reset it:
    1.  Press `F11` to exit full-screen mode.
    2.  Close the application window (you may need to use `Alt+F4`).
    3.  The application is configured to restart automatically after being closed.
*   **Position Loss:** If you need to restart the machine, take a picture of the work coordinates on the screen beforehand so you can manually return to the same position using the `G53` command.
*   **Startup/Homing Fails:** Enable motor drivers (green button) and wait a few seconds before retrying homing. If alarms persist, use Unlock and, if necessary, Software Reset.
*   **Pause Behavior:** Pausing does not stop the spindle. Only lower the brushes when paused and never during a tool change.
*   **Out-of-Range Moves:** Soft limits prevent overshoot. If a move is rejected (e.g., beyond 1000 mm X travel), reduce the commanded distance or jog back within range.
*   **Tool Presence Mismatch:** The controller cannot detect tools. If a tool is manually inserted/removed, correct the state with `GFT T<number>` or `GFT T0` to avoid collisions.
*   **USB File Import:** If the file picker does not appear after inserting a USB, reinsert the drive or upload via the Jobs Manager manually.
*   **Slow Spindle Start:** Allow ~12 seconds for the spindle to reach commanded RPM; consider adding a dwell at job start if your CAM post does not account for spin-up.
