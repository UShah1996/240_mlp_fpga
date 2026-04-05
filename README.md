# CMPE 240 – FPGA-Based MLP Accelerator for Handwritten Digit Recognition

**Course:** CMPE 240 Sec 01 – Advanced Computer Design  
**Platform:** Digilent Zybo Z7-10 (Zynq-7010 SoC)  
**Toolchain:** Xilinx Vivado Design Suite  

## Group 1

| # | Name | Student ID |
|---|------|-----------|
| 1 | Simul Barua | 018315661 |
| 2 | Sriram Sridhar | 016661411 |
| 3 | Manisha Varthamanan | 020095920 |
| 4 | Urmi Shah | 019107595 |
| 5 | Anusha Subramanian | 014122459 |
| 6 | Nkono Andrew Mwase | 014224548 |

---

## Project Overview

This project implements a hardware accelerator for a Multi-Layer Perceptron (MLP) that classifies handwritten digits (0–9) using the MNIST dataset. The MLP runs entirely on the Programmable Logic (PL) fabric of the Zynq-7010 SoC.

**Architecture:** 784 → FC1(64, ReLU) → FC2(32, ReLU) → FC3(10, Argmax)  
**Parameters:** 52,650 total → ~51 KB after INT8 quantization  
**Target accuracy:** ≥ 97% FP32, ≥ 95% INT8

---

## Repository Structure

```
cmpe240-mlp-fpga/
├── phase1/
│   ├── phase1_mlp.py              ← Training + quantization + export script
│   ├── Phase1_Documentation.docx ← Full code walkthrough document
│   └── output/
│       ├── W1.coe                 ← FC1 INT8 weights  (Vivado BRAM init)
│       ├── W2.coe                 ← FC2 INT8 weights
│       ├── W3.coe                 ← FC3 INT8 weights
│       ├── b1.coe                 ← FC1 INT32 biases
│       ├── b2.coe                 ← FC2 INT32 biases
│       ├── b3.coe                 ← FC3 INT32 biases
│       ├── scales.txt             ← Per-layer FP32 scale factors (read by PS/ARM)
│       ├── mlp_fp32.pth           ← Full-precision model checkpoint
│       └── *.npy                  ← Debug arrays
└── phase2/                        ← (RTL implementation — coming soon)
```

---

## Phase 1 – Software Pipeline

### Requirements

```bash
pip install torch torchvision numpy
```

### Run

```bash
python phase1/phase1_mlp.py
```

> **macOS/Windows note:** The script uses `num_workers=0` and a `if __name__ == '__main__'` guard to avoid multiprocessing spawn errors on non-Linux platforms.

### What it does

1. Downloads the MNIST dataset via `torchvision`
2. Trains a FP32 MLP for 20 epochs using Adam + CrossEntropyLoss
3. Applies symmetric per-layer INT8 post-training quantization
4. Verifies quantized accuracy on the full 10,000-image test set
5. Exports Xilinx `.coe` files for Vivado BRAM initialization

### Expected output

```
FP32 test accuracy  : ≥ 97.0%
INT8 test accuracy  : ≥ 95.0%
Accuracy drop       : < 2%
BRAM usage (est.)   : ~2 × 36Kb blocks
```

---

## Phase 2 – Hardware Implementation *(in progress)*

The quantized MLP will be implemented as a custom RTL accelerator in the Zynq-7010 PL:

- **8 parallel INT8 MAC units** mapped to DSP48E1 slices
- **BRAM-based weight storage** initialized from Phase 1 `.coe` files
- **AXI4-Lite interface** for PS ↔ PL communication
- **FSM controller** sequencing layer-by-layer inference
- **Estimated inference time:** ~65.7 µs per image at 100 MHz

---

## License

For academic use only – CMPE 240, Spring 2026.
