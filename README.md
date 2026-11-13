# Huawei ESM-48100 Battery Monitor

ESPHome firmware for bringing a Huawei ESM-48100 battery system into Home Assistant over **RS485 Modbus RTU**. An ESP32 polls the BMS and publishes pack, cell, temperature, and status data as native Home Assistant entities.

> **Project status:** Community integration based on the register map in the configuration. Validate register values and transceiver wiring on your own hardware before relying on the data.

## At A Glance

| | |
| --- | --- |
| Controller | ESP32 DevKit (`esp32dev`) |
| Battery | Huawei ESM-48100U2 / U5 register map |
| Bus | RS485, Modbus RTU, 9600 baud, 8-N-1 |
| Integration | ESPHome Native API and Home Assistant |
| Poll interval | 5 seconds for Modbus entities |
| Configuration | [`huawei_esm48100.yaml`](huawei_esm48100.yaml) |

## What It Monitors

- **Pack:** battery and pack voltage, signed current, state of charge, state of health, and calculated power.
- **Cell diagnostics:** 15 individual cell voltages and the calculated max-to-min voltage delta.
- **Temperature:** highest and lowest cell temperatures, plus 15 diagnostic temperature channels.
- **Status and history:** charging/discharging state, fault indication, discharge amp-hours, discharge count, and capacity-related values.
- **Device diagnostics:** raw status registers, charge/discharge limit coefficients, and ESP IP address.

Calculated entities include battery power, cell voltage delta, and estimated remaining capacity. Remaining capacity uses the configured nominal capacity, SOC, and SOH values.

## Hardware

You will need an ESP32 development board, a **3.3 V logic-compatible RS485 transceiver with automatic transmit/receive direction control**, and a suitable cable to the battery's RJ45 COM port. This converter does not require a separate flow-control GPIO. Check the transceiver's supply and logic-level requirements: ESP32 GPIOs are not 5 V tolerant.

### ESP32 to RS485 transceiver

| ESP32 | Transceiver |
| --- | --- |
| GPIO16 (TX) | DI |
| GPIO17 (RX) | RO |
| 3.3 V / GND | VCC / GND, as supported by your module |

### Transceiver to ESM-48100

The configuration header documents the battery's RJ45 COM pins as:

| RJ45 pin | Signal |
| --- | --- |
| 1 | RS485 T+ |
| 2 | RS485 T- |
| 4 | RS485 R+ |
| 5 | RS485 R- |

Pins 3 and 6 are reserved; pins 7 and 8 are CAN and are not used by this project. Confirm the pinout for your battery revision and use the correct differential pair for your interface. Fit a 120-ohm termination resistor at the far end of the RS485 bus where required by the installation.

## Configure And Flash

1. Install [ESPHome](https://esphome.io/) and add this repository's YAML to your ESPHome configuration directory.
2. Create a `secrets.yaml` next to the device configuration with the required values:

   ```yaml
   api_encryption_key: "<your ESPHome API encryption key>"
   ota_password: "<a strong OTA password>"
   wifi_ssid: "<your Wi-Fi name>"
   wifi_password: "<your Wi-Fi password>"
   ap_password: "<a strong fallback access point password>"
   ```

3. Check the UART pins and Modbus address against your hardware. With an automatic-direction converter, no ESPHome flow-control pin is needed.
4. Compile and install:

   ```sh
   esphome run huawei_esm48100.yaml
   ```

5. Once online, approve the ESPHome device in Home Assistant if prompted. The configuration exposes the entities through the ESPHome Native API.

The configured Modbus slave address is **214**. If your BMS uses a different address, update `modbus_controller.address` in the YAML.

## Register Notes

The YAML identifies the map as verified for ESM-48100U2/U5, but some values are explicitly uncertain: the cycle-count register at address `32108` is marked unconfirmed, and the capacity register comment lists `32152`, `32153`, or `32155` as possibilities. Treat these entities as experimental until checked against your battery firmware and observed readings.

## Safety And Scope

This project polls and reports BMS data; it is not a battery protection system and should not be used as a substitute for the BMS, electrical protections, or manufacturer guidance. The configuration uses holding-register reads and does not define battery-control writes. Verify bus polarity, isolation, voltage levels, and register interpretation before installation.

## Repository

```text
.
|-- README.md
`-- huawei_esm48100.yaml
```