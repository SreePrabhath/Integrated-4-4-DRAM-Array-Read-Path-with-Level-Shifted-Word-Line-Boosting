# 🧠 DRAM Read Path: 1T-1C Cell, Level Shifter, Row Decoder & Sense Amplifier (180 nm, Cadence)

Design and simulation of the **core read path of a DRAM array** in Cadence Virtuoso. The project follows one read operation from **row selection**, through **charge sharing** on the bit-line, to a **full-swing output** from a latch-type sense amplifier.

> **Course:** Memory Circuits and System Design (Course Project: *DRAM Engineering Lab*)
> **Authors:** Joel Giby (EE25MTECH11006), N. Sree Prabhath (EE25MTECH11010)
> **Technology:** 180 nm CMOS

---

## 📌 What this project covers

| # | Block | What it does |
|---|---|---|
| 1 | **1T-1C DRAM cell** | One NMOS access transistor + one storage capacitor; shows charge sharing with the bit-line |
| 2 | **Latch-type sense amplifier** | Detects the small differential bit-line voltage and amplifies it to full swing |
| 3 | **2-to-4 row decoder** | Selects one of four word-lines from two address bits (`in_a0`, `in_a1`) |
| 4 | **Cross-coupled level shifter** | Boosts the decoder output (1.2 V) to the word-line voltage (2 V) |
| 5 | **4 × 4 DRAM array** | 16 cells with 4 word-lines, 4 bit-lines and 4 sense amplifiers |
| 6 | **Integrated read test** | Reads cell (0,0) (BL0 / WL0) through the full path |

---

## 🔬 How a DRAM read works (short version)

1. **Pre-charge:** the bit-line is set to `VPRE = VDD/2`, and the word-line is low (access transistor OFF).
2. **Access:** the decoder raises one word-line, which turns the access transistor ON and connects the cell capacitor to the bit-line.
3. **Charge sharing:** charge redistributes between `Ccell` and `CBL`, giving a small change on the bit-line:

```
VBL_final = (CBL·VPRE + Ccell·Vstored) / (CBL + Ccell)

ΔV ≈ [ Ccell / (CBL + Ccell) ] · |Vstored − VPRE|
```

   Reading a `1` pushes the bit-line slightly above VDD/2; reading a `0` pulls it slightly below.
4. **Sensing:** the sense amplifier turns this small ΔV into a full-swing logic level.

**Cell testbench values:** `Ccell = 30 fF`, `CBL = 200 fF`, access NMOS `W = 2 µm`, `L = 180 nm`. A stored `1` (2 V) on a bit-line pre-charged to about 1 V settles near **1.15 V** after the word-line goes high.

---

## ❓ Why a level shifter?

If the word-line only swings to VDD, the NMOS access transistor can't pass a full logic-high to the capacitor (it loses one threshold voltage, **Vt**). The cell is then under-charged and the read margin drops.

The word-line is therefore boosted above VDD (close to VDD + Vt). This turns the access transistor on strongly, allows full charge transfer, and improves sensing margin and reliability.

**Implementation:** a cross-coupled level shifter (cross-coupled PMOS pair on the higher supply, NMOS pair driven by the input). In simulation, a 1.2 V input pulse produces a clean **2 V** output.

---

## 🧱 Design Details

- **Sense amplifier:** cross-coupled latch with pins `BL`, `BL_`, `Pre_ch_Bar`, `READ_Bar`, `SAN`, `SAP`
- **Row decoder:** 2-to-4, with mutually exclusive outputs `out_WL0` to `out_WL3` (only one word-line is active at a time)
- **Level shifter:** one per word-line, between the decoder and the array
- **Array:** 4 × 4 cells with `C = 30 fF` each, plus one sense amplifier per bit-line

```
 in_a0, in_a1 ─► 2:4 Decoder ─► Level Shifter ×4 ─► WL0..WL3 ─► 4×4 1T-1C Array
                                  (1.2 V → 2 V)                      │
                                                              BL0..BL3
                                                                      │
                                                          Sense Amplifier ×4 ─► full-swing output
```

---

## 🧪 Simulation Results

- **Cell:** the word-line rises, and the bit-line and storage node meet at an intermediate voltage (charge sharing).
- **Sense amplifier:** a small differential on `BL` / `BL_` is resolved into a full-swing output when `SAN` / `SAP` are enabled.
- **Level shifter:** a 1.2 V input gives a 2 V output pulse train.
- **Integrated read of cell (0,0):** the cell is first charged to logic `1` (2 V) and then left floating. The sequence is pre-charge, word-line access, then sense-amplifier enable, and the bit-line resolves to full swing.
- A second testbench adds **inverters at the outputs** to recover the original bit polarity.

---

## 📁 Suggested Repository Structure

```
├── README.md
├── report/
│   └── DRAM_Engineering_Lab_Report.pdf
├── schematics/
│   ├── dram_cell/
│   ├── sense_amplifier/
│   ├── row_decoder/
│   ├── level_shifter/
│   └── dram_array_4x4/
├── waveforms/              # simulation screenshots
└── cadence_lib/            # (optional) exported Virtuoso library
```

---

## ▶️ How to Reproduce

1. Open the library in **Cadence Virtuoso** with the 180 nm PDK.
2. Simulate each block's testbench first: cell, sense amplifier, level shifter, decoder.
3. Open the integrated array testbench (read of cell (0,0)).
4. Run a **transient** analysis for about 10 ns and plot `PCh`, `Rd`, `SAN`, `SAP` and `BL_0`.

---

## ✅ Conclusion

The DRAM read path works end to end: word-line selection, word-line boosting through a level shifter, charge redistribution on the bit-line, and sensing by a latch-type amplifier. The project shows why **voltage boosting, timing control and differential sensing** matter for accurate DRAM operation.

---

## College
Indian Institute of Technology Hyderabad , MVLSI , ECE department
