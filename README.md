# AMD XDMA Driver for Ubuntu 24.04 & Linux Kernel 6.18 (Artix-7)

This repository contains the adapted Xilinx XDMA reference driver compatible with modern Linux distributions (**Ubuntu 24.04 LTS**), modern compiler toolchains (**GCC 13**), and recent kernel versions (**Linux Kernel 6.18+**), successfully verified on an **Artix-7 FPGA M.2** hardware platform.

## Key Updates & Bug Fixes

* **Kernel 6.6+ / 6.18 Compatibility:** Fixed compilation issues regarding AIO (`ki_complete` arguments) and updated `iov_iter` accessors.
* **UBSAN Compliance:** Resolved "array-index-out-of-bounds" warnings in `libxdma.c` associated with flexible array member handling under strict runtime sanitizers.
* **Modern Toolchain Support:** Cleanly compiled and tested using **GCC 13** on Ubuntu 24.04 without deprecated macro warnings (`_DEFAULT_SOURCE` migration).
* **MSI Interrupt Mechanism:** Configured and validated single-vector MSI interrupts to eliminate transfer timeouts.

## Hardware Environment

* **FPGA:** Xilinx Artix-7 (e.g., XC7A200T)
* **PCIe Link:** Gen2 x4 (Stable connection verified via `lspci`)
* **OS:** Ubuntu 24.04 LTS (Kernel 6.18+)

## Build & Installation

1. Clone the repository:
```bash
● git clone https://github.com/DaiZhiyuan/xdma-artix7-linux.git
● cd xdma-artix7-linux/xdma
● make
● cd ../tools/
● make
```
2. Load the driver:

```bash
● insmod ./xdma.ko interrupt_mode=1

● lspci -v -s 01:00.0
01:00.0 Serial controller: Xilinx Corporation Device 7024 (prog-if 01 [16450])
Subsystem: Xilinx Corporation Device 0007
Flags: bus master, fast devsel, latency 0, IRQ 135
Memory at e1100000 (32-bit, non-prefetchable) [size=1M]
Memory at e1200000 (64-bit, non-prefetchable) [size=64K]
Capabilities: [40] Power Management version 3
Capabilities: [48] MSI: Enable+ Count=1/1 Maskable- 64bit+
Capabilities: [60] Express Endpoint, MSI 00
Capabilities: [100] Device Serial Number 00-00-00-00-00-00-00-00
Kernel driver in use: xdma
```

3. Verification & Testing

Navigate to the test directory and run the verification script to execute H2C and C2H DMA data transfers:

```bash
● cd ../tests/

● ./run_test.sh
Info: Number of enabled h2c channels = 2
Info: Number of enabled c2h channels = 2
Info: The PCIe DMA core is memory mapped.
Info: Running PCIe DMA memory mapped write read test
      transfer size:  1024
      transfer count: 1
Info: Writing to h2c channel 0 at address offset 0.
Info: Writing to h2c channel 1 at address offset 1024.
Info: Wait for current transactions to complete.
/dev/xdma0_h2c_0 ** Average BW = 1024, 6.841490
/dev/xdma0_h2c_1 ** Average BW = 1024, 6.920556
Info: Writing to h2c channel 0 at address offset 2048.
Info: Writing to h2c channel 1 at address offset 3072.
Info: Wait for current transactions to complete.
/dev/xdma0_h2c_0 ** Average BW = 1024, 9.195237
/dev/xdma0_h2c_1 ** Average BW = 1024, 6.611613
Info: Reading from c2h channel 0 at address offset 0.
Info: Reading from c2h channel 1 at address offset 1024.
Info: Wait for the current transactions to complete.
/dev/xdma0_c2h_0 ** Average BW = 1024, 6.478266
/dev/xdma0_c2h_1 ** Average BW = 1024, 13.330556
Info: Reading from c2h channel 0 at address offset 2048.
Info: Reading from c2h channel 1 at address offset 3072.
Info: Wait for the current transactions to complete.
/dev/xdma0_c2h_0 ** Average BW = 1024, 6.294837
/dev/xdma0_c2h_1 ** Average BW = 1024, 6.516523
Info: Checking data integrity.
Info: Data check passed for address range 0 - 1024.
Info: Data check passed for address range 1024 - 2048.
Info: Data check passed for address range 2048 - 3072.
Info: Data check passed for address range 3072 - 4096.
Info: All PCIe DMA memory mapped tests passed.
Info: All tests in run_tests.sh passed.

● cat /proc/interrupts | grep xdma
            CPU0       CPU1       CPU2       CPU3       CPU4       CPU5
 135:         8          0          0          0          0          0  PCI-MSI-0000:01:00.0    0-edge      xdma
```


