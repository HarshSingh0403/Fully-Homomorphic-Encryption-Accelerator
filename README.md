<div align="center">

# FPGA-Accelerated Fully Homomorphic Encryption

### Hardware acceleration of the Brakerski–Vaikuntanathan (BV) FHE pipeline using FPGA technology

[![Language](https://img.shields.io/badge/HDL-Verilog-blue)](https://en.wikipedia.org/wiki/Verilog)
[![C++](https://img.shields.io/badge/C%2B%2B-HLS-00599C)](https://isocpp.org/)
[![Platform](https://img.shields.io/badge/FPGA-PYNQ--Z2-orange)](https://www.pynq.io/)
[![Tools](https://img.shields.io/badge/Tools-Vivado%20%7C%20Vitis%20HLS-red)](https://www.amd.com/en/products/software/adaptive-socs-and-fpgas/vivado.html)

</div>

## Overview

This project explores FPGA-based acceleration of the **Brakerski–Vaikuntanathan (BV) Fully Homomorphic Encryption (FHE) scheme**. Fully homomorphic encryption enables computation on encrypted data without first exposing the underlying plaintext, making it relevant to privacy-preserving computation and secure cloud workloads.

The design brings together key-generation, encryption, decryption, and polynomial-arithmetic components. The **Number Theoretic Transform (NTT)** and **Inverse Number Theoretic Transform (INTT)** are central to accelerating polynomial multiplication. The repository contains Verilog RTL experiments, C++/Vitis HLS implementations, testbenches, notebooks, and PYNQ deployment artifacts.

> **Reported result:** The accompanying project manuscript reports an overall **79.3× speedup** for the key-generation/encryption/decryption pipeline compared with its software baseline. Results depend on the manuscript's experimental setup and should be interpreted in that context.

## Architecture

At a high level, the project is organized around these functional blocks:

```text
                 ┌────────────────────┐
                 │   Key Generation   │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │     Encryption     │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │ Encrypted Payload  │
                 │  / Ciphertext      │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │     Decryption     │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │ Recovered Message  │
                 └────────────────────┘

        NTT / INTT accelerate polynomial arithmetic
        used by the cryptographic processing blocks.
```

The repository also includes standalone NTT/INTT designs and an NTT-based polynomial multiplier, with both small-point RTL implementations and a 512-point implementation.

## Repository contents

| Directory / file | Description |
|---|---|
| `FHE_Vitis/` | C++/Vitis HLS source for key generation, encryption, decryption, shared NTT utilities, test files, and Jupyter notebooks. |
| `NTT_HLS_Codes/NTT/` | HLS implementation of the forward Number Theoretic Transform (NTT). |
| `NTT_HLS_Codes/INTT/` | HLS implementation of the Inverse Number Theoretic Transform (INTT). |
| `NTT_HLS_Codes/NTT_INTT_Roundtrip/` | HLS code for testing an NTT → INTT round trip. |
| `NTT_HLS_Codes/NTT_Multiplier/` | HLS implementation of NTT-based polynomial multiplication. |
| `NTT_INTT_16point/` | Verilog RTL for a 16-point modular NTT/INTT datapath, including butterfly, modular arithmetic, stage, RAM, and bit-reversal modules, plus testbenches. |
| `NTT_INTT_512point/` | Verilog RTL for 512-point NTT and INTT modules, with a testbench. |
| `NTT_PYNQ/` | PYNQ notebooks and FPGA overlay artifacts (`.bit` and `.hwh`) for NTT, INTT, and NTT-based multiplication experiments. |
| `Software_Simulation/` | Notebook-based software simulation for NTT/INTT. |
| `prng_16bit.v` | 16-bit pseudo-random number generator RTL. |
| `prng_16bit_tb.v` | Testbench for the 16-bit PRNG. |
| `prng_512bit_lfsr.v` | LFSR-based PRNG RTL used for a 512-bit sequence. |
| `tb_prng_512bit_lfsr.v` | Testbench for the 512-bit LFSR PRNG. |
| `top_tb.v` | Top-level testbench file. |
| `Project Files.zip` | Additional project files bundled with the source archive. Inspect its contents for the relevant Vivado project or implementation files. |

## Technology stack

- **Verilog HDL** — RTL modules and hardware testbenches
- **C++ and Vitis HLS** — high-level hardware implementations and algorithm validation
- **AMD/Xilinx Vivado** — simulation, synthesis, and FPGA design integration
- **PYNQ-Z2 / Zynq-7000** — target FPGA development platform
- **Python and Jupyter Notebook** — software experiments, validation, and PYNQ-side testing

## Performance reported in the project manuscript

| Operation | Software time (s) | Hardware time (s) | Reported speedup |
|---|---:|---:|---:|
| Key generation | 0.31877 | 0.00459 | 69.5× |
| Encryption | 0.67892 | 0.00750 | 90.5× |
| Decryption | 0.32253 | 0.00455 | 70.9× |
| **Total** | **1.32022** | **0.01664** | **79.3×** |

These are the measurements reported in the accompanying manuscript, not a guarantee that every module or configuration in this repository will reproduce the same figures. Reproducing them requires matching the original software baseline, hardware configuration, clocking, and measurement procedure.

## Getting started

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/<your-repository>.git
cd <your-repository>
```

Replace the URL with your published repository URL.

### 2. Explore the RTL

Open the `.v` files in `NTT_INTT_16point/`, `NTT_INTT_512point/`, and the repository root in your preferred Verilog editor or simulator. The associated testbench files (`tb_*.v` and other `test*.v` files) are the starting points for simulation.

For example, using Icarus Verilog where the selected module and testbench are self-contained:

```bash
iverilog -g2012 -o sim.out path/to/design.v path/to/testbench.v
vvp sim.out
```

Replace the paths with the exact design and testbench files you want to run. Some designs may depend on multiple RTL files or require simulator-specific configuration.

### 3. Explore the HLS code

Open the relevant folder under `NTT_HLS_Codes/` or `FHE_Vitis/` in **Vitis HLS**. Use the included test files as a reference when creating a C simulation or synthesis project. Project settings, target part, clock constraints, and tool versions may need to be configured for your environment.

### 4. Run the PYNQ notebooks

The notebooks in `NTT_PYNQ/` are intended for experiments on a compatible PYNQ environment. The `.bit` and `.hwh` files are FPGA overlay artifacts; use a matching pair and the notebook associated with that design. A compatible PYNQ-Z2 board and configured software image are required for hardware execution.

## Validation and testing

The repository includes testbenches and HLS test files for several individual components, including PRNG, NTT, INTT, NTT/INTT round-trip, and NTT multiplication. Run the tests relevant to the module you are changing, and compare the outputs against a trusted software/reference implementation where available.

**Note:** The repository contains multiple experimental implementations and artifacts. The exact top module, source-file set, clock constraints, and project configuration can vary by design. Check the relevant testbench and project files before running synthesis or deploying an overlay.

## Applications

- Privacy-preserving computation on encrypted data
- Hardware acceleration research for homomorphic encryption
- FPGA-based polynomial arithmetic
- NTT/INTT architecture exploration
- RTL design, simulation, and HLS workflow practice

## Limitations and security note

This repository is an academic/research implementation and **is not a production-ready cryptographic library**. Do not use it to protect real confidential data without a thorough security review, validation of cryptographic parameters, and assessment by qualified cryptography experts. FPGA acceleration and functional simulation alone do not establish cryptographic security or side-channel resistance.

## Acknowledgment

This work was carried out as part of an academic research/internship project on FPGA acceleration of Fully Homomorphic Encryption. Please consult the project manuscript and source files for further technical details and attribution.

---

<div align="center">

**Built around Verilog RTL · C++/Vitis HLS · Vivado · PYNQ-Z2**

</div>
