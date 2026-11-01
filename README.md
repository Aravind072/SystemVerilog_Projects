# Synchronous FIFO SystemVerilog Testbench Architecture

This repository contains a modular, scalable, transaction-level **SystemVerilog Testbench Architecture** designed to verify a Synchronous FIFO with a depth of 16 and a data width of 8 bits. 

The environment isolates stimulus generation, bus driving, activity monitoring, and automated scoreboard checking using object-oriented verification concepts.

## 🏗️ Testbench Architecture Block Diagram

```text
+-------------------------------------------------------------+

|                         TESTBENCH                           |
|  +------------------+                 +------------------+  |
|  |       TEST       |                 |    SCOREBOARD    |  |
|  | (Directs Stimulus|                 | (Checks Output   |  |
|  +--------+---------+                 |  Against Ideal)  |  |
|           |                           +--------^---------+  |
|  +--------v---------+                          |            |
|  |    GENERATOR     |                          |            |
|  | (Creates Packets)|                          |            |
|  +--------+---------+                          |            |
|           | (Mailbox)                          | (Mailbox)  |
|  +--------v---------+                 +--------+---------+  |
|  |     DRIVER       +=================>     MONITOR      |  |
|  | (Drives Signals) |    INTERFACE    | (Samples Signals)|  |
+--+--------+---------+========+========+--------^---------+--+

            |                  |                 |
   =========v==================v=================v========= (Virtual Interface)

            |                  |                 |
   +--------v------------------v-----------------+--------+

   |                     DUT (FIFO)                       |
   +------------------------------------------------------+
```

## 🛠️ Key Design Specifications
* **FIFO Depth:** 16 positions
* **Data Width:** 8-bit broad data bus (`[7:0]`)
* **Clock Domain:** Single, synchronous clock domain
* **Reset Type:** Active-low asynchronous/synchronous reset (`rst_n`)

## 🚀 Verification Components

The architecture is divided into the following functional elements:
* **`fifo_if` (Interface):** Encapsulates the FIFO bus signals, establishing dedicated `clocking` blocks and `modports` to prevent race conditions during logic simulation.
* **`fifo_transaction` (Transaction Object):** Formulates random constraints (`dist` loops) to generate valid read/write operations and hold structural parameters.
* **`fifo_generator` (Generator):** Constrains and randomizes raw test packets, handling object deployment before dispatching to the Driver.
* **`fifo_driver` (Driver):** Manages physical pin interactions, pulling transactions from the mailbox and enforcing proper hardware timing intervals.
* **`fifo_monitor` (Monitor):** Independently monitors the system interface pins, packaging interface behavior back into clean transaction objects.
* **`fifo_scoreboard` (Scoreboard):** Leverages an internal SystemVerilog dynamic queue (`bit [7:0] ideal_fifo[$]`) as a golden model to dynamically validate read/write outputs alongside `full` and `empty` flag accuracy.
* **`fifo_environment` (Environment):** Orchestrates component hooks, links transaction mailboxes, and governs execution ordering (`pre_test`, `test`, `post_test`).

## 💻 Simulation & Execution

### Prerequisites
Ensure you have a modern SystemVerilog-compliant simulator installed (e.g., Siemens Questa/Modelsim, Synopsys VCS, Cadence Xcelium, or EDA Playground access).

### Compilation Commands (Example using Questa/Modelsim)
```bash
# Compile the interface, DUT, and top module code
vlog -sv sync_fifo.v tb_top.sv

# Run the simulation with text output enabled
vsim -c tb_top -do "run -all; quit"
```

## 📊 Sample Testbench Output
Upon running, the environment prints live transaction tracking to the console:
```text
[15] [DRIVER] Reset Deasserted.
[25] MONITOR: WR=1 | RD=0 | DataIn=0x4A | DataOut=0xXX | Full=0 | Empty=1
 -> [SCB MODEL] Pushed: 0x4A (Size: 1)
[35] MONITOR: WR=1 | RD=0 | DataIn=0xF2 | DataOut=0xXX | Full=0 | Empty=0
 -> [SCB MODEL] Pushed: 0xF2 (Size: 2)
[45] MONITOR: WR=0 | RD=1 | DataIn=0x00 | DataOut=0x4A | Full=0 | Empty=0
 -> [PASS] Matched Data: 0x4A

==================================
     SIMULATION REPORT            
==================================
 Total Matches    : 100
 Total Mismatches : 0
==================================
```
