## InkTime - Smartwatch Project


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

## Bill Of Materials(BOM)
| Designator | Component | Package | Manufacturer | Description / Notes |
| :--- | :--- | :--- | :--- | :--- |
| **U1** | **nRF52840** | aQFN-73 | Nordic Semi | SoC Bluetooth 5.0 (C190794) |
| **IC1** | **BQ25180Y** | DSBGA-8 | Texas Inst. | Charger IC Li-Ion (C3682423) |
| **IC2** | **DRV2605Y** | DSBGA-9 | Texas Inst. | Haptic Driver for ERM/LRA (C527464) |
| **IC9** | **RT6160AWSC** | WLCSP-15 | RICHTEK | Buck-Boost Regulator 3.3V (C7065276) |
| **IC3** | **BMA423** | LGA-12 | BOSCH | Triaxial Accelerometer (C5242966) |
| **U3 / IC6** | **MAX17048** | TDFN-8 | Maxim/ADI | 1-Cell Fuel Gauge w/ ModelGauge (C2682616) |
| **ANT1** | **2450AT18B100E** | 1.4mm | JOHANSON | 2.45GHz Chip Antenna |
| **J1** | **503480-2400** | FPC-24 | Molex | FPC Connector 0.5mm RA SMT |
| **J4** | **KH-TYPE-C-16P** | SMT | Kinghelm | USB Type-C 16-pin Connector |
| **Q3** | **SI1308EDL** | SOT-323 | Vishay | N-Ch 30V 1.5A TrenchFET |
| **D3** | **USBLC6-2** | SOT-23-6 | STMicro | ESD Protection (C7528806) |
| **X1** | **32MHz** | 2016 | Generic | High-Frequency Crystal |
| **X2** | **32.768kHz** | 2012 | Generic | Low-Frequency Crystal |


## Hardware Implementation Details

### Energy Management Strategy
To ensure maximum runtime, the system implements a **dual-stage power topology**. The **BQ25180** manages the LiPo charging cycle and provides a system voltage (**VSYS**), which is then regulated to a stable **3.3V** by the **RT6160 Buck-Boost**. The use of a Buck-Boost instead of a standard LDO allows the system to remain operational even as the battery voltage drops significantly below 3.3V.

### Display Power Gating
The E-paper display is inherently energy-efficient as it only draws power during image refreshes. However, to eliminate leakage current during sleep cycles, a **PFET high-side switch (SI2301)** is used. The MCU physically disconnects the display's power rail (**VEPD**) between updates via a dedicated GPIO signal.

### Haptic Feedback and Sensing
Unlike simple vibration circuits, the **DRV2605L** driver provides sophisticated haptic patterns. It controls an **ERM motor** via I2C commands. Motion tracking is handled by the **BMA421**, which monitors steps and orientation in the background, waking the nRF52840 only when necessary via hardware interrupts.

---

## Pinout Mapping (nRF52840)

### Communication Buses
| Bus Type | Pins | Description |
| :--- | :--- | :--- |
| **I2C (SDA/SCL)** | `P0.06` / `P0.07` | Shared by Charger, Fuel Gauge, Accel, and Haptic Driver. |
| **SPI (Display)** | `P0.02`, `P0.03`, `P0.05`, `P0.15`, `P0.16` | SCK, MOSI, CS, DC, and RST for E-paper control. |

### Signal and Control Lines
| Function | Pin | Description |
| :--- | :--- | :--- |
| **Display Status** | `P0.17` | BUSY pin — Monitors E-paper refresh state. |
| **Power Control** | `P1.01` | PFET Enable — Toggles display power rail. |
| **Haptic Enable** | `P0.12` | DRV2605L EN — Activates the haptic driver. |
| **BMA INT** | `P0.08` | Accelerometer hardware interrupt. |
| **MAX ALRT** | `P0.10` | Fuel Gauge low battery alert. |
| **BQ INT** | `P0.11` | Charger state change interrupt. |



 
