# S9-NodeOS

**A Lightweight, Hybrid Debian 12 (Bookworm) `armhf` Operating System for Antminer S9 Control Boards (Xilinx Zynq-7010)**

Turn repurposed Antminer S9 control boards into silent, ultra-low-power ARM Linux homelab servers, IoT gateways, and cluster nodes.

---

## Why S9-NodeOS? (Architecture & Trust)

Running a modern Linux distribution on older Zynq-7010 miner boards is challenging because mainline Debian kernels lack proprietary FPGA bitstreams, while stock miner firmwares are locked down and outdated. **S9-NodeOS** solves this with a transparent **two-layer hybrid design**:

1. **Proven Hardware & Kernel Layer (Braiins OS / Xilinx BSP):**  
   Retains the battle-tested Zynq bootloader (`boot.bin`, `u-boot.img`), FPGA bitstream (`system.bit.gz`), Device Tree (`devicetree.dtb`), and `Linux 4.4.0-xilinx` kernel repackaged into a clean `fit.itb` image—ensuring 100% hardware stability for Gigabit/100M Ethernet, GPIO LEDs, and internal XADC temperature sensors.
2. **Clean, Modern Debian 12 Userland:**  
   Bootstrapped directly from official Debian repositories (`deb.debian.org`) using `debootstrap`. Replaced heavy `systemd` with lightweight `sysvinit-core` for full Linux 4.4 compatibility and an idle RAM footprint of **~31 MB**.

### Safety & Transparency Guarantees
- **100% Non-Destructive (MicroSD-Only):** S9-NodeOS runs entirely from the MicroSD card and **never touches or overwrites your board's internal NAND flash**. To return to your previous miner firmware, simply power off, remove the MicroSD card, and move the boot jumper back to `NAND`.
- **Auditable & Reproducible:** Built from official Debian packages and transparent shell scripts—no closed-source binaries, tracking, or hidden services.

---

## Hardware Preparation: Setting the Boot Jumper

By default, Antminer S9 control boards are configured to boot from internal **NAND** storage and will ignore the MicroSD card unless you switch the boot mode jumper:

1. **Disconnect 12V power** from the control board completely before touching any jumpers.
2. Locate the 3-pin boot jumper header near the MicroSD card slot (typically labeled **`JP4`** or **`JP1`**, with silkscreen markings **`NAND`** and **`SD`**).
3. Move the jumper cap from the **`NAND`** position to the **`SD`** position.
4. Insert your flashed MicroSD card into the slot and connect Ethernet and 12V power.

> **Note:** On some S9 control board revisions without a pre-soldered header cap, bridging the two `SD` boot pins with a standard 2.54mm jumper cap is required.

---

## Key Features

- **Full Debian 12 (Bookworm) `armhf` Ecosystem:** Install Python, Node.js, WireGuard, Mosquitto, or custom services via `apt`.
- **Ultra-Low Memory Footprint (~31 MB Used):** Leaves ~470 MB of physical RAM free out of 512 MB, backed by **256 MB ZRAM** compressed swap.
- **Built-in Web Telemetry Dashboard (`:8080`):** Real-time monitoring of CPU load, RAM/ZRAM, Zynq-7010 SoC temperature (XADC), storage, network throughput, and a physical **Blink LEDs** locator button.
- **Cluster-Ready First Boot:**
  - Automatically expands the root partition to utilize the full capacity of any MicroSD card (`4GB`, `8GB`, `16GB`, `32GB+`).
  - Locks a persistent MAC address (preventing S9 random MAC changes across reboots).
  - Generates a unique hostname (`s9-node-xxxx`) and unique SSH host keys per board.
  - Syncs system clock automatically over HTTP on boot (compensating for the lack of an onboard RTC battery).
- **Clear Hardware LED Feedback:** Once boot finishes, the **Red LED turns off** and the **Green LED pulses** with a steady heartbeat.

---

## Quick Start Guide

1. **Download:** Grab `S9-NodeOS-v1.0-armhf.img.xz` from the [Releases](https://github.com/ParrotRex/S9-NodeOS/releases) page.
2. **Flash:** Write the image directly to a healthy MicroSD card (4 GB or larger) using **Rufus** (Windows) or **BalenaEtcher** (Windows/Linux/macOS).
3. **Boot:** Set the control board jumper to **`SD`**, insert the card, plug in Ethernet, and power on.
4. **Wait ~30 Seconds:** Watch for the **Red LED to turn off** and the **Green LED to blink steadily**.
5. **Connect:** Check your router's DHCP client list for `s9-node-xxxx`:
   - **Web Dashboard:** Open `http://<BOARD_IP>:8080` in your browser.
   - **SSH Access:**
     ```bash
     ssh root@<BOARD_IP>
     ```
     *(Default password: `root`)*

### Recommended Post-Install Security
Change the default `root` password immediately after your first login:
```bash
passwd
