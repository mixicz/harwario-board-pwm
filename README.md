# Universal PWM board for Hardwario TOWER
There are many PWM controlled applications, from driving LED strips or fans to controlling servos. Purpose of this board is to allow for wide range of use cases, so you can get bunch of them professionally manufactured (which is usually much cheaper when you order more of them) and customize it to particular use just by fitting right components.

I had several requirements:
- low profile (I sometimes need to fit it in slim places),
- wide range of input voltage, at least 5-24V,
- able to handle decent current for driving LED strips,
- any output can be configured to GND, VCC or PWM,
- any core module GPIO output can be connected to any output (not all outputs support PWM),
- optional mini battery module including battery level monitoring (so you can keep communicating even after power outage),
- possibility to route battery level signals to non-PWM ADC inputs (P4, P5),
- proper handling of Hardwario TOWER power signals (so you can use USB to update firmware or debug without need of disconnecting PSU),
- optional use of Adafruit I2C PWM module instead of direct GPIO (https://www.adafruit.com/product/815).

For discussion regarding board design see Hardwario Forum topic https://forum.hardwario.com/t/universal-pcb-1-pwm/655

## Features
- 5-35V input,
- should be able to handle up to ~8A total or ~4A per output pair using 70µm copper (or ~5A total + ~3A per output for 35µm copper),
- all SMD components were selected large enough to be easily soldered by hand,

Physical dimensions 100×80 mm, holes 3.2 mm wide, placed 4 mm from each side.

## HW configuration
For power input and output use connectors or headers with 5mm spacing (and proper power rating according to your needs). In case your connectors does not fit side to side and you cannot leave unused output between, you can place them on both sides of board.

### Required components
These components needs to be fitted in all cases:
- input power terminal (on short side of the board),
- core module header (CORE) - you need to use socket header here to fit core module pin side,
- PSU section: AP1501 step down + **L1**, **D1**, **D4**, **C1**, **C2**
- Tower power control signals: **D3**, **R3** 

### Configuring general board features
1. If you want to use battery module, you need to fit pin headers to **BATT** and **D2**+**R2** for *BAT_OFF* signal. You may also use 0R SMD resistors **R4**+**R5** to allow for battery level measure (this is default assigment, you will loose 2 usable PWM pins though) or alternativelly use **R6**+**R7** to connect it to *P4*+*P5* pins (which does not have PWM capability).
2. If you want to monitor input power status (for example to send message on power outage and restore), you can use **R1** for this.

### Configuring outputs
Each output has place for four resistors (Rx1 - Rx4, where *x* is output number) and one SO-8 N-channel MOSFET transistor (IRF7413).

1. First assign function to each output according to your needs (I recommend to not use even-numbered transistors for high curent outputs in order to leave room for better vias cooling),
2. Decide which PWM outputs from core module to use (not all GPIO does support PWM - see https://tower.hardwario.com/en/latest/hardware/header-pinout/). GPIO are mapped to board P0-P15 starting from side with power input connector.
3. Connect desired core module PWM GPIO to particular outputs using **Rx1** (small resistor 0-390 ohm or just wire). You can generally connect any GPIO to any output, just be carefull to not short-circuit signals when you have uninsulated wires close together.
4. for each used output fit one of:
    - PWM: **Rx2** + transistor **Tx**
    - VCC: **Rx4**
    - GND: **Rx3**
    
### Using Adafruit I2C 16-Channel 12-bit PWM module
Alternatively, instead of directly routing core module GPIO, you can use [Adafruit I2C 16-Channel 12-bit PWM](https://www.adafruit.com/product/815) module. In such case, place 4× 4-pin socket header in **Rx1** holes closer to the outputs (it will fit to PWM outputs from the module) and 6-pin socket header to **JP3** position. Outputs are configured in same manner as mentioned above.

# Part list
## Required
| **component** | **part / value** | 
| ----------- | ----------- |
| step-down | AP1501 (TO263-5L) | 
| C1 | 10µF 16V |
| C2 | 10µF 35V |
| L1 | 33µH (THT) |
| D1 | B340A (SMA) |
| D3, D4 | PMLL41* or similar (SOD80C) diode |
| R3 | 560R (R3216 EU) |
| Rx2 | N× 10k (R3216 EU) |
| Tx | N× IRF7413 (SO8) |
| JP1 (input) | 2-wire 5.08 mm spaced connector / terminal |
| JP2* (outputs) | N× 5.08 mm spaced connector / terminal as needed |
| header - core module | 14-pin, 8-pin and 5-pin 1-row socket header (2.54 mm spaced) |
| core module | [Hardwario TOWER core module](https://shop.hardwario.com/core-module/)

## Optional
| **component** | **part / value** | 
| ----------- | ----------- |
| Rx1 | wire or 0-390R THT resistor |
| R4-7, Rx3, Rx4 | 0R (R3216 EU) |
| R1, R2 | 560R (R3216 EU) |
| D2 | PMLL41* or similar (SOD80C) diode |
| header - battery module | 14-pin, 8-pin and 5-pin 1-row socket header (2.54 mm spaced) |
| mini battery module | [Hardwario TOWER mini battery module](https://shop.hardwario.com/mini-battery-module/)
| header - PWM module | 1× 6-pin + 4× 4-pin socket header (2.54 mm spaced) |
| PWM module | [Adafruit I2C 16-Channel 12-bit PWM](https://www.adafruit.com/product/815)

# Known issues
## v 1.0
- Battery module needs higher header (at least 6mm) because of:
    - C1 is too close to battery module header (it can be placed on the other side as workaround),
    - R142, R162 are right below battery connector on module, risking short circuit,
- L1 has too small footprint for most inductors available,
- signal paths are only 0.3mm wide which complicates manufacturing with thicker Cu plating (70 µm or more).
