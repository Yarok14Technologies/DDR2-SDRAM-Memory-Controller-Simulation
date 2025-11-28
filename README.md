# 🚀 DDR2 SDRAM Memory Controller Simulation

This repository contains the RTL design, behavioral memory models, and scripts for simulating a **DDR2 SDRAM Controller** using the **Cadence NCSIM** flow. The environment is configured to run file-driven test patterns against a **Micron DDR2 memory model**.

-----

## 📁 Repository Structure

The project follows a standard structure optimized for the Cadence simulation flow:

```
├── .denalirc             # Denali Memory Modeler configuration
├── cds.lib               # Cadence library definition (includes OSU 0.18um std cells)
├── hdl.var               # HDL compilation variables
├── makefile              # Automation script for the Cadence NCSIM flow
├── README.markdown       # This file
├── design/               # Core design, logic, and I/O models
│   ├── ddr2_controller.v
│   ├── ddr2_init_engine.v
│   ├── ddr2_ring_buffer8.v
│   ├── FIFO.v
│   ├── mt47h32m16_37e.v
│   ├── process_logic.v
│   ├── SSTL18DDR2.v
│   ├── SSTL18DDR2DIFF.v
│   └── SSTL18DDR2INTERFACE_RTL.v
├── scripts/              # Simulation control scripts and waveform configuration
│   ├── ddr2sdram.sv
│   ├── runscript.tcl
│   └── runscript_nogui.tcl
└── tb/                   # Testbench files
    └── tb.v
```

| Directory | Content | Description |
| :--- | :--- | :--- |
| **`./`** | `makefile`, `cds.lib`, `.denalirc`, `hdl.var` | Top-level configuration files and build scripts. |
| **`./design`** | All RTL files (`.v`) and I/O buffer models. | The core controller logic and memory interface components. |
| **`./tb`** | Testbench file (`tb.v`). | The top-level test harness for the controller. |
| **`./scripts`** | Tcl and SimVision command files. | Scripts for automating simulation runs and configuring the waveform viewer. |
| **`./work`** | Cadence work library. | Automatically generated compilation and elaboration artifacts. |

-----

## ⚙️ Design & Models (`./design`)

The design implements a functional DDR2 memory controller and includes necessary behavioral models for simulation:

### Core Controller Logic

| Filename | Component/Function |
| :--- | :--- |
| **`ddr2_controller.v`** | **Top-Level Module** - Connects the user interface to the memory core. |
| **`process_logic.v`** | **Command State Machine** - Converts user requests (Read/Write) into DDR2 protocol commands (Activate, Precharge, etc.). |
| **`ddr2_init_engine.v`** | **Initialization Engine** - Handles the power-up sequence and Mode Register programming. |
| **`FIFO.v`** | **Command/Data Queues** - Generic single-clock FIFO for internal command and data buffering. |
| **`ddr2_ring_buffer8.v`** | **Data Reordering** - 8-deep ring buffer for handling read data alignment from the DDR interface. |

### Memory and I/O Models

| Filename | Component/Function |
| :--- | :--- |
| **`mt47h32m16_37e.v`** | **DDR2 Memory Model** - Behavioral model of the **Micron MT47H32M16-37E** device (provided by Denali). |
| **`SSTL18DDR2INTERFACE_RTL.v`** | **Physical Interface** - Instantiates the I/O buffers for connection to the memory model. |
| **`SSTL18DDR2.v`** / **`SSTL18DDR2DIFF.v`** | **SSTL-18 I/O Models** - Simple simulation models for single-ended and differential I/O pads. |

-----

## 🧪 Testbench Requirements (`./tb/tb.v`)

The `tb.v` testbench is designed to be driven by a user-supplied command file, requiring a special interface to load this data during simulation.

### 1\. External File I/O (PLI Requirement)

  * **PLI Library**: The testbench uses the **Chris Spear's file I/O PLI**. You must have the compiled shared library **`fileio.so`** in your working directory.
  * **Test Pattern File**: Commands are read from a file named **`INPUT_FILE_NAME`** (as configured internally in `tb.v`).

### 2\. Test Pattern File Format

The test pattern file is a space-separated ASCII file with five columns:

| Column | Data Type | Description |
| :--- | :--- | :--- |
| `WaitForCycles` | `[decimal]` | Number of clock cycles to wait before executing the next command. |
| `Cmd` | `[decimal]` | The command ID to be issued to the controller. |
| `Address` | `[hex]` | The 25-bit address for the operation. |
| `Data` | `[hex]` | The 16-bit write data (ignored for Read commands). |
| `Read` | `[decimal]` | Flag (1 or 0) indicating if the command expects a read response. |

-----

## 💻 Simulation Usage

The **`makefile`** automates the entire Cadence NCSIM flow (compilation, elaboration, simulation). The assumed top module for elaboration and simulation is **`tb`**.

### 🛠️ Critical Setup Step: Linking the PLI

Because the testbench uses a PLI function to read the test pattern file, you **must** ensure the `makefile` includes the switch to link the PLI library during the elaboration phase (`ncelab`):

```makefile
# This line (or equivalent) must be added to your ncelab switches:
NCELAB_SWITCHES += -loadpli1 $(PWD)/fileio.so:bstrap
```

### ▶️ Build and Run Commands

| Command | Action | Description |
| :--- | :--- | :--- |
| `make` | **Compile & Elaborate** | Runs `ncvlog` (compilation) on all design/testbench files and `ncelab` (elaboration) on the top module (`tb`), linking the PLI. |
| `make simg` | **Run GUI Simulation** | Executes `ncsim`, launches the **SimVision GUI**, and loads the pre-configured waveform setup from `./scripts/ddr2sdram.sv`. |
| `make sim` | **Run Console Simulation** | Executes `ncsim` in headless mode using `./scripts/runscript_nogui.tcl`. Suitable for regression or long runs. |
| `make clean` | **Clean Project** | Removes all generated files, logs, and the `work` directory for a fresh start. |

### Simulation Scripts (`./scripts`)

  * **`runscript.tcl`**: Tcl script for GUI simulation (`make simg`). Creates a trace database (`.shm`) and launches SimVision.
  * **`runscript_nogui.tcl`**: Tcl script for console simulation (`make sim`). Creates a trace database and exits.
  * **`ddr2sdram.sv`**: **SimVision saved session file**. Defines which signals are displayed, their format, and initial cursor position for the waveform viewer.

Enjoy your simulation\!
