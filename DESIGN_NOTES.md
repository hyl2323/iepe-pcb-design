# Design Notes - IEPE Microphone Signal Conditioning

---

## Design Decisions & Rationale

### 1. Op-Amp Selection: OPA2333

**Why OPA2333?**
- **Low Noise:** 10 nV/√Hz @ 1kHz (critical for microphone circuits)
- **Rail-to-Rail Output:** Can use full 3.3V supply
- **Single-Supply Operation:** Works directly from 3.3V
- **Low Cost:** ~$0.89 in single quantities
- **SOIC-8 Package:** Easy hand soldering
- **Dual Channel:** Two op-amps in one IC (Stage 1 + Stage 2)

**Alternatives Considered:**
- OPA2134: Lower noise (8 nV/√Hz) but more expensive ($2.50)
- NE5532: Good audio-grade op-amp but higher noise floor
- LM358: Not suitable (too much noise for microphone)

### 2. Dual LDO Voltage Regulation (3.3V + 2.5V)

**Why two regulators?**
- **+3.3V:** Powers the op-amp (analog supply, low noise critical)
- **+2.5V:** Powers the constant current bias source
- **Separate supplies:** Reduces cross-coupling noise between signal path and bias network
- **Better performance:** Each regulator can be optimized for its function

**Why NCP1117?**
- Dropout voltage ~0.6V (acceptable with 5V input)
- Low noise (good for analog applications)
- Low cost (~$0.35 each)
- Available in multiple voltage outputs (1.5V - 5V)

### 3. Gain = 100x (40dB)

**Calculation:**
```
Gain = 1 + (R7 / R6)
Gain = 1 + (990kΩ / 10kΩ) = 100
```

**Why 100x?**
- Typical IEPE microphone output: 5-50 mV/Pa (pressure)
- Desired output swing: 0-3.3V (full scale)
- For 1 Pa reference: 100x gain → ~20-50mV input → 2-5V output
- Provides good signal-to-noise ratio without overdrive

**Resistor Selection:**
- R6 = 10kΩ (standard value, easy to source)
- R7 = 990kΩ (calculated for 100x gain)
- Both: 0.1% tolerance, 5ppm/°C tempco (critical for stable gain)

### 4. High-Pass Filter (20Hz, fc = 1/(2π*R*C))

**Calculation:**
```
fc = 20Hz
fc = 1 / (2π * R * C)
R = 1.6kΩ
C = 5µF
fc = 1 / (2π * 1.6k * 5µF) = 19.89 Hz ≈ 20Hz
```

**Why 20Hz?**
- Attenuate DC offset and low-frequency vibrations
- Preserve audio band (20Hz - 20kHz)
- Reduce 1/f (pink) noise from microphone

**Why passive RC high-pass?**
- Simple, requires only one resistor and capacitor
- No additional power consumption
- Integrated with op-amp feedback network

### 5. Low-Pass Filter (20kHz, Anti-Aliasing)

**Calculation:**
```
fc = 20kHz
R8 = 1.6kΩ
C8 = 5nF
fc = 1 / (2π * R * C) = 1 / (2π * 1.6k * 5nF) = 19.89 kHz ≈ 20kHz
```

**Why 20kHz?**
- Nyquist frequency for 40kHz sampling
- Preserves full audio bandwidth (20Hz - 20kHz)
- Attenuates high-frequency noise and aliasing

**Why C0G/NP0 capacitor?**
- Temperature-stable (0±30 ppm/°C)
- Low distortion
- Ensures consistent cutoff frequency

### 6. AC Coupling (10µF Capacitor)

**Why AC coupling?**
- Blocks DC bias current from IEPE sensor
- Prevents DC offset from reaching next stage
- Enables DC operating point setting with R2

**Why 10µF?**
```
Xc = 1 / (2π * f * C)
@ 20Hz: Xc = 1 / (2π * 20 * 10µF) ≈ 800Ω (small relative to 10kΩ load)
```
- Low impedance at 20Hz cutoff
- Reasonable capacitor size

### 7. Input Impedance (10kΩ Load)

**Why 10kΩ?**
- Typical IEPE microphone output impedance: 100Ω - 1kΩ
- 10kΩ load: High impedance relative to source (impedance matching)
- Reduces loading effects on microphone
- Provides stable DC operating point

---

## Performance Analysis

### Noise Budget

```
OPA2333 Input-Referred Noise:
  Thermal noise: en = √(4*k*T*R*Δf)
  For R = 10kΩ, T = 300K, Δf = 20kHz:
  en ≈ 57 nV RMS (from resistors alone)
  
Op-amp noise: 10 nV/√Hz * √(20kHz) ≈ 1.4 µV RMS

Total input-referred noise: ≈ 1.5 µV RMS
Referenced to output (after 100x gain): 150 µV RMS
```

### Signal-to-Noise Ratio (SNR)

```
Assuming microphone output: 10 mV peak @ 1Pa
After 100x amplification: 1V peak output
Noise floor: 150 µV RMS

SNR = 20*log10(1V / 0.15mV) = 76.5 dB
(Good for audio applications)
```

### Frequency Response

- **HPF @ 20Hz:** -3dB attenuation starts below 20Hz
- **Flat Region:** 20Hz - 20kHz (±0.5dB)
- **LPF @ 20kHz:** -3dB attenuation at 20kHz, roll-off at -20dB/decade

---

## Component Tolerances & Stability

### Temperature Drift

**Gain Temperature Coefficient:**
```
R6 & R7: 5ppm/°C (thin-film resistors)
Gain drift over 0-70°C: ±0.35% (acceptable)
```

**Filter Frequency Drift:**
```
C: ±5% (typical film cap)
R: 1% (thin-film)
fc drift: ~1.5% over 0-70°C (acceptable for audio)
```

### Capacitor Selection Rationale

- **C1, C3 (Bypass):** Low-ESR electrolytic for low-frequency decoupling
- **C5, C9 (AC Coupling):** Film capacitors (PP/Polyester) for low distortion
- **C6 (HPF):** Film capacitor for stable frequency response
- **C7 (Op-amp bypass):** Ceramic or film, very close to IC
- **C8 (LPF):** C0G ceramic for temperature stability

---

## Testing & Validation

### Bench Testing Procedure

1. **Power Supply Check:**
   ```
   Measure +3.3V at U1 pin 8
   Measure +2.5V at R1 top node
   Measure bias current through R1: I = V/R = 2.5V/2.2kΩ ≈ 1.14mA
   ```

2. **Op-Amp Functionality:**
   ```
   Inject 1kHz sine wave (50mV) at input
   Measure output: should be ~5V peak (100x gain)
   Check for oscillation or instability
   ```

3. **Frequency Response:**
   ```
   Sweep 10Hz - 100kHz sine wave (50mV input)
   Measure gain at each frequency:
   - @ 10Hz: -6dB (HPF rolls off)
   - @ 100Hz: 40dB (flat gain)
   - @ 20kHz: 40dB (still flat)
   - @ 100kHz: ~34dB (LPF rolls off)
   ```

4. **Noise Measurement:**
   ```
   Connect input to 50Ω resistor to ground
   Measure output noise voltage (no input signal)
   Should be <150µV RMS with 20Hz-20kHz BW
   ```

5. **Total Harmonic Distortion (THD):**
   ```
   Inject 1kHz, 1V output sine wave
   Measure THD at output
   Should be <0.1% (excellent audio quality)
   ```

---

## Limitations & Future Improvements

### Current Limitations

1. **Power Consumption:** ~200mW (typical)
   - Dominated by op-amp and regulator quiescent currents
   - Not suitable for battery operation

2. **Output Swing:** 0 - 3.3V
   - Limited dynamic range
   - Could be extended to ±5V with bipolar supply

3. **Single-Channel Only**
   - Stereo would require two boards or quad op-amp

### Possible Improvements

1. **Lower Noise:** Replace OPA2333 with OPA2134 (8 nV/√Hz)
2. **Lower Power:** Use rail-to-rail op-amp with lower quiescent current
3. **Programmable Gain:** Add digitally-controlled gain (variable resistor)
4. **Programmable Filters:** Add tunable HPF/LPF with microcontroller
5. **Multi-Channel:** Design quad or octal version for array applications

---

## Manufacturing Considerations

### Recommended PCB Parameters
- **Copper Weight:** 1 oz (35µm) standard
- **Surface Finish:** HASL or ENIG (ENIG preferred for better solder joints)
- **Solder Mask:** Green (standard)
- **Silkscreen:** White

### Component Sourcing
- Most components available from Digi-Key, Mouser, or Alibaba
- Lead time: 1-2 weeks typical
- Cost per unit: ~$11 (prototype, single unit)

### Soldering
- Op-amp: Can be soldered with hotplate + oven OR hand soldering with patience
- Passives: Can be hand soldered with fine-tip iron
- Connectors: Standard wave or hand soldering

---

## References & Further Reading

1. **Analog Devices IEPE Sensor Conditioning:**
   - Application Note AN-282
   - https://www.analog.com/

2. **TI Op-Amp Design:**
   - SNOA384: Piezoelectric Transducer Interface Design
   - https://www.ti.com/

3. **Audio Engineering Society:**
   - Microphone measurement standards
   - https://www.aes.org/

4. **IEC 61094-2:** Measurement Microphone Standard
   - Defines IEPE specifications

---

## Contact & Support

For questions on this design, refer to:
- Schematic documentation (SCHEMATIC.md)
- PCB layout guide (PCB_LAYOUT_GUIDE.md)
- Bill of Materials (BOM.csv)
