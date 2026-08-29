# Labs Guide

# HAPS Labs Guide

## E-CoreFlow Setup Procedure

1. **Launch E-CoreFlow**

   - Synopsys ProtoCompiler is an automated software tool designed exclusively for Synopsys HAPS FPGA-based prototyping hardware to accelerate SoC and ASIC design bring-up.
   - E-CoreFlow is automated software for FPGA configuration, customized for EPS-VU13P and EPS-VU19P.

   ```bash
   $ chmod 777 E-CoreFlow
   $ GDK_BACKEND=x11 ./E-CoreFlow >/dev/null 2>&1 &
   ```

   ![E-CoreFlow launch](media/image11.png)

2. **Select Board → VU13P**

   ![Select board](media/image5.png)

3. **Open configuration files:** `*.bmprj`

4. **Select System → Lan Select System → USB Ethernet**

5. **Select System → Lan Select System → USB Ethernet**

   Set host fixed IP address:
   - Address: `192.168.1.100`
   - Netmask: `255.255.255.0`

   ![IP address settings](media/image6.png)
   ![IP address settings](media/image9.png)

   Scan the network and connect the HAPS.

   ![Connect HAPS](media/image3.png)

   Scan the network.

   ![Scan network](media/image1.png)
   ![Scan network results](media/image8.png)

6. **Power on the HAPS**

   ![Power on HAPS](media/image7.png)
   ![Power on HAPS](media/image2.png)

7. **Configure the System and FPGA**

   Configure the system via virtual JTAG; this takes a few seconds to complete.

   Confirm the bit file for FPGA configuration.

   ![Confirm bit file](media/image4.png)

   Configure the system.

   ![Configure system](media/image10.png)

## Connect SMF Daughter Card

1. **Connect the SMF daughter card using a USB-UART bridge**

   To connect an SMF daughter card to a host computer via a USB-UART bridge, follow this step-by-step wiring and setup guide.

   Use a standard USB Type-A to Micro-USB cable to connect a peripheral device (with a Micro-USB port) to a USB Type-A port on a host PC.

   **Connection steps:**
   - Plug the USB Type-A end into an available rectangular port on your host PC.
   - Plug the Micro-USB end into your target device (such as a phone, development board, or external drive).
   - Allow your PC to automatically detect the device and install the required drivers.

2. **Open the terminal using a utility program, such as PuTTY**

   You can open a remote terminal session by launching PuTTY and entering your server's host port name (ttyUSB port) with a 115200 baud rate.

   - **Connection type:** Select `Serial` in PuTTY (instead of SSH or Telnet).
   - **Serial line:** Enter your specific port (e.g., `/dev/ttyUSB0` on Linux).
   - **Speed:** Enter `115200`.

   ```bash
   $ sudo chmod 666 /dev/ttyUSB*; ls /dev/ttyUSB*
   $ putty -serial -sercfg 115200,8,n,1,N /dev/ttyUSB0 -fn "client:Ubuntu Mono 16" &
   ```

3. **Power on the SMF daughter card and observe the Linux boot sequence in the terminal**

   To view the Linux boot sequence on an SMF daughter card, connect a serial cable to the board's UART port and open a serial terminal program (such as PuTTY or minicom) configured to the correct baud rate (commonly 115200).

   **Typical embedded Linux boot stages:**
   - **ROM bootloader / initial hardware init:** The processor executes internal boot ROM code, initializes basic clocks and memory controllers, and loads the secondary bootloader.
   - **U-Boot / bootloader:** Displays version information, initializes board peripherals, counts down the autoboot prompt, and loads the Linux kernel image and device tree into RAM from flash or an SD card.
   - **Linux kernel initialization:** Decompresses the kernel, probes hardware drivers, detects CPU and memory, and mounts the initial RAM filesystem (initramfs).
   - **User space / init process:** Launches systemd or init (PID 1), starts system services, and finally presents the login or console shell prompt.

   **Linux boot message:**

   ```
   Xilinx Zynq MP First Stage Boot Loader
   U-Boot 2022.01 (Sep 20 2022)
     CPU: ZynqMP, Silicon: v3
     DRAM: 20GB (16GB+4GB)
     PMUFW: v1.1
     EL Level: EL2
     Chip ID: zu4
   PynqLinux, based on Ubuntu 22.04
   xilinx@pynq (password: xilinx)
   ```

   ![Boot sequence](media/image13.png)
   ![Boot sequence](media/image17.png)
   ![Boot sequence](media/image15.png)
   ![Boot sequence](media/image16.png)

## Login to SMF Daughter Card via SSH

**Connection steps:**

1. Open your terminal application (such as Terminal on macOS/Linux, or PowerShell/PuTTY on Windows). Check the IP address:

   ```bash
   $ ifconfig
   ```

2. Run the SSH command using the appropriate user and management IP:

   ```bash
   $ ssh xilinx@<management-ip-address>
   Password: xilinx
   ```

   Accept the host key fingerprint if prompted, then enter your password or present your SSH private key.

   ![SSH login](media/image14.png)

   ```
   $ ssh xilinx@192.168.50.52 -X
   The authenticity of host '192.168.50.52 (192.168.50.52)' can't be established.
   ED25519 key fingerprint is SHA256:vrFwqkqWIfyDZ1S66aJb1gLy59wx3LakOqhnyQJEOhg.
   This key is not known by any other names.
   Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
   Warning: Permanently added '192.168.50.52' (ED25519) to the list of known hosts.
   xilinx@192.168.50.52's password: xilinx
   Welcome to PYNQ Linux, based on Ubuntu 22.04 (GNU/Linux 5.15.36-xilinx-v2022.2 aarch64)
   ```

## Running YOLO Object Detection

```bash
$ source /usr/local/share/pynq-venv/bin/activate
$ python ../src/yolov8n_pred.py
```

```python
# yolov8n_pred.py
import argparse
import cv2
import numpy as np
import matplotlib.pyplot as plt
from ultralytics import YOLO

ap = argparse.ArgumentParser()
ap.add_argument("-i", "--image", required=True)
args = vars(ap.parse_args())

image = cv2.imread(args["image"])

# Load a model
model = YOLO('yolov8n.pt')  # load an official model

# Predict with the model
results = model(image)  # predict on an image

# Process results list
for result in results:
    boxes = result.boxes        # Boxes object for bounding box outputs
    masks = result.masks        # Masks object for segmentation masks outputs
    keypoints = result.keypoints  # Keypoints object for pose outputs
    probs = result.probs        # Probs object for classification outputs
    result.save(filename='result.jpg')  # save to disk
```

![YOLO detection result](media/image12.png)

## Troubleshooting: Ultralytics Import Failure

If `python ../src/yolov8n_pred.py` fails while importing `ultralytics`, and the traceback ends with the following error inside `sympy`, the virtual environment is falling back to the system Python's `sympy` package instead of its own, and that system package has a corrupted or version-mismatched bytecode cache:

```
ValueError: bad marshal data (invalid reference)
```

Resolve the issue in the following order.

1. **Clear the corrupted bytecode cache for the system `sympy` package.**

   ```bash
   $ sudo find /usr/lib/python3/dist-packages/sympy -name "__pycache__" -exec rm -rf {} +
   ```

   Re-run the script. Python recompiles fresh `.pyc` files automatically, and the import error is typically resolved.

2. **If the import still fails, install `sympy` directly inside the venv** so it no longer falls back to the system copy.

   ```bash
   $ source /usr/local/share/pynq-venv/bin/activate
   $ pip install sympy --force-reinstall
   ```

3. **Confirm whether the venv was created with access to system-wide packages.**

   ```bash
   $ grep include-system-site-packages /usr/local/share/pynq-venv/pyvenv.cfg
   ```

   If this is set to `true` and system packages are not required, set it to `false` and reinstall the needed packages inside the venv to prevent the venv from reaching outside itself.

> **Note:** On PYNQ boards, corrupted `.pyc` files are commonly caused by an unclean shutdown or a filesystem that ran out of space mid-write. If this issue recurs, also check available disk space:
>
> ```bash
> $ df -h
> ```
