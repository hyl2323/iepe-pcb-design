# PCB Layout Guide - IEPE Microphone Signal Conditioning

## Board Specifications
- **Size:** 97mm × 57mm (Credit card form factor)
- **Layers:** 2 (Top and Bottom copper)
- **Copper Weight:** 1 oz (35µm)
- **Trace Width (Signal):** 0.25mm (10 mil)
- **Trace Width (Power):** 0.5mm (20 mil)
- **Via Diameter:** 0.3mm (12 mil)
- **Via Pad:** 0.6mm (24 mil)
- **Minimum Clearance:** 0.15mm (6 mil)
- **Solder Mask:** Standard green
- **Silkscreen:** White

---

## Layer 1 (Top) - Component Placement

```
+------- 97mm -------+
| J1 (BNC Input)    |
|                   |
| U1 (OPA2333)      |   57mm
| R6 R7 (Gain)      |
|                   |
| U2 U3 (Regulators)|
| C1-C9 (Bypass)    |
|                   |
| J2 (BNC Output)   |
| J3 (5V Power)     |
+-------------------+
```

### Component Placement Priorities

1. **Input Section (Left side):**
   - J1 (BNC Microphone Input) - top left corner
   - C5 (AC Coupling) - directly after J1
   - R2 (Input impedance) - near C5
   - U1 (Op-amp) - central position

2. **Amplifier Section (Center):**
   - U1A (Buffer stage) - input side
   - R3, C6 (HPF network) - between U1A and U1B
   - U1B (Gain stage) - output side
   - R6, R7 (Gain resistors) - **KEEP CLOSE TO U1B** (<10mm)

3. **Output Section (Right side):**
   - R8, C8 (LPF) - after U1B
   - C9 (Output coupling) - before J2
   - J2 (BNC Output) - right edge

4. **Power Supply Section (Bottom):**
   - U2, U3 (Regulators) - bottom center
   - C1, C2, C3, C4 (Bypass caps) - within 5mm of regulator pins
   - J3 (Power input) - bottom left or right

---

## Layer 2 (Bottom) - Ground Plane

**Completely fill with GND (ground plane) with minimal clearances for:**
- Regulator output pads
- Op-amp ground pads
- Connector ground pads
- Via stitching to top layer

---

## Signal Path Routing

### Critical Paths (Keep Short & Direct)

1. **Input Path:**
   - J1 (Hot) → C5 → R2 → U1A Pin 3 (Non-inverting) = **<30mm**
   - Keep width: 0.25mm
   - Keep away from power traces

2. **Feedback Path (Gain-setting):**
   - U1B Pin 7 (Output) → R7 → U1B Pin 6 (Inverting) = **<15mm**
   - Keep width: 0.3mm (slightly wider for stability)
   - **CRITICAL:** Minimize parasitic capacitance
   - Use guard trace on sides if needed

3. **Output Path:**
   - U1B Pin 7 → R8 → C8 → C9 → J2 = **<40mm**
   - Keep width: 0.25mm

### Ground Connections

- **Input GND:** Single star point at R2 base (to ground plane with multiple vias)
- **Op-amp GND:** Direct via to ground plane (short traces <5mm)
- **Regulator GND:** Wide trace to ground plane
- **Connector Shields:** Direct to ground plane with large vias

---

## Bypass Capacitor Placement

| Capacitor | Location | Distance from Pin | Purpose |
|-----------|----------|-------------------|---------|
| C2 (0.1µF) | U2 GND pin | <5mm | High-frequency bypass |
| C1 (10µF) | U2 GND pin | <5mm | Low-frequency bypass |
| C4 (0.1µF) | U3 GND pin | <5mm | High-frequency bypass |
| C3 (10µF) | U3 GND pin | <5mm | Low-frequency bypass |
| C7 (10µF) | U1 Pin 4 (GND) | <5mm | Op-amp supply bypass |

---

## Via Stitching

- Place vias around ground plane perimeter (1cm spacing) to connect layers
- Place vias under all connector pads
- Place vias under regulator pads
- Minimum via size: 0.3mm drill, 0.6mm pad

---

## Trace Routing Strategy

### Top Layer Routing
1. **Route input signal path first** (J1 → U1A)
2. **Route feedback network** (U1B feedback)
3. **Route output path** (U1B → J2)
4. **Route power distribution** (5V from J3)
5. **Fill remaining space with ground polygons**

### Bottom Layer (Ground Plane)
- Fill 100% with GND copper
- Create cutouts only where necessary for trace routing
- Maintain continuity (no isolated islands)

---

## EMI Mitigation

1. **Guard Traces:**
   - Run GND traces on both sides of input signal path
   - Keep input traces >2mm from power traces

2. **Shielding:**
   - Optional: Add shielded enclosure around input stage
   - BNC connector shields must connect to ground plane

3. **Component Grouping:**
   - Group input stage components together
   - Group output stage components together
   - Separate power supply stage physically

4. **Ferrite Bead:**
   - Optional: Add 0Ω ferrite bead on 5V input to J3
   - Helps reduce high-frequency noise coupling

---

## Thermal Considerations

- **Op-amp U1:** Dissipates ~50mW (minimal heat)
- **Regulators U2, U3:** Dissipate ~50mW each
- **No heat sinks required** (passive design)
- Spread components evenly to avoid hot spots

---

## Manufacturing Notes

1. **Solder Mask:** Green solder mask (standard)
2. **Silkscreen:** White, place reference designators on top layer
3. **Edge Clearance:** 0.5mm clearance from PCB edge to copper
4. **Connector Placement:** Mount BNC connectors on board edges for ergonomics
5. **Test Pads:** Optional: Add 0.5mm test points for VCC, GND, output signal

---

## DFM (Design for Manufacturability) Checklist

- [ ] No traces <0.2mm width
- [ ] No clearances <0.15mm
- [ ] All vias >0.3mm drill
- [ ] No isolated copper islands
- [ ] Copper-to-edge clearance >0.5mm
- [ ] Solder mask clearance correct
- [ ] Silkscreen text not overlapping pads
- [ ] All components have correct footprints
- [ ] No dead copper (remove orphaned traces)

---

## Assembly Order

1. **Install through-hole connectors** (J1, J2, J3)
2. **Solder SMD resistors** (R1-R8)
3. **Solder SMD capacitors** (C1-C9)
4. **Solder voltage regulators** (U2, U3)
5. **Solder op-amp IC** (U1) - use hot air rework if no reflow oven
6. **Solder TVS diode** (D1)
7. **Inspect for cold solder joints**
8. **Apply conformal coating** (optional, for humidity protection)

---

## Recommended PCB Vendors

- **JLCPCB** - Fast turnaround, affordable
- **PCBWay** - Good quality, assembly services
- **iTeadstudio** - Budget option
- **Seeed Studio** - High quality

Typical lead time: 7-14 days
Estimated PCB cost: $2-5 for prototype (qty 5-10)
