# SignalSim: Signal Generation Capabilities

## Supported Signals

### GPS

| Signal | Center Freq | Chip Rate | Modulation | Nav Message |
|--------|------------|-----------|------------|-------------|
| L1CA | 1575.42 MHz | 1.023 Mcps | BPSK(1) | LNAV |
| L1C | 1575.42 MHz | 2.046 Mcps | BOC(1,1) | CNAV2 |
| L2C | 1227.60 MHz | 1.023 Mcps | BPSK(1), TM-QPSK* | CNAV |
| L5 | 1176.45 MHz | 10.23 Mcps | BPSK(10) | CNAV |

*L2C time-multiplexes L2CM (data) and L2CL (pilot).

### BeiDou

| Signal | Center Freq | Chip Rate | Modulation | Nav Message |
|--------|------------|-----------|------------|-------------|
| B1C | 1575.42 MHz | 2.046 Mcps | BOC(1,1) | B-CNAV1 |
| B1I | 1561.098 MHz | 2.046 Mcps | BPSK(2) | D1/D2 |
| B2I | 1207.14 MHz | 2.046 Mcps | BPSK(2) | D1/D2 |
| B2a | 1176.45 MHz | 10.23 Mcps | BPSK(10) | B-CNAV2 |
| B2b | 1207.14 MHz | 10.23 Mcps | BPSK(10) | B-CNAV3 |
| B3I | 1268.52 MHz | 10.23 Mcps | BPSK(10) | B-CNAV3 |

### Galileo

| Signal | Center Freq | Chip Rate | Modulation | Nav Message |
|--------|------------|-----------|------------|-------------|
| E1 | 1575.42 MHz | 2.046 Mcps | BOC(1,1) | I/NAV |
| E5a | 1176.45 MHz | 10.23 Mcps | BPSK(10) | F/NAV |
| E5b | 1207.14 MHz | 10.23 Mcps | BPSK(10) | I/NAV |
| E6 | 1278.75 MHz | 5.115 Mcps | BPSK(5) | — |

### GLONASS (FDMA)

| Signal | Center Freq | Chip Rate | Modulation | Nav Message |
|--------|------------|-----------|------------|-------------|
| G1 | 1602 + k×0.5625 MHz | 0.511 Mcps | BPSK(0.5) | GNAV |
| G2 | 1246 + k×0.4375 MHz | 0.511 Mcps | BPSK(0.5) | GNAV |

*k = −7…+6 (frequency channel number).*

---

## Output Parameters

| Parameter | JSON key | Notes |
|-----------|----------|-------|
| Sample format | `format` | `IQ2`, `IQ4`, `IQ8`, `IQ16` (signed integer I/Q pairs) |
| Sampling frequency | `sampleFreq` | MHz, user-defined; must satisfy Nyquist for all selected signals |
| IF centre frequency | `centerFreq` | MHz, arbitrary |
| Noise floor | `noiseFloor` | dBHz, additive white Gaussian noise |
| Initial CN0 | `initPower` | dBHz (default 47), can be per-satellite |
| Elevation mask | `elevationMask` | degrees (default 3°) |
| Elevation-dependent power | `elevationAdjust` | boolean |

Output is a raw binary file of interleaved I/Q samples (`.bin`) plus a `.tag` metadata sidecar.

---

## Multi-constellation Frequency Plans

Common pre-validated combinations from the included configs:

| Signals | Centre Freq | Min Fs |
|---------|------------|--------|
| L1CA + L1C + B1C + E1 | 1575.42 MHz | 4.1 MHz |
| L1CA + L1C + B1C + B1I + E1 | 1568.29 MHz | 18.5 MHz |
| L1CA + L1C + B1C + B1I + E1 + G1 | 1582.21 MHz | 46.3 MHz |
| L2C + B2I + B2b + E5b + G2 | 1221.88 MHz | 53.5 MHz |
| L5 + B2a + E5a | 1176.45 MHz | 20.5 MHz |
| B3I + E6 | 1273.64 MHz | 30.8 MHz |

---

## Notes

- **L1P / L2P** (GPS) and **E5 AltBOC** (Galileo) require a commercial license and are not available in the open-source build.
- **GLONASS** requires proportionally wider sampling bandwidth when all 14 FDMA channels are active (~8–10 MHz for G1, ~6–8 MHz for G2).
- Navigation data is generated from RINEX ephemeris input; almanac is derived automatically.
- OpenMP parallelisation gives ~10× speed-up on multi-core hosts; enable with `-DUSE_NATIVE_OPT=ON` at build time.
