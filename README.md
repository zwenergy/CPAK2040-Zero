# CPAK2040-Zero
An RP2040-based controller pak for the N64 in a small form factor.

<img src="./assets/cpak2040_zero.png" alt="drawing" width="600"/>

## Overview
The main idea of CPAK2040 is to a have a simple, cheap and easily DIY-able controller pak alternative for the N64 which is non-volatile (meaning not relying on a battery to keep the controller pak data like the original based on SRAM). 
For this, an RP2040-Zero board is used, as it is widely available, cheap and easy to solder even for beginners.

A regular RP2040-Zero board has a 2 MB Flash, whereas a single N64 controller pak has a capacity of 32 kB.
The software running on the RP2040 to emulate the controller pak only takes up a fraction of the available Flash memory, hence quite many "virtual controller paks" can be stored in the Pico's Flash.
By default, the CPAK2040 holds 10 "virtual controller paks", as a more efficient navigating method needs to implemented to better use a larger number of controller paks.

**DO NOT CONNECT VIA USB WHEN PLUGGED IN A CONTROLLER!**

## BOM
Well, the BOM is short.
You need the **PCB**, an **RP2040-Zero board** and a [**tactile switch**](https://www.lcsc.com/product-detail/Tactile-Switches_XKB-Connection-TC-1101DE-C-F_C561490.html). 

## PCB
The Gerber files to create the PCB can be found in the "gerber" folder.
You need to order the PCB with **1.2 mm thickness**, **ENIG or hard gold** and **bevelled edges** at the connector.
Using regular HASL is not recommended, as the PCB contacts quickly wear down, while solder may also rub onto the controller's connection pins.

## Soldering up the RP2040-Zero Board
We do not only make use of the RP2040-Zero board's castellated edges, but also the small contact pads on the back.
These pads are rather small, so make sure to align the board correctly, such that the VIAs of the PCB line up with the pads.
It can also be helpful to pre-tin the RP2040-Zero's additional pad with a **thin** layer before actually soldering the pads via the through-holes.

<img src="./assets/backsolder.jpg" alt="drawing" width="600"/>


## Setting up the RP2040-Zero
Hold down the BOOT button, connect the RP2040-Zero via USB to your computer and drag'n'drop the .UF2 from the latest release onto the RP2040-Zero device.

Once flashed with the FW, you can also use the additional switch of the PCB as a BOOT button to update the firmware, so that you do not need to open the case.

## Using the CPAK2040
If you're using the CPAK2040 without the 3D printed case, make sure that you orientate the PCB correctly.

Initially, all virtual controller paks will be "corrupted", as they are not formatted as a controller pak data.
Start a game using a controller pak and hold START during the startup, this will bring up the controller pak manager.
Here, you can typically "repair" aka format the current virtual controller pak.

### Storing behaviour
The current controller pak's content is hold in the RP2040's SRAM.
When a write the controller pak happens, the updated contents are stored into the Flash.
In order to avoid unneccassary writes to the Flash, the Flash content is not directly updated, but a small timeout of some seconds is introduced, such that multiple writes can be combined into a single Flash rewrite.
This is signaled using the RP2040-Zero's LED.
When then LED is on, this means that there is fresh data in the controller pak which is not written back to the Flash yet.
**Do not unplug the controller pak, while the LED is on, or otherwise you risk losing the latest controller pak data or data corruption.**

This delayed storing behaviour is done to reduce the number of Flash rewrites.
The Flash has a limited number of writes of ca. 100,000 before it wears out.
**This also means, that each "virtual controller pak" on the CPAK2040 has a lifetime of around 100,000 writes.**
Realistically, this should be enough forever for most people I guess.
If you save each day 10 times to the controller pak, you can do that for 27 years straight before you're getting close the 100,000 writes.

### Changing Virtual Controller Pak
To change the current virtual controller pak (VCP), shortly the BOOTSEL button down (<1 second).
The CPAK2040 will jump to the next VCP, indicating the current index by flashing the LED.
After the last VCP, the index will jump back to the first.
The current VCP index is stored into the Flash after 2s not changing it, so also after unplugging the CPAK2040 it will remember the last VCP used.

Note that many games will not reload the controller pak content, unless they sense that the controller pak is physically removed and plugged back in again.

The simple "hold button to increase VCP index" is not ideal, especially with a larger number of VCPs.

### Backup Controller Pak Data
When plugging in the CPAK2040 via USB to a computer, press the BOOTSEL button or the additional switch long (>2 seconds) and let go (**AFTER** plugging in via USB).
This enables USB read mode, which registers the CPAK2040 as a mass storage device and all the controller paks can be copied off the device (this is implemented using TinyUSB).
There is currently no way to manually exit the USB mode again, so the CPAK2040 needs to be unplugged before used again normally.

### Writing Controller Pak Data
This is currently still in **experimental mode**.

**ALWAYS MAKE A BACKUP OF YOUR SAVE GAMES!**

To enter USB write mode, hold the BOOTSEL or the additional switch button long (>2 seconds), let go shortly, then again hold it long (**AFTER** plugging in via USB).
This enables USB write mode, which registers the CPAK2040 as a mass storage device.
In this mode, only the current chosen VCP is visible and named "MEMPAK.MPK".
In order to replace this VCP, rename the memory pak file to be written to the CPAK2040 also to "MEMPAK.MPK" and copy it to the CPAK2040 drive (so it replaces the file on the drive).
After writing, the CPAK2040 should automatically disconnect and restart, showing the current chosen VCP via the blinking LED.


## Shell
A first version of a 3D printable shell can be found [here](https://www.printables.com/model/1255674-cpak2040-zero-shell).
It's a remix of JTG_16's shell for FRAM controller pak PCBs which can be found [here](https://www.thingiverse.com/thing:6784987).

## Disclaimer
**Use the files and/or schematics to build your own board at your own risk**.
This board works fine for me, but it's a simple hobby project, so there is no liability for errors in the schematics and/or board files.
**Use at your own risk**.
