
## Details

## Target
Manufacturer: TP-Link

Part Number: TL-WR841N

Serial Number: 22487Q7006381

## Equipment
Multimeter

Logic Analyzer: AZDelivery Logic Analyzer

USB to UART Adapter: Hailage CP2102

Flash ROM Programmer: APKLVSR CH341A

Environment: VMWare Kali Linux

Software:
* Sigrok Pulsview
* NMAP
* Firefox

## Initial Recon

### 1. Visual Inspection
Inspection of the external plastic cover of the router revealed a label that contained the FCC ID of the device

FCC ID: 2AXJ4WR841NV14

![Label](https://github.com/user-attachments/assets/d6d04eba-7be9-4cf4-bfe2-7bc98678748d)
Image 1. Label of the router

![Motherboard_Labeled](https://github.com/user-attachments/assets/5022a262-95de-4ad9-89e9-8cc9715de46d)
Image 2. Picture of motherboard


### 2. On Board Testing
By inspecting the motherboard, it can be noted that 4 through holes labeled VCC, GND, RX and TX can be found on the top right corner of the PCB shown as label E in Image 2. This would most likely indicate an UART connection.

The operating voltage was measured as 9V by measuring the voltage drop across the input jack using a multimeter.

A ground connection P1 was found shown as label B.

The UART connection was verified by taking the following steps:
* VCC: Measured the voltage drop between ground and the VCC as 3.3V
* GND: Verified that the GND pin is connected to ground by checking connectivity between GND and P1 (label B) using a multimeter
* TX: Measured voltage drop between ground and TX  as 3.3V during standard operation
* RX: Measured voltage drop between ground and RX as 0V. Additionally tested connection to ground. No ground connection was found

After confirming the UART connection the voltage drop between ground and TX was measured during bootup. The voltage drop was measured varying between 1v-3.3V indicating active data transmission.

Additional pins were soldered to the UART connection to allow easier access to the pins.

The bootup data transmission were caputred using a logic analyzer

![image](https://github.com/user-attachments/assets/b3a65311-4803-4b9c-a5b2-19bb30c47e97)
Image 3. Screenshot of the capture from PulseView.

The baud rate was measured as 125 kHz. However, when compared to standard baud rates, it is likely the real rate is actually 115200.

An UART decoder was applied based on the measurements.

![image](https://github.com/user-attachments/assets/8c3d6a15-bea9-4398-b17d-e14ecd54fdc2)

Image 4. Screenshot or UART decoder settings

### 3. OSINT and Online Recone
Using the previously obtained FCC ID a lookup was done through fccid.io [https://fccid.io/2AXJ4WR841NV14](https://fccid.io/2AXJ4WR841NV14).

The documents show that the previous FCC ID before a change was TE7WR841NV14

![image](https://github.com/user-attachments/assets/c8f0699b-01d9-4c39-8e4c-249d934beb24)
Image 5. Screenshot of the FCC filing indicating changed FCC ID

Lookup using the old FCC ID revelas detailed photos of the components. The circuit diagram, block diagram and operational description are marked as confidential and not visible to the public.

![image](https://github.com/user-attachments/assets/9c5e9699-55d3-4517-9e18-b50c02440a46)
Image 6. Detailed pictures of chips

Chip A from Image 2 identified as Zentel A3S56D40GTP -50L.

Chip C from Image 3 identfied as MEDIATEK MT7628NN.

Details of chip D could not be found from the documents.

The chip was identified by taking a detailed picture of the physical device.
![image](https://github.com/user-attachments/assets/2632655c-3e3b-467b-9794-40bc7026c3d8)

Image 7. Picture of chip D

Chip D was identified as cFeon QH32B-104HIP.

Data sheets were located online for each of the chips.

MEDIATEK MT7628NN: System On Chip containing CPU. Ppurpose-built SOC for N300 routers. The CPU is MIPS24KEc, which supports Linux 2.6.36 SDK and Linux 3.10 SDK. Interfaces with flash memory via SPI.

Product Page: https://www.mediatek.com/products/home-networking/mt7628k-n-a

Data Sheet: https://files.seeedstudio.com/products/114992470/MT7628_datasheet.pdf

Zentel A3S56D40GTP -50L: DRAM 32Kb

Data Sheet: https://www.mouser.ca/datasheet/2/1130/DSA3S56D340GTPF_02-1984099.pdf

cFeon QH32B-104HIP: Id'd as full part number EN25Q32B-104HIP. Noted as Flash ROM that communicates via SPI.

Data Sheet: https://www.alldatasheet.com/datasheet-pdf/pdf/675622/EON/EN25QH32A.html

Searching for the firmware found it supplied through the official support page for the router.

https://www.tp-link.com/ca/support/download/tl-wr841n/#Firmware


### Initial Network Recon

After connecting to the network hosted by the router a nmap scan was run

![image](https://github.com/user-attachments/assets/f171c339-0045-4fa5-89af-b80b7206a3ec)

Image 8. Nmap TCP scan of the router at 192.168.0.1

The scan revelaed that ports 22 and 80 are open

![image](https://github.com/user-attachments/assets/2abf1d2d-211b-47bb-a99c-b22d9c21a5ec)

Image 9. nmap UDP scan of the router at 192.168.0.1

The UDP scan didn't show any clearly open ports.

### Previous CVEs
A common theme of previous CVEs is a lack of proper user input validation on forms and inputs in the web portal that led to either buffer overflow or command injection of an underlying function or service being called.

https://www.opencve.io/cve?vendor=tp-link&product=tl-wr841n

https://blog.viettelcybersecurity.com/1day-to-0day-on-tl-link-tl-wr841n/

https://ktln2.org/2020/03/29/exploiting-mips-router/


![image](https://github.com/user-attachments/assets/71f540d5-919e-49cf-b0af-b7cdfbf7a3e1)











