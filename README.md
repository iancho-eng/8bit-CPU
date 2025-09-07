# 8bit-CPU
Discrete-logic 8-bit CPU with schematics and report (PCB in progress)

This project is a fully functional 8-bit CPU built from a combination of 74LS-series TTL chips and 74HCT CMOS chips, designed and documented as both a learning platform and a demonstration of classic computer architecture. Inspired by Ben Eater’s 8-bit computer, it implements the fetch–decode–execute cycle with a microcoded control unit, LED instrumentation, and 7-segment output displays. The system was first prototyped on breadboards, then migrated into KiCad schematics and PCB layouts, with extensive documentation provided in the report and supporting diagrams.

## 📖 Documentation
- [Full Project Report (PDF)] [8-bit CPU documentation (final) (PDF)](8-bit%20CPU%20documentation%20%28final%29.pdf)
- [Block Diagram](docs/block-diagram.png)  
- [Complete Schematic](docs/complete-schematic.png)
- [Individual Schematics](docs/All-schematics.pdf)

## 🔧 System Features
- 8-bit data bus with tri-state isolation
- Microcoded control unit (EEPROM + step counter)
- Arithmetic Logic Unit supporting ADD, SUB, AND, OR, XOR
- 128 bytes of RAM (74189 × pairs)
- LED and 7-segment output instrumentation
- Adjustable clock (manual / automatic)

## 🗂 Repository Layout
- `docs/` → diagrams and schematics (PNG)
- `schematics/` → KiCad schematic files
- `code/` → Arduino EEPROM programmer + sample programs  
- `pcb/` → PCB layouts & gerbers  (in progress)

## 🚀 Resources
- [Ben Eater’s 8-bit computer tutorials](https://eater.net/8bit)  
- [KiCad EDA](https://www.kicad.org/)  
- [Arduino IDE](https://www.arduino.cc/en/software)

## 📸 Preview
- [Block Diagram](docs/block-diagram.png)
