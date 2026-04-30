# 📦 Sample — Building Automation Fieldbusses

---

## 🧠 Remarks and limitations

- This is a **sample project** demonstrating how to use various building automation fieldbus protocols with Beckhoff TwinCAT 3.
- Minimum TwinCAT version: **3.1.4024.74**
- Each fieldbus protocol lives in its own PLC project within a single TwinCAT solution. They can be enabled or disabled independently.
- The DIOC (Fixsus) sub-project requires the third-party **DiocLibrary** from Upzio, which must be installed separately.
- BACnet Rev12 contains only an empty MAIN program and serves as a placeholder configuration.
- Hardware terminals (KL6301, KL6581, KL6781, KL6821, KL6831, EL6821, EL6851) must be present and configured in the I/O tree for the corresponding sub-project to function.

---

## 🔧 Functionality Overview

### `BACnet_Rev14`
Demonstrates BACnet/IP communication using the TwinCAT 3 BACnet Rev14 library. Creates a hierarchical BACnet object tree with buildings (Office, Production Plant, Warehouse), levels, and rooms, each containing analog input (AI) objects for temperature readings and analog value (AV) objects for setpoints.

- Uses `FB_BACnet_Adapter` for adapter state, `FB_BACnet_View` for hierarchy nodes, `FB_BACnet_AI` / `FB_BACnet_AV` for data points
- Requires a BACnet I/O Device configured in the TwinCAT project

### `DIOC_Fixsus`
Controls a TP10 DIOC touch panel (Upzio/Fixsus) with LEDs, buttons, and environmental sensors (temperature, humidity, CO2, air quality).

- Uses `FB_TP10` from the third-party DiocLibrary
- Requires `{attribute 'TcCallAfterOutputUpdate'}` and a 12 ms task cycle

### `EL6821_Tc3DALI`
DALI lighting control via the EL6821 EtherCAT terminal using the TwinCAT 3 DALI library.

- Uses `FB_EL6821Communication` for terminal communication and `FB_DALI102DirectArcPowerControl` for dimming
- Sets DALI address 10 to power level 127 (mid-range) as an example

### `EL6851_DMX`
DMX512 master and slave communication via the EL6851 terminal for stage/architectural lighting.

- **Master** (`P_DMX_Master`): sends up to 512 channels of DMX data using `FB_EL6851CommunicationEx`
- **Slave** (`P_DMX_Slave`): receives DMX channel data (default 64 channels, configurable)

### `KL6301_EIB_KNX`
KNX/EIB communication via the KL6301 Bus Terminal for home and building automation.

- Uses `KL6301` FB with physical address configuration and group address filtering (up to 8 filters)
- Demonstrates bit-level send/receive with `FB_EIB_BIT_SEND` / `FB_EIB_BIT_REC` and multiple trigger modes (manual, polling, on-change)
- Includes helper function `F_EIB_GroupAddress` to construct group addresses from main/sub/number

### `KL6581_EnOcean`
EnOcean wireless sensor/switch communication via the KL6581 Bus Terminal.

- Uses `FB_KL6581` for terminal communication and `FB_Rec_RPS_Switch` to decode 4-pushbutton rocker switch telegrams
- Supports up to 64 KL6581 terminals

### `KL6781_MBus`
M-Bus (Meter-Bus) communication via the KL6781 Bus Terminal for reading energy meters and utility meters.

- **Standard** (`P_MBus`): uses `FB_MBUS_EMU_32x7` to read operating hours, energy, power, and error status from a meter
- **Fast** (`P_MBus_Fast`): uses `FB_MBUSKL6781` with global I/O variables for higher-speed polling (2 ms task)

### `KL6821_Tc2DALI`
DALI lighting control via the KL6821 Bus Terminal using the older TwinCAT 2 DALI library.

- Uses `FB_KL6821Config` for digital input event configuration (watchdog, DI1/DI2 edges) and `FB_DALIV2DirectArcPowerControl` for dimming
- Demonstrates configuration of the KL6821 command buffer

### `KL6821_Tc3DALI`
DALI lighting control via the KL6821 Bus Terminal using the TwinCAT 3 DALI library.

- Uses `FB_KL6821Communication` with embedded initialization link and `FB_DALI102DirectArcPowerControl`
- Simplified setup compared to the Tc2 variant

### `KL6831_SMI`
SMI (Standard Motor Interface / Somfy) control for motorized blinds and screens via the KL6831 Bus Terminal.

- Uses `FB_KL6831KL6841Config` for digital input assignment, `FB_SMIDown` / `FB_SMIUp` for motor control, and `FB_SMIDiagAll` for diagnostics
- Supports priority-based Up/Down commands on DI1/DI2

### `Modbus_RTU`
Modbus RTU serial communication in both master and slave roles.

- **Master** (`P_ModbusRTU_Master`): sequential state machine reading input/holding registers and writing to slave devices via `ModbusRtuMasterV2_PcCOM` (also supports KL6x5B/KL6x22B variants)
- **Slave** (`P_ModbusRTU_Slave`): exposes input registers (256 words), output registers (256 words), and memory area (2048 words) at device address 17

### `Modbus_TCP`
Modbus TCP/IP master communication over Ethernet.

- Uses `FB_MBReadInputs` and `FB_MBWriteRegs` to communicate with a Modbus TCP slave at a configurable IP address and port
- Demonstrates reading input registers and writing holding registers

### Task architecture

| Task | Cycle | Priority | Runs |
|------|-------|----------|------|
| PlcTask_EIB_KNX | 10 ms | 20 | KL6301_EIB_KNX |
| PlcTask_Tc3DALI | 10 ms | 21 | KL6821_Tc3DALI |
| PlcTask_EnOcean | 10 ms | 22 | KL6581_EnOcean |
| PlcTask_SMI | 10 ms | 23 | KL6831_SMI |
| PlcTask_DMX | 10 ms | 24 | EL6851_DMX |
| PlcTask_Tc2DALI | 10 ms | 25 | KL6821_Tc2DALI |
| PlcTask_BACnetRev12 | 10 ms | 26 | BACnet_Rev12 |
| PlcTask_BACnetRev14 | 10 ms | 27 | BACnet_Rev14 |
| PlcTask_ModbusRTU | 10 ms | 28 | Modbus_RTU |
| PlcTask_ModbusTCP | 10 ms | 29 | Modbus_TCP |
| PlcTask_MBus | 10 ms | 30 | KL6781_MBus |
| PlcTask_MBus_Fast | 2 ms | 9 | KL6781_MBus (fast polling) |
| PlcTask_DIOC | 12 ms | 31 | DIOC_Fixsus |
| PlcTask (EL6821) | 10 ms | 32 | EL6821_Tc3DALI |

---

## 🧪 Example

<!-- BACnet Rev14 — initialize adapter and serve BACnet objects -->
```iecst
PROGRAM MAIN
VAR
END_VAR

P_BacnetRev14();
```

<!-- Modbus RTU Master — read 6 input registers from device address 10 -->
```iecst
PROGRAM P_ModbusRTU_Master
VAR
    fbModbus    : ModbusRtuMasterV2_PcCOM;
    Rcv         : ARRAY[0..100] OF WORD;
    Step        : INT := 0;
END_VAR

CASE Step OF
    0:  // Read 6 input registers from device 10, starting at register 0
        fbModbus.UnitID         := 10;
        fbModbus.Quantity       := 6;
        fbModbus.MBAddr         := 0;
        fbModbus.cbLength       := SIZEOF(Rcv);
        fbModbus.pMemoryAddr    := ADR(Rcv);
        fbModbus.ReadInputRegs  := TRUE;
        Step := 1;
    1:  // Wait for completion
        fbModbus.ReadInputRegs  := FALSE;
        IF NOT fbModbus.BUSY THEN
            Step := 0;
        END_IF
END_CASE

fbModbus();
```

Adapt each example by replacing device addresses, register maps, DALI short addresses, or group addresses with values matching your field installation.

---

## 🧠 Notes

- The DIOC sub-project requires the `TcCallAfterOutputUpdate` attribute on MAIN and a 12 ms task cycle for correct communication timing with the TP10 panel.
- The KL6301 EIB/KNX project uses a `GVL_IO` with 24-byte input/output arrays mapped to the terminal process data; the KL6781 M-Bus fast variant similarly uses a `GVL_IO` for shared I/O structures.
- Modbus RTU master code includes commented-out variants for KL6x5B and KL6x22B Bus Terminals — uncomment the appropriate FB type for your hardware.
- The BACnet Rev14 sub-project creates all BACnet objects dynamically at runtime via PLC code; the BACnet I/O Device must be configured in the TwinCAT I/O tree first.
- For M-Bus fast polling, the task runs at 2 ms (priority 9) to ensure timely serial communication with the KL6781 terminal.

---

## 🔢 Additional information

**Required libraries**

| Library | Vendor | Purpose |
|---------|--------|---------|
| Tc2_Standard | Beckhoff Automation GmbH | Standard PLC functions (all projects) |
| Tc2_System | Beckhoff Automation GmbH | System functions (all projects) |
| Tc3_Module | Beckhoff Automation GmbH | Module interface (all projects) |
| Tc3_BACnetRev14 | Beckhoff Automation GmbH | BACnet Rev14 protocol |
| Tc3_BA2_Common | Beckhoff Automation GmbH | BA2 common types for BACnet Rev14 |
| Tc2_DALI | Beckhoff Automation GmbH | DALI v2 protocol (KL6821 Tc2) |
| Tc3_DALI | Beckhoff Automation GmbH | DALI 102 protocol (KL6821/EL6821 Tc3) |
| Tc2_DMX | Beckhoff Automation GmbH | DMX512 protocol (EL6851) |
| Tc2_EIB | Beckhoff Automation GmbH | EIB/KNX protocol (KL6301) |
| Tc2_EnOcean | Beckhoff Automation GmbH | EnOcean wireless protocol (KL6581) |
| Tc2_MBus | Beckhoff Automation GmbH | M-Bus protocol (KL6781) |
| Tc2_SMI | Beckhoff Automation GmbH | SMI motor protocol (KL6831) |
| Tc2_ModbusRTU | Beckhoff Automation GmbH | Modbus RTU serial protocol |
| Tc2_ModbusSrv | Beckhoff Automation GmbH | Modbus TCP server/client |
| DiocLibrary | Upzio | DIOC TP10 touch panel (third-party) |
| Tc2_DataExchange | Beckhoff Automation GmbH | Data exchange (BACnet Rev14) |
| Tc2_IoFunctions | Beckhoff Automation GmbH | I/O functions (BACnet Rev14) |

**Supported platforms**: x64 (64-bit)

**Minimum TwinCAT version**: 3.1.4024.74

**License**: [BSD Zero Clause](LICENSE.md)
