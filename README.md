# Renesas RA6E2 + DA14531 BLE Sensor Demo — Complete README

## 1. Purpose

This document records the complete procedure used to build, flash, and test the Renesas QuickConnect Studio **Sensor Data over BLE** project on the **BGK-RA6E2** board.

The final working setup was:

**BGK-RA6E2 + DA14531MOD Bluetooth module + US082-14 + sensor + Quick-Connect Mobile Sandbox**

The project README says the RA MCU reads sensor data and publishes it through the DA14531/DA14535 BLE module. It also describes LED control, sensor visualization, and BLE operation.

---

## 2. Hardware

### Main board
- BGK-RA6E2 / RA6E2 MCU board
- MCU: Renesas **R7FA6E2BB**
- Core: **Cortex-M33**

### Wireless module for this project
- **DA14531MOD** — BLE/Bluetooth module

### Interposer/PMOD
- **US082-14**

### Other module in the kit
- **DA16600MOD** — Wi-Fi/BLE module
- **Important:** DA16600MOD was **not used for this particular `Sensor Data over BLE` project**.

---

## 3. Physical Arrangement

The physical arrangement was matched to the `.qcc` diagram:

```text
LEFT / PMOD1       RIGHT / PMOD2
DA14531MOD         US082-14
Bluetooth          PMOD/interposer
      \              //
       \            //
          BGK-RA6E2
```

Do not substitute the DA16600MOD for the DA14531MOD while running this BLE project.

---

## 4. Software

Used during the procedure:

- Renesas QuickConnect Studio
- SEGGER J-Link software
- SEGGER J-Link Commander
- SEGGER J-Link Configurator
- SEGGER J-Flash
- SEGGER J-Flash Lite
- PuTTY
- Quick-Connect Mobile Sandbox

The generated project README specifies:
- FSP 5.9.0
- FreeRTOS 11.1.0
- AzureRTOS 6.4.0
- SEGGER RTT 8.10
- Compiler optimization `-O2`
- C99

---

## 5. QuickConnect Studio Project

The project used was:

```text
first
```

The configuration file was:

```text
configuration/first.qcc
```

The generated project contained folders such as:

```text
Debug/
ra/
ra_cfg/
ra_gen/
src/
configuration/
```

Open the `.qcc` diagram and verify that the selected hardware matches the physical board.

---

## 6. Build the Project

1. Open the project in QuickConnect Studio.
2. Open/verify `first.qcc`.
3. Build the project.
4. Check the build output.

The build completed with:

```text
Build Finished. 0 errors, 0 warnings.
```

A generated ELF and SREC were present in `Debug/`.

---

## 7. Locate the SREC

The exact QCStudio/Linux path was:

```text
/home/qcstudio/workspace/first/Debug/first.srec
```

Because J-Flash Lite was running on Windows, the file was downloaded to Windows.

In QuickConnect Studio:

1. Right-click `first.srec`.
2. Select **Download**.
3. Save it to a Windows folder.

The Windows folder used was:

```text
C:\Users\manji\Downloads\renesas\
```

The complete file was:

```text
C:\Users\manji\Downloads\renesas\first.srec
```

---

## 8. USB/J-Link Connection

Connect the RA6E2 board to the PC using the board's dedicated debug USB connection.

The board was powered and Windows detected a serial interface:

```text
USB Serial Device (COM20)
```

The on-board debugger appeared in SEGGER J-Link Configurator as:

```text
J-Link-OB-RA4M2 V1.00
```

This is the on-board J-Link debug hardware used with the board.

---

## 9. Initial J-Link Error

The first J-Link connection attempt produced:

```text
Communication timeout. Emulator did not re-enumerate.
Cannot connect to the probe/programmer.
Failed to connect.
Could not establish a connection to the J-Link.
Connect failed.
```

J-Link Commander also showed:

```text
Updating firmware: J-Link OB-RA4M2 ...
Replacing firmware: J-Link OB-RA4M2 ...
FAILED: Communication timeout.
Emulator did not re-enumerate.
```

### What fixed the situation

J-Link Configurator was opened and showed the on-board J-Link:

```text
Product: J-Link-OB-RA4M2 V1.00
Host Firmware: 2026 May 6
Probe/Programmer Firmware: 2024 Oct 9 11:01 (Old)
```

Do **not** manually force the firmware update during this programming procedure.

When J-Flash Lite asks whether to update the connected emulator firmware, select:

```text
NO
```

---

## 10. Successful J-Link Connection

After the connection issue was resolved, J-Link successfully detected the target:

```text
Core: Cortex-M33
Core ID: 0x6BA02477
Target interface speed: 4000 kHz
Connected successfully
```

This confirmed that the PC, on-board J-Link, and RA6E2 were communicating.

---

## 11. Full J-Flash vs J-Flash Lite

The full:

```text
SEGGER J-Flash
```

was initially used.

Programming failed because it reported:

```text
No valid license for J-Flash found.
```

Therefore, use:

```text
SEGGER J-Flash Lite
```

for this procedure.

---

## 12. Configure J-Flash Lite

Open **SEGGER J-Flash Lite**.

Set:

```text
Target device:       R7FA6E2BB
Target interface:    SWD
Speed:               4000 kHz
```

### Flash bank

Enable:

```text
0x00000000  Internal program flash
```

Leave these disabled unless your specific application requires them:

```text
0x0100A100  Internal option-setting memory
0x08000000  Internal data flash
0x60000000  External QSPI flash
```

If J-Flash Lite reports:

```text
Error: No flash bank selected.
```

enable the **Internal program flash** checkbox and continue.

---

## 13. Load `first.srec`

In J-Flash Lite:

1. Click the `...` button beside **Data File**.
2. Open:

```text
C:\Users\manji\Downloads\renesas\first.srec
```

3. Select the file.

J-Flash Lite showed:

```text
Selected file:
C:\Users\manji\Downloads\renesas\first.srec
```

It detected 5 memory ranges.

The main program range was:

```text
0x00000000 - 0x0000A33B
```

---

## 14. Program the RA6E2

Click:

```text
Program Device
```

If this popup appears:

```text
A new firmware version is available for the connected emulator.
Do you want to update to the latest firmware version?
```

select:

```text
NO
```

Do not select **Yes** for this setup.

The successful programming log showed:

```text
Connecting to J-Link...
Bank selection: BankAddr=0x0100A100 Disabled
Bank selection: BankAddr=0x00000000 Enabled
Bank selection: BankAddr=0x08000000 Disabled
Bank selection: BankAddr=0x60000000 Disabled
Connecting to target...
Downloading...
Done.
```

This confirmed that the application was downloaded to the RA6E2 internal program flash.

---

## 15. Reset / Run the Application

After programming, reset the board.

The board has buttons marked:

```text
S1
S2
```

During testing, the upper button marked **S2** was used for the reset/test step.

The project README says that after reset the MCU should start BLE advertising and the green LED should blink.

---

## 16. Serial Check

Windows showed:

```text
USB Serial Device (COM20)
```

PuTTY was tested using:

```text
Connection type: Serial
Serial line: COM20
Baud rate: 115200
```

No useful serial text appeared in PuTTY during the test.

This did not stop the BLE application from working.

---

## 17. Quick-Connect Mobile Sandbox

Open the **Quick-Connect Mobile Sandbox** application on the phone.

Use its BLE scan function.

The BLE device from the RA6E2/DA14531 setup appeared and could be connected.

---

## 18. Final Successful Test

The BLE application worked successfully.

From the phone:
- The LED could be controlled **ON/OFF**.
- Temperature values could be viewed.

Therefore the communication path was working:

```text
Phone
  |
 BLE
  |
DA14531MOD
  |
 UART
  |
RA6E2
  |
 Sensor
```

LED control demonstrated the reverse direction:

```text
Phone
  |
 BLE command
  |
DA14531MOD
  |
RA6E2
  |
LED
```

Sensor data flowed:

```text
Sensor
  |
RA6E2
  |
DA14531MOD
  |
 BLE
  |
Phone
```

---

## 19. Troubleshooting

### A. J-Link cannot connect

Error:

```text
Communication timeout.
Emulator did not re-enumerate.
```

Check:
1. USB cable.
2. Correct debug USB connection.
3. Board power.
4. Windows Device Manager.
5. J-Link Configurator.
6. J-Link Commander.

Do not repeatedly force a J-Link firmware update.

### B. Full J-Flash asks for a license

Error:

```text
No valid license for J-Flash found.
```

Use:

```text
J-Flash Lite
```

### C. J-Flash Lite says no flash bank is selected

Error:

```text
No flash bank selected.
```

Enable:

```text
Internal program flash
0x00000000
```

### D. J-Flash Lite asks to update J-Link firmware

Select:

```text
No
```

### E. BLE device does not appear

Check:
1. DA14531MOD is connected.
2. US082-14 is connected in the correct PMOD position.
3. Physical hardware matches the `.qcc` diagram.
4. Correct `first.srec` was programmed.
5. Reset the board.
6. Wait several seconds for BLE advertising.
7. Scan again in Quick-Connect Mobile Sandbox.

### F. PuTTY is blank

Check:
- COM number
- Baud rate
- Correct serial interface
- Board reset

A blank PuTTY window did not prevent the BLE demo from working in this test.

---

## 20. DA14531MOD vs DA16600MOD

The kit contains multiple connectivity modules.

### DA14531MOD

Used for this project:

```text
Sensor Data over BLE
```

### DA16600MOD

A separate Wi-Fi/BLE connectivity module.

It was **not used in this project**.

For a DA16600MOD experiment, create/configure a separate QuickConnect Studio project rather than replacing the DA14531MOD in this working BLE project.

---

## 21. Complete Quick Procedure

For future use:

```text
1. Connect BGK-RA6E2.
2. Connect DA14531MOD to the left/PMOD1 position.
3. Connect US082-14 to the right/PMOD2 position.
4. Open QuickConnect Studio.
5. Open/create the project.
6. Verify the .qcc diagram.
7. Build the project.
8. Confirm 0 errors / 0 warnings.
9. Locate Debug/first.srec.
10. Right-click first.srec and Download it.
11. Save it to Windows.
12. Open J-Link Configurator if J-Link needs checking.
13. Confirm J-Link-OB-RA4M2 is detected.
14. Open J-Flash Lite.
15. Select R7FA6E2BB.
16. Select SWD.
17. Set 4000 kHz.
18. Enable Internal program flash (0x00000000).
19. Select first.srec.
20. Click Program Device.
21. If firmware update appears, select No.
22. Wait for Downloading... Done.
23. Reset the board.
24. Open Quick-Connect Mobile Sandbox.
25. Scan for BLE.
26. Connect to the RA MCU BLE device.
27. Test LED ON/OFF.
28. Check temperature values.
```

---

## 22. Working Configuration Summary

| Item | Working value |
|---|---|
| Board | BGK-RA6E2 |
| MCU | R7FA6E2BB |
| CPU | Cortex-M33 |
| Project | Sensor Data over BLE |
| BLE module | DA14531MOD |
| PMOD/interposer | US082-14 |
| Interface | SWD |
| Speed | 4000 kHz |
| Flash | Internal program flash |
| SREC | `first.srec` |
| Windows SREC path | `C:\Users\manji\Downloads\renesas\first.srec` |
| Serial device | COM20 |
| Mobile app | Quick-Connect Mobile Sandbox |
| Result | BLE + LED control + temperature working |

---

## 23. Important Result

The complete BLE demo was successfully verified:

```text
BGK-RA6E2
    +
DA14531MOD
    +
US082-14
    +
Sensor
    +
Quick-Connect Mobile Sandbox
```

The phone successfully controlled the LED and displayed temperature values.

This confirms that the programmed RA6E2 application, BLE module, sensor path, and mobile BLE communication are working.

---

## 24. Next Step

The next separate experiment can be based on:

```text
DA16600MOD
```

That should be treated as a separate QuickConnect Studio project for Wi-Fi/BLE functionality, while keeping this working `Sensor Data over BLE` project unchanged.
