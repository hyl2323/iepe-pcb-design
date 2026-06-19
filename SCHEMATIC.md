# IEPE Microphone Signal Conditioning Board
## Single-Channel, Gain 100x, 20Hz HPF, 20kHz LPF, 5V Input

---

## Circuit Block Diagram

```
[5V Input] --> [NCP1117-3.3] --> [+3.3V]
                |
                +-> [NCP1117-2.5] --> [+2.5V (Bias)]
                     |
                     [R1: 2.2kΩ]
                     |
[IEPE Microphone] <--+
      |
      [C5: 10µF AC Coupling]
      |
    [R2: 10kΩ]
      |
[OPA2333-A: Buffer + HPF 20Hz]
      |
[OPA2333-B: Gain 100x Amplifier]
      |
  [LPF 20kHz: R8=1.6kΩ, C8=5nF]
      |
   [C9: 10µF Output Coupling]
      |
   [BNC Output]
```

---

## Detailed Schematic

### Power Supply Section

```
+5V Input
    |
   [TVS Diode]
    |
   [C_in: 10µF Electrolytic]
    |
    +--[U2: NCP1117-3.3]--> +3.3V
    |   [GND]
    |   [C1: 10µF + C2: 0.1µF]
    |
    +--[U3: NCP1117-2.5]--> +2.5V (Bias Reference)
        [GND]
        [C3: 10µF + C4: 0.1µF]
```

### IEPE Bias Network

```
+2.5V
  |
 [R1: 2.2kΩ, 1% precision]
  |
  +--[J1: BNC Connector (Hot pin)]
     [IEPE Microphone]
     [Shield --> GND]
```

**Bias Current**: I = (2.5V) / 2.2kΩ ≈ 1.14 mA

### Signal Path: Stage 1 - Buffer & High-Pass Filter (20Hz)

```
[BNC Input]
    |
   [C5: 10µF Film Cap]
    |
   [R2: 10kΩ to GND]
    |
    +--> [OPA2333-A Pin 3 (Non-inverting input)]
         [Pin 2 (Inverting)] --[R4: 10kΩ feedback]--[Pin 1 Output]
         [Pin 4: -GND]
         [Pin 8: +3.3V]
         |
         [High-Pass Filter]
         [R3: 1.6kΩ]
         [C6: 5µF]
         |
         [Output to Stage 2]

fc (HPF) = 1 / (2π * R3 * C6) = 1 / (2π * 1.6k * 5µF) ≈ 20Hz
```

### Signal Path: Stage 2 - Non-Inverting Amplifier (Gain = 100x)

```
[Input from HPF]
    |
    +--> [OPA2333-B Pin 5 (Non-inverting input)]
         [Pin 6 (Inverting)] <--[R7: 990kΩ feedback]--[Pin 7 Output]
         |                   |
         [R6: 10kΩ to GND]   |
         |                   |
         [C7: 10µF bypass]   |
         |                   |

Gain = 1 + (R7 / R6) = 1 + (990k / 10k) = 100x = 40dB
```

### Output: Anti-Aliasing Low-Pass Filter (20kHz)

```
[Amplifier Output]
    |
   [R8: 1.6kΩ]
    |
    +--[LPF Network]
    |  [C8: 5nF to GND]
    |
   [C9: 10µF Film Cap (Output Coupling)]
    |
   [J2: BNC Connector]
    |
   [Load Resistor: ~10kΩ (typical)]
    |
   GND

fc (LPF) = 1 / (2π * R8 * C8) = 1 / (2π * 1.6k * 5nF) ≈ 20kHz
```

---

## Component List

### Power Supply & Regulation
| Reference | Value | Package | Part # | Notes |
|-----------|-------|---------|--------|-------|
| U2 | NCP1117-3.3 | TO-220 or SOT-223 | NCP1117ST33T100 | 3.3V LDO Regulator |
| U3 | NCP1117-2.5 | TO-220 or SOT-223 | NCP1117ST25T100 | 2.5V LDO Regulator |
| C1, C3 | 10µF | 1210 or Radial | EEVFK1E100SR | Low-ESR Electrolytic |
| C2, C4 | 0.1µF | 0805 or 1206 | GRM31CR61A104KA19L | Ceramic X7R |
| D1 | TVS Diode | SOD-123 | SMBJ5.0A | 5V Input Protection |

### IEPE Bias Network
| Reference | Value | Package | Tolerance | Notes |
|-----------|-------|---------|-----------|-------|
| R1 | 2.2kΩ | 0805 | 1% | Constant Current Source |

### AC Coupling & Impedance
| Reference | Value | Package | Notes |
|-----------|-------|---------|-------|
| C5 | 10µF | 1206 Film | Polyester or PP, 50V |
| R2 | 10kΩ | 0805 | 1% Precision |

### Stage 1: Buffer & High-Pass Filter
| Reference | Value | Package | Tolerance | Notes |
|-----------|-------|---------|-----------|-------|
| U1 | OPA2333 | SOIC-8 | - | Dual Op-Amp, Low-Noise |
| R3 | 1.6kΩ | 0805 | 1% | HPF Resistor |
| C6 | 5µF | 1206 Film | - | HPF Capacitor |
| R4 | 10kΩ | 0805 | 1% | Feedback (Unity Gain) |
| C7 | 10µF | 1206 | - | Bypass Capacitor |

### Stage 2: Gain Amplifier (100x)
| Reference | Value | Package | Tolerance | Notes |
|-----------|-------|---------|-----------|-------|
| R6 | 10kΩ | 0805 | 0.1% | **CRITICAL: 5ppm/°C** |
| R7 | 990kΩ | 0805 | 0.1% | **CRITICAL: 5ppm/°C** |

### Anti-Aliasing Low-Pass Filter
| Reference | Value | Package | Notes |
|-----------|-------|---------|-------|
| R8 | 1.6kΩ | 0805 | 1% Precision |
| C8 | 5nF | 0805 | C0G/NP0 Ceramic |

### Output Coupling
| Reference | Value | Package | Notes |
|-----------|-------|---------|-------|
| C9 | 10µF | 1206 Film | Polyester or PP, 50V |

### Connectors
| Reference | Value | Type | Notes |
|-----------|-------|------|-------|
| J1 | BNC | Chassis Mount | Microphone Input |
| J2 | BNC | Chassis Mount | Signal Output |
| J3 | 2-pin Header | 0.1" Pitch | 5V Power Input |

---

## Electrical Specifications

| Parameter | Value | Notes |
|-----------|-------|-------|
| **Input Supply Voltage** | 5V ± 5% | |
| **Input Supply Current** | ~100mA | Peak |
| **Voltage Gain** | 100x (40dB) | Non-inverting amplifier |
| **High-Pass Frequency** | 20Hz (-3dB) | |
| **Low-Pass Frequency** | 20kHz (-3dB) | |
| **Input Impedance** | ~10kΩ | Dominated by R2 |
| **Output Impedance** | ~1kΩ | From OPA2333 |
| **Noise Floor** | <15µV RMS | 20Hz - 20kHz |
| **CMRR** | >80dB | Common-Mode Rejection |
| **THD** | <0.1% | Total Harmonic Distortion |
| **Output Swing** | 0 - 3.3V | Rail-to-rail |

---

## Frequency Response

```
        |\n        | \\_____ (Flat Gain Region, 40dB)
        |      \\
Gain    |       \\___
(dB)    |          \\___
        |              \\___
        |                  \\_____ LPF -20dB/decade
        |__________________________|____________________
        1Hz      20Hz          20kHz      100kHz
        ← HPF → ← Flat Gain → ← LPF →
```

---

## PCB Layout Guidelines

### Board Dimensions: 97mm × 57mm (2-Layer)

1. **Layer 1 (Top):**
   - Component placement
   - Signal traces (short)
   - Power distribution

2. **Layer 2 (Bottom):**
   - Continuous ground plane
   - Return paths
   - High-current traces

3. **Trace Width:**
   - Signal traces: 0.25mm
   - Power traces: 0.5mm
   - Ground traces: 0.5mm

4. **Via Spacing:** 1.27mm minimum

5. **Component Placement:**
   - U1 (Op-amp) near input connector
   - Bypass capacitors within 5mm of IC pins
   - R7 (gain resistor) close to U1B
   - Output BNC connector at board edge

6. **Ground:**
   - Star-point connection at input
   - Multiple vias to ground plane
   - Separate analog and digital grounds if applicable

7. **Shielding:**
   - Guard trace around input signal path
   - BNC shield to ground plane

---

## Assembly Instructions

1. **Solder passive components first** (resistors, capacitors)
2. **Install IC sockets** (if desired) or solder directly
3. **Solder voltage regulators** (U2, U3)
4. **Solder op-amp IC** (U1)
5. **Install connectors** (J1, J2, J3)
6. **Apply solder mask** to prevent shorts
7. **Test continuity** before power-up

---

## Testing Checklist

- [ ] Verify +3.3V at U1 pin 8
- [ ] Verify +2.5V at bias network
- [ ] Verify bias current (~1.1mA) with multimeter
- [ ] Connect microphone and apply 1kHz test signal
- [ ] Verify gain ≈ 40dB at 1kHz
- [ ] Check output frequency response (20Hz - 20kHz)
- [ ] Verify noise < 15µV RMS
- [ ] Check THD < 0.1% at 1V output

---

## Notes

- All resistors should be thin-film or metal-film types (0.1% tolerance for gain resistors)
- OPA2333 is chosen for low noise (10 nV/√Hz) and single-supply operation
- Keep supply traces wide and short to minimize noise
- Use ferrite bead on input 5V supply if EMI is a concern
