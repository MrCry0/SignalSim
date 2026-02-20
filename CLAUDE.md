# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**SignalSim** is a multi-stage GNSS (Global Navigation Satellite System) signal and data simulator. It generates simulated GNSS signals for GPS, BeiDou, Galileo, and GLONASS constellations at various processing stages: reference trajectory, observation data (pseudorange/carrier phase/CNR), baseband correlation results, and digital IF (Intermediate Frequency) samples. The primary executable is `IFdataGen`.

## Build Commands

The main build target is `IFdataGen`. CMake is recommended; a `Makefile` is also available.

```bash
cd IFdataGen

# Release build (recommended)
cmake -S . -B out/build/release -DCMAKE_BUILD_TYPE=Release -DUSE_NATIVE_OPT=ON
cmake --build out/build/release -j$(nproc)

# Debug build
cmake -S . -B out/build/debug -DCMAKE_BUILD_TYPE=Debug
cmake --build out/build/debug -j$(nproc)

# Or via Makefile
make
```

**Dependencies:** C++20 compiler, CMake 3.16+, OpenMP (optional but recommended for ~10x speedup). Install on Ubuntu with: `sudo apt install build-essential cmake ninja-build libomp-dev`

For `XmlObsGen` and `JsonObsGen`, each has its own `CMakeLists.txt` in its subdirectory.

## Running and Testing

There is no automated test suite. Testing is done by running the binary against provided config files:

```bash
# Run with a config file
./out/build/release/IFdataGen -c configs/GPS_BDS_GAL_L1CA_L1C_B1C_B1I_E1.json

# Validate config without generating output
./out/build/release/IFdataGen -c configs/GPS_BDS_GAL_L1CA_L1C_B1C_B1I_E1.json -vo

# Time the run (outputs timing info)
./out/build/release/IFdataGen -c configs/IfGenTest.json -t

# Force single-threaded or multi-threaded mode
./out/build/release/IFdataGen -c config.json -st
./out/build/release/IFdataGen -c config.json -mt
```

Test configs are in `IFdataGen/configs/`. Ephemeris data files (RINEX) are in `EphData/`.

## Architecture

### Directory Structure

- `inc/` — All header files (public API)
- `src/` — All shared implementation files
- `IFdataGen/` — Primary executable: generates IF digital signal samples (`.bin` files)
- `XmlObsGen/` — XML-based observation data generator (legacy)
- `JsonObsGen/` — JSON-based observation data generator
- `EphData/` — RINEX ephemeris data files used as input
- `docs/` — Navigation message structure documentation

### Key Architectural Layers

**1. Data Types & Constants** (`inc/BasicTypes.h`, `inc/ConstVal.h`)
- `BasicTypes.h` defines all core structs: `GNSS_TIME`, `LLA_POSITION`, `KINEMATIC_INFO`, `GPS_EPHEMERIS`, `GLONASS_EPHEMERIS`, `SATELLITE_PARAM`, etc.
- `ConstVal.h` defines GNSS frequency constants and physical constants.
- `inc/SignalSim.h` is the master public header that includes all other headers.

**2. Navigation Message Generation** (`src/*NavBit.cpp`)
Each GNSS signal has a dedicated navigation bit stream class:
- GPS: `LNavBit` (L1CA), `CNavBit` (L2C/L5), `CNav2Bit` (L1C)
- BeiDou: `D1D2NavBit`, `BCNav1Bit`, `BCNav2Bit`, `BCNav3Bit`
- Galileo: `INavBit` (E1/E5b), `FNavBit` (E5a)
- GLONASS: `GNavBit`
- `PilotBit.cpp` (~100KB) handles pilot channel bit generation.

**3. Satellite & Signal Modeling** (`src/SatelliteSignal.cpp`, `src/SatIfSignal.cpp`)
- `SatelliteSignal` computes satellite position, velocity, clock, and signal parameters from ephemeris.
- `SatIfSignal` generates the actual IF signal samples for a satellite.
- `PrnGenerate` generates PRN spreading codes; pre-computed codes are stored in `src/MemoryCode.dat` (349KB binary).

**4. Trajectory & Coordinates** (`src/Trajectory.cpp`, `src/Coordinate.cpp`)
- `Trajectory` models the receiver's motion (static, kinematic, etc.).
- `Coordinate` handles transformations between LLA, ECEF, and local frames.

**5. Configuration Parsing** (`src/JsonParser.cpp`, `src/JsonInterpreter.cpp`, `src/XmlInterpreter.cpp`)
- JSON is the primary configuration format for `IFdataGen` and `JsonObsGen`.
- XML is the legacy format used by `XmlObsGen`.
- Config files specify: simulation time, receiver trajectory, ephemeris file paths, output parameters, signal power settings.

**6. I/O** (`src/Rinex.cpp`, `src/Almanac.cpp`)
- `Rinex` handles reading RINEX 2/3 navigation and observation files.
- Output is raw binary IF samples (16-bit I/Q pairs) plus a `.tag` metadata file.

**7. Main Entry Point** (`IFdataGen/IFdataGen.cpp`)
- ~1200 lines. Reads JSON config, sets up satellites, runs simulation loop.
- `IFdataGenThread.cpp` is an experimental multi-threaded variant using `std::thread`.
- OpenMP is used for parallelizing signal generation across satellites.

### Signal Support Matrix
- **GPS**: L1CA, L1C, L2C, L5
- **BeiDou**: B1C, B1I, B2I, B2a, B2b, B3I
- **Galileo**: E1, E5a, E5b, E6
- **GLONASS**: G1, G2 (FDMA)

### Configuration Format
JSON configs reference RINEX ephemeris files (absolute or relative paths). See `IFdataGen/configs/` for examples and `SignalSim JSON format specification.pdf` for the full schema.
