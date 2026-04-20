## InkTime - Smartwatch Project
Open-source smartwatch hardware project based on the Nordic nRF52840 MCU, 
featuring a 1.54" e-paper display and optimized for ultra-low power consumption. 

## Block Diagram
```text
[ USB Type-C ]     [ LiPo Battery (250mAh) ]
           | 5V VBUS          |
           v                  | BAT
    +---------------+         v
    |    BQ25180    |<--+ [MAX17048]
    |  (Charger IC) |   | (Fuel Gauge)
    +---------------+   |     |
           | VSYS       +-----+ I2C
           v                  v
    +---------------+      +-------------+
    |    RT6160     |      |             |
    | (Buck-Boost)  |      |  nRF52840   |
    +---------------+      |     MCU     |
           | 3.3V (VDD_3V3)|  (Control   |
           | System Rail   |    Hub)     |
           |               |             |
 +---------+---------------+------+------+
 |         |               |      |
 |         v               |      |
 |  +-------------+        |      | SPI
 |  | PFET Switch |<--GPIO-+      v
 |  +------+------+        |  +-------------+
 |         | VEPD          |  | 1.54" E-paper|
 |         v               |  |   Display   |
 |  +-------------+        |  +-------------+
 |  | [Display]   |        |
 |  +-------------+        | I2C Bus
 |                         |
 +---I2C==> [ BMA421 Accel ]
 |                         |
 +---I2C==> [ DRV2605L Driver ] --> [ Vibration Motor ]
 |
 |          [ PCB Antenna ] <--> [ BLE RF ]
 |
 +---GPIO-> [ User Buttons ]
```

## Bill Of Materials (BOM)

| Designator | Component | Package | Manufacturer | Description / Notes |
| :--- | :--- | :--- | :--- | :--- |
| **U1** | [**nRF52840**](https://www.nordicsemi.com/Products/nRF52840) | aQFN-73 | Nordic Semi | SoC Bluetooth 5.0 (Procesor principal) |
| **IC1** | [**BQ25180Y**](https://www.ti.com/product/BQ25180) | DSBGA-8 | Texas Inst. | Charger IC Li-Ion (Management baterie) |
| **IC2** | [**DRV2605Y**](https://www.ti.com/product/DRV2605) | DSBGA-9 | Texas Inst. | Haptic Driver (Control motor vibrații) |
| **IC9** | [**RT6160AWSC**](https://www.richtek.com/Products/Product%20List/RT6160) | WLCSP-15 | RICHTEK | Buck-Boost Regulator 3.3V |
| **IC3** | [**BMA423**](https://www.bosch-sensortec.com/products/motion-sensors/accelerometers/bma423/) | LGA-12 | BOSCH | Accelerometru (Detectie miscare) |
| **U3 / IC6** | [**MAX17048**](https://www.analog.com/en/products/max17048.html) | TDFN-8 | Maxim/ADI | Monitorizare nivel baterie (Fuel Gauge) |
| **ANT1** | [**2450AT18B100E**](https://www.johansontechnology.com/datasheets/2450AT18B100E.pdf) | 1.4mm | JOHANSON | Antena Chip 2.45GHz |
| **J1** | [**503480-2400**](https://www.molex.com/en-us/products/part-detail/5034802400) | FPC-24 | Molex | Conector ecran (0.5mm pitch) |
| **J4** | [**KH-TYPE-C-16P**](https://www.kinghelm.com.cn/product-item-1589.html) | SMT | Kinghelm | Port USB Type-C |
| **Q3** | [**SI1308EDL**](https://www.vishay.com/en/product/71996/) | SOT-323 | Vishay | MOSFET N-Ch (Comutare putere) |
| **D3** | [**USBLC6-2**](https://www.st.com/en/protection-devices/usblc6-2sc6y.html) | SOT-23-6 | STMicro | Protectie ESD linii date USB |
| **X1, X2** | **Crystal** | 2016 / 2012 | Generic | Cuart 32MHz (Sistem) / 32.768kHz (RTC) |
| **L2** | **10µH** | 0402 | Generic | Inductor critic pentru circuitul de boost e-Paper |
| **R1_EP_DR** | **0.47 Ω** | 0201 | Generic | Rezistenta de sens (RESE) pentru ecran |
| **R5, R7, R8** | **10k Ω** | 0201 | Generic | Rezistente Pull-up (I2C / Linii control) |
| **C1-EP-DR, C39** | **10µF** | 0201 | Generic | Condensatori filtrare alimentare |
| **C1, C2, C17, C18** | **12pF** | 0201 | Generic | Condensatori pentru cuart (X1, X2) |
| **C2-EP-DR** | **4.7µF / 25V** | 0201 | Generic | Stocare energie tensiuni mari ecran (VGH/VGL) |
## Hardware Implementation Details

### Energy Management Strategy
To ensure maximum runtime, the system implements a **dual-stage power topology**. The **BQ25180** manages the LiPo charging cycle and provides a system voltage (**VSYS**), which is then regulated to a stable **3.3V** by the **RT6160 Buck-Boost**. The use of a Buck-Boost instead of a standard LDO allows the system to remain operational even as the battery voltage drops significantly below 3.3V.

### Display Power Gating
The E-paper display is inherently energy-efficient as it only draws power during image refreshes. However, to eliminate leakage current during sleep cycles, a **PFET high-side switch (SI2301)** is used. The MCU physically disconnects the display's power rail (**VEPD**) between updates via a dedicated GPIO signal.

### Haptic Feedback and Sensing
Unlike simple vibration circuits, the **DRV2605L** driver provides sophisticated haptic patterns. It controls an **ERM motor** via I2C commands. Motion tracking is handled by the **BMA421**, which monitors steps and orientation in the background, waking the nRF52840 only when necessary via hardware interrupts.

---

## Pinout Mapping (nRF52840)

### Shared I2C Bus
Used by the IMU (BMA421), Fuel Gauge (MAX17048), Li-Po Charger (BQ25180), Haptic Driver (DRV2605), and DC/DC Converter (RT6160).
* **P0.16** — SDA (Serial Data)
* **P0.14** — SCL (Serial Clock)

### E-Paper Display (SPI Interface)
Connected via the 24-pin FPC connector (J1).
* **P0.17** — EPD_SCK (Serial Clock)
* **P0.22** — EPD_MOSI (Serial Data)
* **P0.20** — EPD_CS (Chip Select)
* **P0.13** — EPD_DC (Data/Command Selection)
* **P0.24** — EPD_RST (Hardware Reset)
* **P0.11** — EPD_BUSY (Panel Status Input)

### Peripheral Interrupts & Control
* **P0.15** — CHG_INT (Charger state/interrupt)
* **P0.25** — IMU_INT1 (Primary motion interrupt)
* **P0.26** — IMU_INT2 (Secondary motion interrupt)
* **P0.06** — HAPTIC_INT (Haptic driver control/interrupt)
* **P0.08** — FUEL_ALERT (Battery fuel gauge alert)

### Physical Buttons (Internal Pull-up, Active-Low)
* **P1.11** — BTN_UP (Up Navigation)
* **P1.13** — BTN_ENT (Enter / Select)
* **P1.15** — BTN_DN (Down Navigation)

### Debug & System Clocks
* **XC1/XC2 & XL1/XL2** — External 32MHz and 32.768kHz crystals for system and RTC stability.
* **SWDIO, SWDCLK, SWO, nRESET** — Routed to the Tag-Connect TC2030 debug interface for firmware flashing and real-time debugging.

# Design Log

## Schematic Design

- **nRF52840** (aQFN-73) selected as main MCU, providing Bluetooth 5.0, native USB and sufficient GPIOs for all peripherals.
- **Dual-stage power topology**: BQ25180YBGR handles LiPo charging, RT6160AWSC Buck-Boost regulates the 3.3V system rail — allowing operation even as battery drops below 3.3V.
- **MAX17048** fuel gauge connected on shared I2C for accurate battery level estimation.
- **BMA421** triaxial accelerometer on I2C with two hardware interrupt lines to the MCU.
- **2450AT18B100E** chip antenna with discrete matching network for 2.4GHz BLE.
- Dedicated e-paper drive circuit (MBR0530, SI1308EDL, boost inductor) generating PREVGH/PREVGL voltages for the 1.54" panel via a 24-pin Molex FPC connector.
- **DRV2605YZFR** haptic driver for ERM motor, controlled via I2C.
- **USBLC6-2SC6Y** ESD protection on USB lines, KH-TYPE-C-16P connector with 5.1kΩ CC resistors for power delivery negotiation.
- Three tactile buttons (UP, DOWN, ENTER) with 10kΩ pull-ups.
- **TC2030-IDC** SWD connector with dedicated test points for programming and debug.

---

## PCB Design
- Smartwatch form factor sized for wrist-wear with a dedicated enclosure.
- Antenna and matching network placed at PCB edge with no copper pour underneath, per Nordic's nRF52840 layout guidelines.
- 100nF decoupling capacitors placed close to each nRF52840 power pin (DEC1–DEC6).
- Manufacturing files (Gerber, BOM, CPL) available in `/Manufacturing`.

---

## 3D Design

- Enclosure designed to house the PCB, 250mAh LiPo and 1.54" e-paper display.
- 3D assembly model available in `/Images` for fit and clearance verification.
- USB Type-C and side buttons (UP, DOWN, ENTER) accessible without disassembly.


 
