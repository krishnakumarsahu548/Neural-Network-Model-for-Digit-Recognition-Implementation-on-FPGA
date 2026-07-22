# Neural-Network-Model-for-Digit-Recognition-Implementation-on-FPGA
Field Programmable Gate Arrays (FPGAs) are widely utilized across different industries because of their parallel processing capability.


# FPGA Implementation of Neural Network for Handwritten Digit Recognition

This repository contains the RTL design, training scripts, and hardware implementation for a handwritten digit recognition system built entirely on FPGA — from a Python-trained neural network all the way down to real-time inference on physical hardware.

The project implements and compares two neural network architectures — a **Deep Neural Network (DNN)** and a **Convolutional Neural Network (CNN, modified LeNet-5)** — both trained on the MNIST dataset and deployed on a **Xilinx PYNQ-ZU (Zynq UltraScale+ ZU5EV)** FPGA board using **SystemVerilog** and **AMD Vivado**.

This work was carried out as part of my M.Tech dissertation in VLSI Design & Embedded Systems at the **Department of Electronics & Communication Engineering, National Institute of Technology, Raipur**, under the supervision of **Dr. Mayur Katwe**.

---

## Why this project

Most handwritten digit recognition systems today run in software — on CPUs or GPUs — which works fine but isn't ideal for real-time, low-power, or embedded use cases like banking systems, postal sorting, or OMR evaluation. Neural networks are naturally parallel, and FPGAs are built for parallel, low-power computation. This project explores exactly that overlap: taking a neural network out of Python and PyTorch and implementing it directly in hardware, so that digit recognition happens in real time with a fraction of the power a software-based system would need.

The goal wasn't just to get something working in simulation — the design was pushed all the way to a physical FPGA board, with actual on-board inference and LED-based output, verified against the software model.

---

## What's implemented

- A fully custom **DNN (784 → 16 → 16 → 10)** trained on MNIST and implemented in SystemVerilog
- A **modified LeNet-5 CNN** (2 convolution + pooling stages + 3 fully connected layers, ~9,430 parameters) implemented as a complete RTL pipeline
- Fixed-point **Q8.8** arithmetic throughout (weights, biases, activations) to make the design hardware-friendly
- Custom RTL blocks: convolution engine, parallel MAC units, line buffers for sliding-window generation, max-pooling units, FSM-based control logic, and BRAM-based weight/bias storage
- A complete flow from **Python training → fixed-point weight conversion → `.mem`/`.coe` generation → RTL simulation → synthesis → on-board testing**
- Real hardware verification on a **PYNQ-ZU board**, with digit predictions read out on onboard LEDs and confirmed via ILA (Integrated Logic Analyzer) waveforms

---

## Architecture overview

### DNN
```
Input (784, from 28×28 MNIST image)
   → Hidden Layer 1 (16 neurons, ReLU)
   → Hidden Layer 2 (16 neurons, ReLU)
   → Output Layer (10 neurons, Softmax)
```

### CNN (Modified LeNet-5)
```
Input (32×32×1)
   → Conv1 (5×5, 6 filters) + ReLU        → 28×28×6
   → MaxPool1 (2×2)                        → 14×14×6
   → Conv2 (5×5, 16 filters) + ReLU       → 10×10×16
   → MaxPool2 (2×2)                        →  5×5×16
   → Flatten                                → 400
   → FC1 (16, ReLU) → FC2 (16, ReLU) → FC3 (10, Argmax)
```

Both networks were first trained in Python (PyTorch for CNN, NumPy-based scripts for DNN), then their weights and biases were quantized to fixed-point Q8.8 format and loaded into on-chip BRAM for hardware inference.

---

## Neural network → FPGA flow

1. **Train in Python** — model trained on MNIST, weights and biases extracted
2. **Quantize** — floating-point weights converted to fixed-point (Q8.8)
3. **RTL design** — neurons, layers, and control logic written in SystemVerilog
4. **Testbench simulation** — predicted vs expected outputs verified via waveform and TCL logs in Vivado
5. **BRAM initialization** — `.coe`/`.mem` files generated, clock and pin constraints added
6. **Synthesis & implementation** — RTL mapped to hardware, timing checked
7. **Bitstream generation** — FPGA programmed
8. **On-board testing** — real digit images fed to the board, predictions verified via ILA and LED output

---

## Results

| Model | Accuracy | LUT | FF | BRAM | DSP | Power | Latency |
|---|---|---|---|---|---|---|---|
| **DNN** (784-16-16-10) | 98.0% | 2,972 (2.54%) | 4,689 (2.00%) | 60 (41.67%) | 5 (0.40%) | 0.384 W | 0.52 ms |
| **CNN** (Modified LeNet-5) | 99.0% | 21,679 (18.51%) | 55,625 (23.75%) | 81.5 (56.60%) | 304 (24.36%) | 0.511 W | 874 µs |

- Both models were tested on 100 MNIST samples in simulation, with the DNN correctly classifying 98/100 digits.
- On-board hardware predictions matched RTL/behavioral simulation outputs exactly.
- CNN gives higher accuracy due to better feature extraction, while the DNN is significantly lighter on hardware resources — a classic accuracy-vs-efficiency trade-off, both demonstrated on the same board.

Compared to prior published FPGA implementations, this design achieves higher accuracy at a fraction of the power consumption (see the comparison tables in the dissertation report / paper for full details against existing work).

---

## Hardware setup

- **Board:** Xilinx PYNQ-ZU (Zynq UltraScale+ ZU5EV)
- **Programming:** UART-JTAG connection from a laptop
- **Power supply:** 12V external supply
- **Output indication:** Onboard LEDs (0–3) display the predicted digit in binary
- **Tools:** AMD Vivado 2023.1 for synthesis, implementation, and hardware debugging (ILA)

---

## Tools & libraries used

- **SystemVerilog** — RTL design of neurons, layers, MAC units, and control FSMs
- **AMD Vivado** — simulation, synthesis, implementation, and on-board debugging
- **Python + PyTorch** — CNN model design, training, and optimization
- **Torchvision** — MNIST dataset loading and preprocessing
- **NumPy** — tensor-to-array conversion and fixed-point weight formatting

---

## Publications

1. Krishna Kumar Sahu, Mayur Katwe, Bhavana Agrawal, Pratyaksh Singh, Metuku Eswar Chandra, Saurabh R Chiraniya, *"Digit Recognition using a Neural Network Model Implemented on FPGA with SystemVerilog"*, 4th International Conference on Paradigm Shifts in Communication, Embedded Systems, Machine Learning and Signal Processing (PCEMS 2025), VNIT Nagpur. **(Accepted, Oral Presentation)**

2. Krishna Kumar Sahu, Mayur Katwe, *"Comparative Analysis of CNN and DNN-Based Hardware Accelerators on FPGA Implementation for Pattern Recognition"*, The Journal of Supercomputing (Springer Nature), 2026. **(Submitted)**

---

## Future work

- Real-time physical digit input via camera/touchpad instead of pre-stored `.mem` pixel files
- On-chip training support with hardware-based backpropagation
- Ultra-low precision networks (Q4.4 or Binary Neural Networks) for further resource optimization
- Hardware-based softmax and improved pipelining for faster inference
- Extension to multi-digit recognition and full OCR applications

---

## Applications

This kind of low-power, real-time FPGA-based recognition system is directly relevant to:
- **Banking** — automatic cheque and form digit recognition
- **Postal systems** — sorting by handwritten PIN codes
- **Education** — OMR sheet reading and exam evaluation
- **Security** — handwritten signature verification
- **Embedded/IoT devices** — low-power, real-time recognition on edge hardware

---

## Author

**Krishna Kumar Sahu**
M.Tech, VLSI Design & Embedded Systems
Department of Electronics & Communication Engineering
National Institute of Technology, Raipur

Under the supervision of **Dr. Mayur Katwe**, Assistant Professor, Dept. of ECE, NIT Raipur

---

## Acknowledgements

Thanks to the Department of Electronics & Communication Engineering, NIT Raipur, for the lab resources and support that made the hardware implementation and testing possible.
