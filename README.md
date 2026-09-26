# 🖥️ 8-Bit CPU — Discrete Logic Computer

A fully functional 8-bit CPU designed in KiCAD and prototyped on breadboards, built from discrete TTL (74LS series) logic ICs with a microcoded control unit stored in parallel EEPROMs. The full design has now been carried from schematics through to custom, routed PCBs for every module.

> Inspired by Ben Eater's 8-bit computer series, this implementation extends the original design with custom modifications in components, organization, and debugging instrumentation.

<p align="center">
  <img src="docs/images/Control Logic 3D.png" width="23%">
  <img src="docs/images/alu-3d.png" width="23%">
  <img src="docs/images/ram-3d.png" width="23%">
  <img src="docs/images/pc-output-3d.png" width="23%">
</p>
<p align="center">
  <img src="docs/images/register-a-3d.png" width="23%">
  <img src="docs/images/register-b-3d.png" width="23%">
  <img src="docs/images/mar-ir-3d.png" width="23%">
  <img src="docs/images/clock-3d.png" width="23%">
</p>

---

## 📸 Overview

The processor implements a classic **von Neumann architecture** with a shared 8-bit data bus, fetch–decode–execute pipeline, and over **80 LEDs** for real-time visualization of control signals, bus traffic, and register contents.

---

## ✨ Features

- **Adjustable clock**: sub-Hz manual stepping up to ~100 kHz continuous operation
- **Manual single-step mode**: debounced pushbutton for step-through debugging
- **Microcoded control unit**: EEPROM-based microcode with up to 8 micro-steps per instruction
- **Rich LED instrumentation**: >80 LEDs tracing every register, bus line, and control signal
- **7-segment display output**: binary-to-decimal decoding via EEPROM lookup table
- **Signed/unsigned display modes**: selectable via mode control input
- **Program Mode / Run Mode**: front-panel DIP switches for manual memory programming

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────┐
│              Control Logic                  │
│           (EEPROM + Step Counter)           │
└────────────────────┬────────────────────────┘
                     │
              ┌──────▼───────┐
              │   8-bit BUS  │
              └──────┬───────┘
       ┌─────────────┼─────────────┐
       │             │             │
  ┌────▼────┐  ┌─────▼─────┐ ┌────▼────┐
  │   PC    │  │  Registers │ │   ALU   │
  │(HCT161) │  │ A / B / IR │ │(LS283×2)│
  └────┬────┘  └─────┬─────┘ └────┬────┘
       │             │             │
  ┌────▼────┐  ┌─────▼─────┐ ┌────▼────┐
  │   MAR   │  │    RAM     │ │ Output  │
  │         │  │ (74189 ×2) │ │Register │
  └─────────┘  └───────────┘ └─────────┘
```

### System Characteristics

| Property       | Value                                    |
| -------------- | ----------------------------------------- |
| Bus Width      | 8-bit shared data bus                    |
| Clock Speed    | Sub-Hz (manual) to ~100 kHz (continuous) |
| Supply Voltage | 5V DC regulated                          |
| RAM            | 128 bytes (74189 × 2)                    |
| Indicators     | >80 LEDs                                 |
| Logic Family   | TTL — 74LS / 74HCT series                |
| PCB Stack-up   | 2-layer (F.Cu / B.Cu), KiCad              |

---

## 🔩 PCB Design

Every module of the CPU has been fully migrated from breadboard schematic to a routed, 2-layer KiCad PCB. Each board is a standalone module that plugs into the shared 8-bit bus, keeping the system modular and easy to debug or re-spin individually.

### Register A
<p align="center">
  <img src="docs/images/register-a-3d.png" width="48%">
  <img src="docs/images/register-a-layout.png" width="48%">
</p>

8-bit accumulator register (`pcb/Register_A/Register_A.kicad_pcb`) — dual 74HCT173 D-type registers cascaded for 8 bits, bus-isolated via 74HCT245 transceiver, with LED taps on every output bit.

### Register B
<p align="center">
  <img src="docs/images/register-b-3d.png" width="48%">
  <img src="docs/images/register-b-layout.png" width="48%">
</p>

8-bit operand register (`pcb/Register_B/Register_B.kicad_pcb`) — same 74HCT173 + 74HCT245 architecture as Register A.

### ALU
<p align="center">
  <img src="docs/images/alu-3d.png" width="48%">
  <img src="docs/images/alu-layout.png" width="48%">
</p>

Dual 74LS283 4-bit adders (`pcb/ALU/ALU.kicad_pcb`) cascaded for 8-bit arithmetic, with a 74LS86 XOR network for two's-complement subtraction and 74LS02/74LS08 gating for the carry/zero flag logic.

### RAM
<p align="center">
  <img src="docs/images/ram-3d.png" width="48%">
  <img src="docs/images/ram-layout.png" width="48%">
</p>

128-byte static RAM (`pcb/RAM/RAM.kicad_pcb`) — dual 74189 RAM chips, 74LS157 address/data muxing, 74LS04 inversion correction, and an 8-position DIP switch bank for manual Program Mode entry.

### MAR + Instruction Register
<p align="center">
  <img src="docs/images/mar-ir-3d.png" width="48%">
  <img src="docs/images/mar-ir-layout.png" width="48%">
</p>

Memory Address Register and Instruction Register combined on one board (`pcb/MAR_Instruction_Register/MAR_Instruction_Register.kicad_pcb`) — dual 74HCT173 registers with manual DIP-switch address entry; the IR splits into opcode (upper nibble, to control logic) and operand (lower nibble, to bus).

### Program Counter + Output Register
<p align="center">
  <img src="docs/images/pc-output-3d.png" width="48%">
  <img src="docs/images/pc-output-layout.png" width="48%">
</p>

Combined PC and display output board (`pcb/Program_Counter_Output_Register/Program_Counter_Output_Register.kicad_pcb`) — 74HCT161 program counter, 74HCT273 output latch, EEPROM-based binary-to-7-segment lookup table, and 74LS76/74LS08 digit-multiplexing driving a 4-digit CA56-12CGWA display.

### Control Logic
<p align="center">
  <img src="docs/images/control-logic-3d.png" width="48%">
  <img src="docs/images/control-logic-layout.png" width="48%">
</p>

Microcode sequencer (`pcb/Control_Logic/Control_Logic.kicad_pcb`) — dual 28C16 EEPROMs storing `(opcode + step + flags) → control signals`, a 74HCT161 step counter, and LED taps on every control line.

### Clock
<p align="center">
  <img src="docs/images/clock-3d.png" width="48%">
  <img src="docs/images/clock-layout.png" width="48%">
</p>

Clock generator (`pcb/Clock/Clock.kicad_pcb`) — triple 555 timer setup (astable for continuous clocking, monostable for debounced single-step), trimmer-potentiometer frequency control, and a run/step mode switch.

---

## 🔧 Modules

### Clock Module

- 555 timer in **astable mode** for continuous operation
- Potentiometer-controlled frequency (sub-Hz to ~100 kHz)
- Debounced pushbutton (555 in monostable mode) for single-step debugging

### Register Modules (A, B, Instruction)

- Built from **74HCT173** 4-bit D-type registers (cascaded pairs for 8-bit)
- All registers bus-isolated via **74LS245** transceivers
- LED taps on all outputs for real-time binary visualization
- IR splits into **opcode (upper nibble)** → control logic and **operand (lower nibble)** → bus

### ALU

- Dual **74LS283** 4-bit adders cascaded for 8-bit arithmetic
- XOR gate network (74LS86) enables **two's complement subtraction** via SUB control line
- Supports: `ADD`, `SUB`, bitwise `AND` / `OR` / `NOT` / `XOR`
- **Carry Flag (CF)**: MSB carry-out
- **Zero Flag (ZF)**: NOR/AND gate detection of all-zero result

### RAM

- Two **74189** static RAMs → 128 bytes total
- **Program Mode**: DIP switches + debounced write button
- **Run Mode**: CPU owns the bus; DIP inputs ignored
- 74LS157 multiplexers handle address/data source selection
- Data inversion corrected with 74LS04 inverters

### Program Counter (PC)

- **74HCT161** 4-bit synchronous counter + **74HCT245** bus transceiver
- Control inputs: `Load` (jump), `Enable` (controlled increment), `Clear/Reset`
- Green LEDs on output lines for address visualization

### Output Module

- **74HC273** octal D-type latch captures 8-bit bus value
- **28C16 EEPROM** as binary-to-7-segment lookup table
- **555 timer** + 74LS76 JK flip-flops + 74ACT139 decoder for digit multiplexing
- **CA56-12CGWA** 4-digit 7-segment display array
- Supports signed and unsigned decimal display modes

### Control Logic

- Two **28C16 EEPROMs** store microcode: `(opcode + step + flags) → control signals`
- **74HCT161** step counter cycles through up to 8 micro-steps per instruction
- Zero and Carry flags feed back into EEPROM address lines for conditional branching (`JZ`, `JC`)
- All control lines LED-tapped for real-time instruction sequencing visibility

---

## 📋 Instruction Set

| Mnemonic | Operation                    |
| -------- | ----------------------------- |
| `LDA`    | Load A register from memory  |
| `STA`    | Store A register to memory   |
| `ADD`    | Add memory value to A        |
| `SUB`    | Subtract memory value from A |
| `JMP`    | Unconditional jump           |
| `JC`     | Jump if carry flag set       |
| `JZ`     | Jump if zero flag set        |
| `OUT`    | Output A register to display |
| `HLT`    | Halt execution               |

---

## 💾 Sample Program

```
LDA 14    ; Load value at address 14 into register A
ADD 15    ; Add value at address 15 to register A
OUT       ; Latch result to 7-segment display
HLT       ; Halt
```

**Expected behavior**: fetches two operands from addresses 14 and 15, sums them in the ALU, and displays the decimal result on the 7-segment display before halting.

---

## 🛠️ Tools & Resources

| Tool                   | Purpose                                                           |
| ----------------------- | ------------------------------------------------------------------ |
| **KiCAD**               | Schematic design & PCB layout                                     |
| **Arduino IDE**         | EEPROM programmer firmware                                        |
| **Arduino Nano**        | Custom EEPROM programmer (with shift registers)                   |
| **LucidChart**          | Block diagram representation                                      |
| **Ben Eater's Series**  | Architecture reference — [eater.net/8bit](https://eater.net/8bit) |

---

## ⚙️ Operating Instructions

1. Connect a regulated **5V DC** supply to the PCB stack
2. Set the mode switch to **Program Mode**
3. Use DIP switches to set address and data values; press **WRITE** to store each byte
4. Switch to **Run Mode**
5. Select **Manual** (single-step) or **Continuous** clock mode
6. Press **RESET** to clear the PC and registers
7. Start the clock and observe execution on LEDs and 7-segment displays

---

## ⚠️ Limitations

- **Memory**: 128 bytes total RAM
- **Speed**: ~100 kHz practical limit (bounded by 74LS propagation delays and board wiring)
- **Debugging**: Longer programs require careful micro-step tracing
- **EEPROM wear**: 28C16 devices have finite write cycles — re-flash microcode judiciously

---

## 🚧 Current Status

- [x] Breadboard prototype functional
- [x] All modules designed in KiCAD
- [x] Arduino Nano EEPROM programmer built and tested
- [x] Decoupling/bypass capacitors added throughout
- [x] PCB layout routing complete for all 8 modules
- [ ] PCB prototype fabrication & testing
- [ ] Full-system bring-up on fabricated boards

---

## 📄 License

This project is open-source and available under the [MIT License](https://github.com/iancho-eng/8bit-CPU/blob/main/LICENSE).

---

## 🙏 Acknowledgements

This project draws heavy inspiration from **Ben Eater's 8-bit computer series**. His tutorials and schematics provided a foundational understanding of CPU architecture. This implementation adapts and extends the design with independent modifications, additional debugging instrumentation, and a complete migration from breadboard to custom PCBs.
