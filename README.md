# Kickmix-Circuit-X-Ray-Compression-Toolkit
An open-source diagnostic, profiling, and synthesis toolkit for analyzing and optimizing fault-tolerant quantum circuits in Google's **`kickmix`** (`.kmb` / `.kmx`) format, built around the benchmarks from the *Google Quantum AI — Verifiable Quantum Circuit Compilation* challenge.


---

## Overview

In fault-tolerant quantum computing, non-Clifford gates (Toffoli, `CCZ`, `CSWAP`, and arbitrary `Z_POW` rotations) consume expensive magic states, while logical qubits require hundreds to thousands of physical qubits for error correction. 

This repository provides a standalone Python/Colab pipeline to inspect compiled `.kmb` binaries, audit qubit lifetimes, detect redundant gate patterns, and extract exact Boolean truth tables from reversible QROM/ANF subroutines without requiring proprietary tournament infrastructure.

### Key Capabilities

* **Binary Circuit X-Ray (`inspect_kmb`):** Decompiles `.kmb` circuits into `.kmx` assembly to count exact Toffoli-equivalent costs, distinguish free Clifford/`T` rotations from expensive arbitrary-angle `Z_POW` gates, and measure classical branching (`PUSH_CONDITION`) and measurement-based uncomputation (`HMR`).
* **Ancilla Lifetime & Peak Memory Profiler:** Tracks concurrent active qubits across multi-million-line circuits (e.g., 53M-line `dqi.kmb` decoders) to identify peak memory bottlenecks and register allocation efficiency.
* **Peephole & Gate-Level Auditor:** Scans large circuits for adjacent self-canceling Toffoli gates (`CCX`, `CSWAP`, `CCZ`) and shared-control optimization opportunities.
* **Exact Truth Table Extractor:** Uses `kickmix.Simulator` to reverse-engineer and verify exact Boolean lookup tables (`2048 × 40-bit`) from compiled QROM/ANF blocks.
* **Algebraic Normal Form (ANF) Synthesizer:** Demonstrates Möbius-transform-based synthesis of reversible lookup tables using `HMR + CZ` measurement-based uncomputation, illustrating the fundamental space-time (\(Q \times T\)) trade-off between qubit count and Toffoli overhead.

---

## Key Architectural Findings

Through profiling top public benchmark circuits across the three core tasks (**\(1/\sqrt{x}\)**, **DQI Decoder**, and **FeMoCo Chemistry**), this toolkit highlights several critical compilation principles:

1. **Task-Specific Cost Metrics Drive Architecture:**
   * **\(1/\sqrt{x}\) (\(Q \times C\)):** Balanced space-time optimization. In-place square-root extraction combined with short division compresses circuits down to **46 qubits** while exhausting ~97.4% of the allowed phase error tolerance.
   * **DQI Decoder (\(R \times Q^2 \times \text{Toffoli}\)):** Quadratic sensitivity to qubit count (\(Q^2\)) rewards aggressive register reuse (889 qubits total, with all 49 working ancillas tightly packed at peak) and heavy conversion of quantum controls into classical conditions (`PUSH_CONDITION`).
   * **FeMoCo Chemistry (\(\lambda_{\text{eff}} \times \text{Toffoli} \times Q\)):** Combines tensor double-factorization (\(\lambda_{\text{eff}}\)) with reversible ANF/CNOT networks (`angle_k_0..14.kmb`). Reducing qubit count locally via dirty-ancilla wire folding introduces ~146 extra `CCX` gates per removed wire, confirming that global linear-nonlinear synthesis sits near a local space-time Pareto optimum.

---

## Quickstart (Google Colab / Local)

### 1. Prerequisites
Install `kickmix` from the official [Google Qualtran](https://github.com) repository alongside standard scientific Python libraries:

```bash
pip install numpy pandas
git clone --depth 1 https://github.com.git
pip install --no-deps ./Qualtran/kickmix
```

### 2. Running the Circuit X-Ray
Place your `.kmb` circuits (or unpacked `submission.zip`) into a working directory and run the inspection pipeline in the provided Jupyter Notebook:

```python
import kickmix as km
from pathlib import Path

circuit = km.Circuit.from_file("path/to/circuit.kmb")
print(f"Logical Qubits: {circuit.num_qubits}")
```

*(Note: Large binary `.kmb` files such as the 53M-line `dqi.kmb` are excluded from version control via `.gitignore`.)*

---

## Repository Structure

```text
├── notebooks/
│   └── Kickmix_XRay_and_Compression.ipynb   # Main diagnostic, profiling & ANF synthesis notebook
├── .gitignore                               # Excludes large .kmb binaries and temporary archives
└── README.md                                # Project documentation
```

---

## Acknowledgments & References

* **Simulation Engine:** Built on `kickmix` from [Google Qualtran](https://github.com).
* **Community Benchmarks & Insights:** Inspired by public research and circuit architectures shared by [manwithacat](https://kaggle.com), Syed Asad Ali (46-qubit `invsqrt` architecture), and [Oleg Sher](https://github.com) (high-density ANF/CNOT synthesis and classical-control DQI optimizations).
