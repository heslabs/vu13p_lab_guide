# EPS-VU13P Quick Start Guide

<img width="421" height="240" alt="image" src="https://github.com/user-attachments/assets/25c95ad9-8d3f-4d80-b099-81b8fc0a735e" />

---
## Setup and Configure the ESP-VU13P

#### 1 Launch E-CoreFlow
Make the binary executable, then launch it with the X11 backend forced (helps avoid display issues on some systems): 
```
chmod 777 E-CoreFlow GDK_BACKEND=x11 ./E-CoreFlow >/dev/null 2>&1 &
```

#### 2 Select the board
In the E-CoreFlow UI, choose Select Board -> VU13P (or VU19P, depending on your hardware).

<img width="366" height="124" alt="image" src="https://github.com/user-attachments/assets/a2954038-4c9e-4956-9018-e10ae0918679" />


#### 3 Open your configuration project file
Go to File -> Open and load your *.bmprj configuration file.


#### 4 Set the LAN connection type
Navigate to System -> Lan Select System -> USB Ethernet to configure the board to communicate over the USB Ethernet interface. Scan the network and connect to the HAPS.

<img width="433" height="86" alt="image" src="https://github.com/user-attachments/assets/30b3702b-5dbb-4ca4-9872-d07bed216d0a" />
<br>

<img width="539" height="256" alt="image" src="https://github.com/user-attachments/assets/1f533d15-424e-434a-8f0f-1ea8fa0cfd2c" />


#### 5 Set the host's fixed IP address
In the same LAN settings dialog, set: 
```
Address: 192.168.1.100
Netmask: 255.255.255.0
```
This gives your host machine a static IP on the same subnet as the board.


#### 6 Power on the HAPS
Once the connection is established, power on the HAPS unit.


#### 7 Configure the System and FPGA
Configure the system via virtual JTAG — this takes a few seconds to complete. Then confirm the bit file to be used for FPGA configuration.


---
## Connect the SMF (CPU) module

#### 1 Connect the SMF daughter card via USB-UART bridge
Use a USB Type-A to Micro-USB cable: plug the Type-A end into a free USB port on your host PC, and the Micro-USB end into the SMF daughter card's UART port. Let the PC auto-detect the device and install drivers if needed.

#### 2 Open a serial terminal (PuTTY)
Set the device permissions and identify the port, then launch PuTTY configured for serial communication at 115200 baud: 
```
sudo chmod 666 /dev/ttyUSB*;
ls /dev/ttyUSB*
putty -serial -sercfg 115200,8,n,1,N /dev/ttyUSB0 -fn "client:Ubuntu Mono 16" &
```

In PuTTY:
```
Connection Type = Serial,
Serial Line = /dev/ttyUSB0 (or your port),
Speed = 115200.
```

#### 3 Power on the card and watch it boot
Power on the SMF daughter card. You'll see the boot sequence in the terminal:

* Xilinx ZynqMP First Stage Boot Loader, then
* U-Boot 2022.01 (CPU/DRAM/PMUFW info), then
* PynqLinux (Ubuntu 22.04) finishing at a login prompt.
* Default login is xilinx / xilinx.

<br>
<img width="580" height="431" alt="image" src="https://github.com/user-attachments/assets/fd4397d1-6871-4207-9a0e-a4f7d7ccf9ea" />

<br>
<img width="640" height="389" alt="image" src="https://github.com/user-attachments/assets/00ea3e49-4866-4b63-a992-2d130ebdb6d9" />


#### 4 Find the card's IP address
Once logged in at the serial console (or from another machine on the same network as the card), run: ifconfig Note the management IP address assigned to the board.

#### 5 SSH into the SMF daughter card
From your terminal: 
```
ssh xilinx@<management-ip-address>
Password: xilinx
```
Accept the host key fingerprint if prompted.

#### 6 Run YOLO object detection
Once connected via SSH, activate the PYNQ virtual environment and run the detection script: 
```
source /usr/local/share/pynq-venv/bin/activate
python ../src/yolov8n_pred.py
```


<br>
<img width="578" height="346" alt="image" src="https://github.com/user-attachments/assets/5296e16e-0606-4efd-8179-24fdfa135b75" />



#### Python sample code
```
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
    boxes = result.boxes  # Boxes object for bounding box outputs
    masks = result.masks  # Masks object for segmentation masks outputs
    keypoints = result.keypoints  # Keypoints object for pose outputs
    probs = result.probs  # Probs object for classification outputs
    result.save(filename='result.jpg')  # save to disk
```

