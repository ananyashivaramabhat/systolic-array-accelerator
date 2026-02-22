# Systolic Array Accelerator - Xilinx Artix-7

![Status](https://img.shields.io/badge/Status-In%20Progress-orange)
![Language](https://img.shields.io/badge/Verilog%20%7C%20SystemVerilog-blue)
![Tool](https://img.shields.io/badge/Vivado%202025.1-red)
![Board](https://img.shields.io/badge/Arty%20A7-Xilinx%20Artix--7-green)
![Arty A7 board used in this project](images/board.jpeg)

A hardware accelerator built from scratch on an FPGA that performs matrix multiplication using a systolic array - the same fundamental architecture behind Google's TPU and Nvidia's Tensor Cores.

I started this project to understand how ML hardware actually works under the hood, not just use it. If you're curious about the same thing, hopefully this repo is useful.

---

## Background

Every neural network bottlenecks on one operation: matrix multiplication. CPUs do it sequentially. GPUs parallelize it. TPUs take it further - they use a **systolic array**, a grid of tiny multiply-accumulate (MAC) units where data flows directly between neighbors every clock cycle, never going back to memory.

This project implements that idea on a Xilinx Artix-7 FPGA. Small scale, but the architecture is the same.

---

## How it works

The core building block is a **MAC unit**. Every clock cycle it does:

```
acc = acc + (a_in × b_in)
```

It also passes `a_in` to its right neighbor and `b_in` to its bottom neighbor. Wire 16 of these in a 4×4 grid, stagger the inputs correctly, and you get a machine that computes a full 4×4 matrix multiply in hardware - all 16 output elements accumulating simultaneously.

```
        B col0↓   B col1↓   B col2↓   B col3↓

A row0→ [MAC00] → [MAC01] → [MAC02] → [MAC03]
           ↓          ↓         ↓         ↓
A row1→ [MAC10] → [MAC11] → [MAC12] → [MAC13]
           ↓          ↓         ↓         ↓
A row2→ [MAC20] → [MAC21] → [MAC22] → [MAC23]
           ↓          ↓         ↓         ↓
A row3→ [MAC30] → [MAC31] → [MAC32] → [MAC33]
```

---

## Project structure

```
systolic-array-fpga/
├── src/
│   ├── mac_unit.v            # MAC processing element
│   └── systolic_array.v      # 4×4 top-level (in progress)
├── sim/
│   ├── mac_unit_tb.sv        # MAC unit testbench
│   └── systolic_array_tb.sv  # Array testbench (in progress)
└── README.md
```

---

## Simulation results

MAC unit tested over 4 clock cycles with `a = [1,2,3,4]`, `b = [5,6,7,8]`.
Expected: `1×5 + 2×6 + 3×7 + 4×8 = 70`

```
Cycle 1 | a=1 b=5 | acc=5    PASS
Cycle 2 | a=2 b=6 | acc=17   PASS
Cycle 3 | a=3 b=7 | acc=38   PASS
Cycle 4 | a=4 b=8 | acc=70   PASS
Final   | acc=70              PASS
a_out passthrough             PASS
b_out passthrough             PASS
After reset | acc=0           PASS
```

All 8 assertions passed in Vivado xsim 2025.1.

---

## Running it yourself

You'll need Vivado ML Edition (free) and an Arty A7 board.

1. Clone the repo
   ```bash
   git clone https://github.com/ananya473/systolic-array-accelerator.git
   ```
2. Open Vivado, create a project targeting `xc7a35ticsg324-1L`
3. Add `src/mac_unit.v` as a design source, `sim/mac_unit_tb.sv` as a sim source
4. Run Behavioral Simulation and check the TCL console

---

## What's next

- [x] MAC unit design and simulation
- [ ] 4×4 systolic array top-level
- [ ] Input staggering logic
- [ ] UART interface to send matrices from a laptop
- [ ] Full end-to-end demo

---

## References

- Kung, H.T. (1982). *Why Systolic Architectures?* - IEEE Computer
- Jouppi et al. (2017). *In-Datacenter Performance Analysis of a Tensor Processing Unit* - Google

---

**Ananya Shivarama Bhat** - MSE ESE, University of Pennsylvania  
[LinkedIn](https://www.linkedin.com/in/ananya473/) · [Portfolio](https://ananyabhat.framer.website/)
