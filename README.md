# 🖥️ 8-Bit CPU — Discrete Logic Computer

A fully functional 8-bit CPU designed in KiCAD and prototyped on breadboards, built from discrete TTL (74LS series) logic ICs with a microcoded control unit stored in parallel EEPROMs.

> Inspired by Ben Eater's 8-bit computer series, this implementation extends the original design with custom modifications in components, organization, and debugging instrumentation.

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

| Property        | Value                                     |
|-----------------|-------------------------------------------|
| Bus Width       | 8-bit shared data bus                     |
| Clock Speed     | Sub-Hz (manual) to ~100 kHz (continuous)  |
| Supply Voltage  | 5V DC regulated                           |
| RAM             | 128 bytes (74189 × 2)                     |
| Indicators      | >80 LEDs                                  |
| Logic Family    | TTL — 74LS / 74HCT series                 |

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

| Mnemonic | Operation                        |
|----------|----------------------------------|
| `LDA`    | Load A register from memory      |
| `STA`    | Store A register to memory       |
| `ADD`    | Add memory value to A            |
| `SUB`    | Subtract memory value from A     |
| `JMP`    | Unconditional jump               |
| `JC`     | Jump if carry flag set           |
| `JZ`     | Jump if zero flag set            |
| `OUT`    | Output A register to display     |
| `HLT`    | Halt execution                   |

---

## 💾 Sample Program

```asm
LDA 14    ; Load value at address 14 into register A
ADD 15    ; Add value at address 15 to register A
OUT       ; Latch result to 7-segment display
HLT       ; Halt
```

**Expected behavior**: fetches two operands from addresses 14 and 15, sums them in the ALU, and displays the decimal result on the 7-segment display before halting.

---

## 🛠️ Tools & Resources

| Tool                    | Purpose                                          |
|-------------------------|--------------------------------------------------|
| **KiCAD**               | Schematic design & PCB layout                    |
| **Arduino IDE**         | EEPROM programmer firmware                       |
| **Arduino Nano**        | Custom EEPROM programmer (with shift registers)  |
| **LucidChart**          | Block diagram representation                     |
| **Ben Eater's Series**  | Architecture reference — [eater.net/8bit](https://eater.net/8bit) |

---

## ⚙️ Operating Instructions

1. Connect a regulated **5V DC** supply to the PCB
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
- [ ] PCB layout routing in progress
- [ ] PCB prototype fabrication & testing

---

## 📄 License

This project is open-source and available under the [MIT License](LICENSE).

---

## 🙏 Acknowledgements

This project draws heavy inspiration from **Ben Eater's 8-bit computer series**. His tutorials and schematics provided a foundational understanding of CPU architecture. This implementation adapts and extends the design with independent modifications, additional debugging instrumentation, and ongoing PCB migration work.
