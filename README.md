# ⚛️ Quantum Computing with Qiskit & IBM Quantum

A hands-on, lesson-driven repository exploring quantum information science, circuit synthesis, transpilation, and execution on real quantum hardware using **Qiskit 1.x** and **IBM Quantum Runtime**.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Key Topics & Concepts](#key-topics--concepts)
  - [1. Circuit Construction & Gate Operations](#1-circuit-construction--gate-operations)
  - [2. Hardware-Targeted Transpilation (ISA)](#2-hardware-targeted-transpilation-isa)
  - [3. IBM Quantum Runtime & Sampler V2](#3-ibm-quantum-runtime--sampler-v2)
- [Prerequisites & Environment Setup](#prerequisites--environment-setup)
- [Configuration & API Setup](#configuration--api-setup)
- [Quickstart: Workflow Example](#quickstart-workflow-example)
- [Best Practices](#best-practices)
- [Roadmap](#roadmap)
- [References & Resources](#references--resources)

---

## 🌌 Overview

Quantum computing leverages the fundamental principles of quantum mechanics—such as **superposition**, **entanglement**, and **interference**—to process information in ways that classical computers cannot replicate. 

This repository provides educational notebooks and practical implementations focusing on:
- Designing quantum circuits with multi-qubit interactions and measurement.
- Preparing circuits for real quantum hardware execution via **Instruction Set Architecture (ISA)** transpilation.
- Interfacing with IBM Quantum devices through `qiskit-ibm-runtime` and executing jobs via modern Qiskit Primitives (`Sampler`).

---

## 📁 Repository Structure

```text
quantum computing/
├── .env                  # Environment variables for IBM Quantum API keys (git-ignored)
├── .gitignore            # Protects sensitive files and environment variables
├── README.md             # Project documentation and guide
├── Lesson1.ipynb         # Lesson 1: Fundamentals of Quantum Computing & Single-Qubit Gates
└── Lesson3.ipynb         # Lesson 3: Quantum Circuits, Transpilation & IBM Runtime Service
```

---

## 🧠 Key Topics & Concepts

### 1. Circuit Construction & Gate Operations
- **Qubit Initialization & Superposition**: Applying Hadamard ($H$) gates to transform basis states $|0\rangle$ into equal superpositions $\frac{1}{\sqrt{2}}(|0\rangle + |1\rangle)$.
- **Entanglement**: Generating Bell pairs using Controlled-NOT ($CX$) gates.
- **Single-Qubit Rotations & Pauli Operators**: Utilizing Pauli-$X$ (bit-flip) and Pauli-$Z$ (phase-flip) gates.
- **Circuit Visualization**: Rendering circuits with Matplotlib integration (`qc.draw('mpl')`).

### 2. Hardware-Targeted Transpilation (ISA)
Physical quantum processing units (QPUs) have specific qubit coupling maps, native gate sets (e.g., $ECR$, $CZ$, or $RZ$), and calibration parameters.
- **Preset Pass Managers**: Using `generate_preset_pass_manager` with optimization levels (e.g., `optimization_level=3`) to compile virtual quantum circuits into **Instruction Set Architecture (ISA)** circuits.
- **Virtual-to-Physical Mapping**: Routing and swapping virtual qubits onto physical qubits with the lowest error rates.

### 3. IBM Quantum Runtime & Sampler V2
- **Backend Selection**: Querying operational utility-scale quantum processors (e.g., 127-qubit Eagle QPUs like `ibm_fez`) using `service.least_busy()`.
- **Primitives Execution**: Utilizing the `Sampler` primitive to sample quasi-probability distributions and bitstring outcomes from quantum circuits.

---

## 🛠️ Prerequisites & Environment Setup

### 1. Python Environment
Python **3.10+** is recommended. Create and activate an isolated virtual environment:

```bash
# Create a virtual environment
python -m venv .venv

# Activate on Windows (PowerShell)
.venv\Scripts\Activate.ps1

# Activate on macOS/Linux
source .venv/bin/activate
```

### 2. Install Required Dependencies

Install Qiskit, IBM Runtime, and visualization libraries:

```bash
pip install qiskit qiskit-ibm-runtime qiskit-aer python-dotenv matplotlib pylatexenc numpy jupyterlab
```

| Package | Purpose |
| :--- | :--- |
| `qiskit` | Core quantum software development kit (1.0+) |
| `qiskit-ibm-runtime` | Cloud access to real IBM Quantum processors and runtime primitives |
| `qiskit-aer` | High-performance quantum circuit simulator |
| `python-dotenv` | Secure loading of environment variables from `.env` |
| `matplotlib` & `pylatexenc` | High-quality circuit diagrams and histogram visualization |

---

## 🔑 Configuration & API Setup

To run workloads on real IBM Quantum QPUs, an IBM Quantum account is required:

1. Create a free account at [IBM Quantum Platform](https://quantum.cloud.ibm.com/).
2. Obtain your **API Token** from your account dashboard.
3. Create a `.env` file in the root directory (ensure it is listed in `.gitignore`):

```env
QISKIT_API_KEY=your_ibm_quantum_api_token_here
```

4. Authenticate within Python / Jupyter:

```python
import os
from dotenv import load_dotenv
from qiskit_ibm_runtime import QiskitRuntimeService

load_dotenv()
API_KEY = os.getenv("QISKIT_API_KEY")

# Save credentials for default usage
QISKIT_API_KEY = API_KEY
QiskitRuntimeService.save_account(
    channel="ibm_quantum_platform",
    token=API_KEY,
    overwrite=True,
    set_as_default=True
)
service = QiskitRuntimeService()
```

---

## 🚀 Quickstart: Workflow Example

Here is the standard Qiskit 1.0+ workflow demonstrated in [Lesson3.ipynb](file:///c:/CHANGE/projects/quantum%20computing/Lesson3.ipynb):

```python
from qiskit import QuantumCircuit
from qiskit.transpiler.preset_passmanagers import generate_preset_pass_manager
from qiskit_ibm_runtime import QiskitRuntimeService, SamplerV2 as Sampler

# 1. Create a 2-qubit circuit
qc = QuantumCircuit(2)
qc.h(0)
qc.cx(0, 1)
qc.x(1)
qc.z(0)
qc.measure_all()

# 2. Select the least-busy QPU
service = QiskitRuntimeService()
backend = service.least_busy(operational=True, simulator=False, min_num_qubits=127)
print(f"Target backend: {backend.name}")

# 3. Transpile circuit to target backend's native ISA
pm = generate_preset_pass_manager(target=backend.target, optimization_level=3)
qc_isa = pm.run(qc)

# 4. Initialize Sampler and submit execution job
sampler = Sampler(mode=backend)
job = sampler.run([qc_isa])
print(f"Submitted Job ID: {job.job_id()}")
```

---

## 🛡️ Best Practices

- **Never Commit API Keys**: Keep API tokens inside `.env` and verify that `.env` is included in `.gitignore`.
- **Always Transpile to ISA Before Runtime**: IBM Quantum Runtime Primitives (`Sampler`, `Estimator`) require circuits that conform to the target device's ISA (i.e. virtual qubits mapped to physical qubits and decomposed into native gates).
- **Test Locally on Simulators First**: Utilize `qiskit_aer.AerSimulator` to debug and validate quantum circuits before consuming queue time on real cloud QPUs.
- **Handle Deprecation Warnings**: Note that Qiskit Runtime APIs evolve continuously. Prefer the latest `Sampler` or client-side execution interfaces per package release notes.

---

## 🗺️ Roadmap

- [x] Basic gate operations and Bell state creation
- [x] Integration with IBM Quantum Runtime Service
- [x] Instruction Set Architecture (ISA) transpilation with optimization passes
- [ ] Lesson 2: Quantum superposition, phase kickback, and state tomography
- [ ] Quantum algorithms: Grover's Search & Quantum Fourier Transform (QFT)
- [ ] Error suppression and mitigation techniques (Dynamical Decoupling, TREX)
- [ ] Expectation value measurement using `EstimatorV2`

---

## 📚 References & Resources

- [Qiskit Documentation (1.x)](https://docs.quantum.ibm.com/)
- [Qiskit Runtime Migration Guide](https://docs.quantum.ibm.com/migration-guides/qiskit-runtime)
- [IBM Quantum Learning Courses](https://learning.quantum.ibm.com/)
- [Qiskit GitHub Repository](https://github.com/Qiskit/qiskit)
