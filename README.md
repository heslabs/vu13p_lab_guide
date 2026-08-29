# EPS-VU13P Quick Start Guide

#### 1 Launch E-CoreFlow
Make the binary executable, then launch it with the X11 backend forced (helps avoid display issues on some systems): 
```
chmod 777 E-CoreFlow GDK_BACKEND=x11 ./E-CoreFlow >/dev/null 2>&1 &
```

#### 2 Select the board
In the E-CoreFlow UI, choose Select Board -> VU13P (or VU19P, depending on your hardware).


#### 3 Open your configuration project file
Go to File -> Open and load your *.bmprj configuration file.


#### 4 Set the LAN connection type
Navigate to System -> Lan Select System -> USB Ethernet to configure the board to communicate over the USB Ethernet interface. Scan the network and connect to the HAPS.


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
