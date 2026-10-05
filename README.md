# cophi

Compile [Blackbird](https://quantum-blackbird.readthedocs.io) quantum-optics programs into qubit-based [OpenQASM](https://openqasm.com) circuits using truncated Fock-space encoding.

## Overview

cophi bridges continuous-variable (CV) photonic circuits and discrete-variable gate-based quantum hardware.  It takes a Blackbird program describing Gaussian Boson Sampling (or related) circuits and produces an equivalent OpenQASM program where photon-number states are encoded in binary across qubits.

**Supported CV operations:**

| Blackbird gate    | Class            | Description                |
|:------------------|:-----------------|:---------------------------|
| `Sgate(r, φ)`    | `Squeeze`        | Single-mode squeezing      |
| `Dgate(r, φ)`    | `Displacement`   | Coherent-state displacement|
| `Rgate(φ)`       | `PhaseShift`     | Phase-space rotation       |
| `BSgate(θ, φ)` | `BeamSplitter`     | Beam splitter        |

**Fock cutoffs:**

| Cutoff | Qubits/mode | Photon states         |
|:-------|:------------|:----------------------|
| 2      | 1           | \|0⟩, \|1⟩           |
| 4      | 2           | \|0⟩, \|1⟩, \|2⟩, \|3⟩ |

## Installation

Requires Python ≥ 3.11.  Using [uv](https://docs.astral.sh/uv/):

```bash
# Clone and install in development mode
git clone https://github.com/Quantum-Cosmos-Lab/cophi.git
cd cophi
uv sync
```

Or install directly:

```bash
uv pip install .
```

## Quick start

```python
from cophi import Cophi
import blackbird as bb
import pennylane as qml

# 1. Load a Blackbird circuit
bb_circuit = bb.dumps(bb.load("examples/example.xbb"))

# 2. Compile to OpenQASM
compiler = Cophi(fock_cutoff=4)
qasm = compiler.compile_to_qasm(bb_circuit)

# 3. Simulate with PennyLane
num_qubits = compiler.required_qubits_num(bb_circuit)
circ = qml.from_qasm(qasm, measurements=qml.sample())
dev = qml.device("default.qubit", wires=num_qubits)
qnode = qml.QNode(circ, dev, shots=100)
raw_samples = qnode()

# 4. Convert binary → photon numbers
photon_counts = compiler.binary_to_photon_meas(raw_samples)
print(photon_counts)
```

## Working with individual operations

Each CV operation can be used standalone for inspection or custom circuit building:

```python
from cophi import Squeeze, Displacement, PhaseShift, BeamSplitter

# Symbolic matrix (SymPy)
from sympy import symbols
sq = Squeeze(fock_cutoff=4)
print(sq.symbolic_matrix(r=symbols('r'), phi=symbols(r'\phi')))

# Numerical matrix (NumPy)
disp = Displacement(fock_cutoff=4)
matrix = disp.numerical_matrix(r=0.1, phi=0.3)

# PennyLane gate decomposition
script = disp.gate_decomposition(r=0.1, phi=0.3, mode=0)
print(list(script))
```

## Development

```bash
# Install with dev + docs dependencies
uv sync --group dev --group docs

# Run tests
uv run pytest

# Run tests with coverage
uv run pytest --cov=cophi

# Lint
uv run ruff check src/ tests/

# Build docs locally
uv run mkdocs serve
```

## Citation

If you use cophi in your research, please cite it. Citation metadata is in
[`CITATION.cff`](CITATION.cff); GitHub's **"Cite this repository"** button exports it
as BibTeX or APA. Each release is archived on [Zenodo](https://zenodo.org) with its own DOI.

## Project structure

```
cophi/
├── src/cophi/
│   ├── __init__.py        # Public API
│   ├── compiler.py        # Cophi: Blackbird → QASM
│   ├── operations.py      # CV gate implementations
│   └── py.typed           # PEP 561 marker
├── tests/
│   └── test_cophi.py
├── docs/
│   ├── index.md
│   ├── getting-started.md
│   ├── theory.md
│   └── reference/
│       ├── operations.md
│       └── compiler.md
├── examples/
│   ├── example.xbb
│   └── example.py
├── mkdocs.yml
├── CITATION.cff
├── pyproject.toml
└── README.md
```