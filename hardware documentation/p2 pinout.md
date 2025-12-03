## REFERENCE: Mini-DIN-8 to Wire Color Mapping (DO NOT CHANGE)

**Standard Mini-DIN-8 Pin → Wire Color:**
- Pin 1 = Brown wire = GND
- Pin 2 = Red wire = GND
- Pin 3 = Orange wire = MISO
- Pin 4 = Yellow wire = SCK
- Pin 5 = Green wire = CS
- Pin 6 = Blue wire = +5V
- Pin 7 = Purple wire = MOSI
- Pin 8 = Grey wire = +12V
- Metal Shell = Black wire = GND

**synchronous serial protocol** similar to SPI:

| Signal | Channel | Pin | Direction | Description |
|--------|---------|-----|-----------|-------------|
| **DATA** | Channel 0 | MISO | Machine → CB-1 | Serial data output from machine |
| **CLOCK** | Channel 5 | SCK | CB-1 → Machine | Clock signal |
| **FRAME** | Channel 3 | CS/Frame | CB-1 → Machine | Message framing / chip select |
| Power | Channel 4 | VCC | - | Always HIGH (power or pull-up) |

### Microcontrollers
- **UPD78212** (CB-1 controller) - 8-bit µC with CSI serial interface
- **UPD75004** (Machine controller) - Main knitting machine processor

### Pin 3 (Orange wire) - MOSI/MISO (Bidirectional Data)
- 1nF capacitor to GND (noise filtering)
- → R9 (1kΩ series resistor - current limiting)
- → R11 (1kΩ pull-up to +5V)
- → JW10 (jumper wire)
- → µPD75004CU-291 Pin 13

### Pin 4 (Yellow wire) - SCK or CS (Input to Machine)
- → Q2 Pin 3 (IN) - DTC124EF digital transistor/inverter
- Q2 Pin 2 (OUT) → JW9 → 10kΩ pull-up to +5V → µPD75004 Pin 14
- Q2 Pin 1 (GND) → Ground
- **Function**: Input from CB-1, inverted by Q2, likely SCK (Clock)

### Pin 5 (Green wire) - CS or SCK (Input to Machine)
- → Q1 Pin 3 (IN) - DTC124EF digital transistor/inverter
- Q1 Pin 2 (OUT) → JW7 → JW12 → µPD75004 Pin 6
- Q1 Pin 1 (GND) → Ground
- **Function**: Input from CB-1, inverted by Q1, likely CS (Chip Select)

### Pin 7 (Purple wire) - MISO/MOSI (Bidirectional Data)
- 1nF capacitor to GND (noise filtering)
- → R8 (1kΩ series resistor - current limiting)
- → R10 (1kΩ pull-up to +5V)
- → JW11 (jumper wire)
- → 10kΩ pull-up to +5V (additional pull-up)
- → µPD75004CU-291 Pin 12

## Circuit Analysis

### Protection & Signal Conditioning:
- **1nF capacitors** on Orange/Purple wires = High-frequency noise filtering
- **1kΩ series resistors** = Current limiting/ESD protection
- **Pull-up resistors** = Ensure defined logic levels
- **DTC124EF inverters** on Yellow/Green = Logic level translation with inversion

### Signal Direction:
- **Inputs to Machine** (from CB-1): Yellow (Pin 4), Green (Pin 5) - via inverters
- **Bidirectional** (Machine ↔ CB-1): Orange (Pin 3), Purple (Pin 7) - with pull-ups

## ✅ Signal Assignment VERIFIED

**Signal assignments confirmed based on Saleae Logic 2 protocol analysis:**

From `test files/KH970_Protocol_Analysis.md` and `digital.csv` capture:
- **Channel 0** = DATA (MISO) - Machine → CB-1 data output
- **Channel 3** = CHIP SELECT / FRAME - CB-1 → Machine control
- **Channel 5** = CLOCK (SCK) - CB-1 → Machine clock (10 kHz)
- **Channel 1** = Likely MOSI - CB-1 → Machine commands (high activity)

### Pin 3 (Orange wire) - **MISO** ✅
- µPD75004CU-291 Pin 13
- **Function**: Machine sends status data to CB-1 (Channel 0 in captures)
- Bidirectional with pull-ups (data output from machine)

### Pin 4 (Yellow wire) - **SCK** (Clock) ✅
- µPD75004CU-291 Pin 14 via Q2 inverter
- **Function**: CB-1 generates clock signal, inverted by Q2 (Channel 5 in captures)
- Input only, 10 kHz, inverted logic

### Pin 5 (Green wire) - **CS** (Chip Select) ✅
- µPD75004CU-291 Pin 6 via Q1 inverter
- **Function**: CB-1 controls message framing (Channel 3 in captures)
- Input only, inverted logic

### Pin 7 (Purple wire) - **MOSI** ✅
- µPD75004CU-291 Pin 12
- **Function**: CB-1 sends commands to machine (likely Channel 1 in captures)
- Bidirectional with pull-ups (data input to machine)

**Protocol Characteristics:**
- Clock: **10 kHz**
- Data: MSB first, 8-bit bytes
- Sampling: Rising edge of clock
- Message gap: >10ms between messages
- Control signals are INVERTED by DTC124EF transistors on machine side
- Bidirectional SPI: CB-1 is MASTER (generates clock), Machine is SLAVE

## Notes

- P2 is a **MALE** Mini-DIN-8 connector on the KH970 board
- Interface board needs **FEMALE** Mini-DIN-8 connector
- Ground appears on pins 1, 2, and shield (good for reliability)
- Two separate power rails: +5V and +12V
- the solenoids are actually rated as 6V solenoids and they work well down to 7V when the voltage drop across the regulator is too much causing the mcu to crash. they will probably work just as well on 5V.
-The CB-1 is marked as being powered by 9.5V
-The CB-1 power supply AD-400 is marked as 9.5V but outputs 12V
-voltage drop on these old basic power supplies can be substantial under load
-Machine works perfectly well using a modern MeanWell GST18A09-P1J 9V 2A but polarity **MUST** be reversed
-The CB-1 requires a positive ring and negative centre pin.
- 

## ⚠️ CRITICAL: Power Requirements

- **Pin 8 (+12V) MUST be connected** - The KH970 has only ONE connector (P2) for both signal AND power
- The machine cannot function without power supplied through Pin 8 (Grey wire)
- Interface board MUST provide +12V on Pin 8 to power the KH970 solenoids
- The interface board is the ONLY power source for the KH970 when connected

- **Pin 6 (+5V) MUST be connected**
