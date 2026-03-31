# AGV-PLC-TwinCAT

**PLC Task 3 – ATM700 | University West – Department of Engineering Science**

A TwinCAT 3 PLC project implementing a full operator control system for a Virtual Automatic Guided Vehicle (AGV). The project covers PLC programming, inter-instance communication, and HMI design using IEC 61131-3 structured programming principles.

---

## Project Overview

This project simulates an industrial AGV control system where an operator can:

- Select a target position (A, B, C, D, or H) via an HMI
- Start the AGV movement with a Start button
- Stop the AGV immediately at any time with a Stop button
- Reset the system after a stop, which sends the AGV back to its HOME position
- Monitor the AGV state through live HMI indicators (Moving, Ready, Stop, Position display)

The control logic is split across two TwinCAT PLC instances inside one solution:
- **PLCSim** — runs the provided Virtual AGV (`AGV.TcPOU`) and its simulator visualization
- **PLCStudent** — runs the student-written operator control (`RunStudent`) and HMI (`Visualization`)

---

## Requirements

| Tool | Version |
|---|---|
| TwinCAT 3 XAE (Engineering) | 3.1 Build 4024 or later |
| TwinCAT 3 XAR (Runtime) | Included with XAE |
| Windows OS | Windows 10 or later (64-bit) |

---

## Directory Structure

```
TwinCAT Virtual AGV v2_0/
│
├── TwinCAT Virtual AGV.sln                  # Visual Studio solution file
│
└── TwinCAT Virtual AGV/
    ├── TwinCAT Virtual AGV.tsproj           # TwinCAT project file
    ├── TrialLicense.tclrs                   # Trial license file
    │
    ├── PLCSim/                              # PLC instance: Virtual AGV (provided)
    │   ├── PLCSim.plcproj                   # PLC project file
    │   ├── PlcTask.TcTTO                    # Task configuration
    │   ├── Visualization Manager.TcVMO      # Visualization manager
    │   ├── GlobalTextList.TcGTLO
    │   │
    │   ├── GVLs/
    │   │   └── GVL.TcGVL                    # AGV interface signals (I/O mapped)
    │   │
    │   ├── POUs/
    │   │   ├── AGV.TcPOU                    # Provided AGV function block (do not modify)
    │   │   ├── MAIN.TcPOU                   # Calls fbAGV, entry point for PLCSim
    │   │   └── Process.TcPOU               # AGV internal process logic
    │   │
    │   └── VISUs/
    │       └── AGVSimulator.TcVIS           # AGV simulator visualization (provided)
    │
    └── PLCStudent/                          # PLC instance: Student control & HMI
        ├── PLCStudent.plcproj               # PLC project file
        ├── PlcTask.TcTTO                    # Task configuration
        ├── Visualization Manager.TcVMO      # Visualization manager
        ├── GlobalTextList.TcGTLO
        │
        ├── GVLs/
        │   ├── GVL.TcGVL                    # AGV interface signals (linked to PLCSim)
        │   └── GVL_CTRL.TcGVL              # Operator control variables (AT %M* mapped)
        │
        ├── POUs/
        │   ├── MAIN.TcPOU                   # Entry point: calls RunStudent and FB_AGVComm
        │   ├── FB_AGVComm.TcPOU             # FB: AGV communication interface (reusable)
        │   └── RunStudent.TcPOU             # PRG: operator control logic, HMI, Stop/Reset
        │
        └── VISUs/
            └── Visualization.TcVIS          # Operator HMI (student-designed)
```

---

## HMI Controls & Indicators

| Element | Type | Description |
|---|---|---|
| Target Selection | Drop-down / text input | Select destination: A, B, C, D, or H |
| Start Button | Button | Sends AGV to selected target (ignored during movement) |
| Stop Button | Button | Immediately stops the AGV at any time |
| Reset Button | Button | Sends AGV to HOME, re-enables control after a stop |
| Moving Indication | Blinking indicator | ON when AGV is in motion |
| Ready Indication | Indicator | ON when a new target can be selected and started |
| Stop Indication | Indicator | ON from emergency stop until HOME is reached |
| Position Display | Text / indicator | Shows current position (A/B/C/D/H) when standing still |

---

## How to Run the Project

### 1. Open the solution

1. Launch **TwinCAT 3 XAE** (Visual Studio shell)
2. Open `TwinCAT Virtual AGV.sln` from the project folder
3. The solution contains both **PLCSim** and **PLCStudent** instances

### 2. Activate and start the runtime

1. Go to **TwinCAT → Activate Configuration** — this loads both PLC instances
2. Confirm to set the system to **Run** mode when prompted
3. For each PLC instance (PLCSim and PLCStudent):
   - Go to **PLC → Login**
   - Then **PLC → Start**

### 3. Open the AGV Simulator

1. In the PLCSim project, navigate to **VISUs → AGVSimulator**
2. Open it to confirm the virtual AGV is running and responsive

### 4. Open the Operator HMI

1. In the PLCStudent project, navigate to **VISUs → Visualization**
2. Open it to access the student operator HMI
3. Use the HMI to control the AGV:
   - Select a target → Press **Start**
   - Press **Stop** at any time to halt the AGV mid-movement
   - Press **Reset** to return the AGV to HOME and re-enable control

> **Note:** The operator HMI (`Visualization.TcVIS`) is separate from the AGV simulator. Do not operate the AGV directly from the AGVSimulator screen.

---

## Program Architecture

```
PLCSim
└── MAIN
    └── calls AGV (FB)          → Virtual AGV logic (provided, do not modify)
        └── Process             → Internal AGV process handling

PLCStudent
└── MAIN
    ├── calls FB_AGVComm (FB)   → Handles all communication with the AGV FB in PLCSim
    └── calls RunStudent (PRG)  → Operator control: Start/Stop/Reset logic + HMI outputs
```

Key design decisions:
- **`FB_AGVComm`** is implemented as a reusable function block — AGV communication is encapsulated here as required by the spec
- **`RunStudent`** contains all operator-facing logic: target locking, stop latch, reset sequencing, and HMI indicator control
- **Target locking** — once Start is pressed, target changes are ignored until the AGV reaches its destination
- **Stop latch** — after a Stop, new targets and Start commands are blocked until Reset completes
- **Reset sequence** — Reset sends the AGV to HOME; control only re-enables after HOME is confirmed reached
- **Position Display** — shown only when the AGV is stationary at a known position, off during movement

---

## Behaviour Summary

| Situation | System Response |
|---|---|
| Target changed during movement | Ignored — current target is locked |
| Start pressed during movement | Ignored |
| Stop pressed during movement | AGV halts immediately, Stop indication turns ON |
| Reset pressed after Stop | AGV moves to HOME, Stop indication stays ON until HOME reached |
| AGV at HOME after Reset | Ready indication turns ON, system accepts new target |
| AGV moving | Moving indication ON, Position display OFF |
| AGV stationary at position | Position display shows current position (A/B/C/D/H) |

---

## Author

**Tejashwi Jagadish**
University West – Department of Engineering Science

