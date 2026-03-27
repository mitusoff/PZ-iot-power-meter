# PZ-iot-power-meter

⚡ **Smart Electricity Meter Based on PZEM-004T**

A ready-made solution for electricity monitoring based on ESPHome, the BK7231 microcontroller (CB3S module), and the PZEM‑004T sensor. The project turns an ordinary meter into a full-fledged smart device with a web interface, detailed statistics, and Home Assistant integration.

Ideal for tracking power consumption, calculating electricity costs, and monitoring grid quality in your smart home.

---

## ✨ Key Features

- **Real-time monitoring**: measures voltage (V), current (A), power (W), frequency (Hz), and power factor (cos φ).
- **Energy accounting**: automatically tracks electricity usage with daily, weekly, and monthly counter resets.
- **Cost calculation**: calculates electricity costs based on a configurable tariff (RUB/kWh).
- **Synchronization with an external meter**: allows you to enter readings from your regular electricity meter and synchronize PZEM data with them.
- **Web server**: includes a built-in web interface with convenient parameter grouping, which works even without Home Assistant.
- **Power quality analysis**: automatically calculates apparent power (VA) and reactive power (VAR).
- **Smart alerts**: binary sensors report load presence, overload, and low power factor.
- **Reliable connectivity**: supports two Wi‑Fi networks and access point mode for easy connection recovery.

---

## 🛠️ Required Components

To assemble the device, you will need:

- **Microcontroller**: a module based on the BK7231 chip (for example, CB3S, WB3S, WB2S). The configuration is optimized for the CB3S board.
- **Energy sensor**: PZEM‑004T V3.0 module. A popular electricity meter with a Modbus (TTL) interface.
- **Power supply**: a stable 3.3 V power source for the BK7231 module (or 5 V depending on your board, since the configuration is scalable and can easily be adapted for ESP8266 or ESP32 depending on your needs).

### PZ IoT Power Meter — Wiring Diagram
![Wiring Diagram](img/connection-scheme.png)

## Connection

### Power Supply
1. **220V AC** → to the input of the AC-DC converter (5V power supply in my example) (220V → 5V) and to the power input of the PZEM-004T
2. **5V** → to the input of the DC-DC step-down module (5V → 3.3V) (or a linear AMS1117-3.3 regulator with the required supporting components)
3. **3.3V** → to the VCC of the microcontroller (CB3S (for ESP8266 similarly)) and to the VCC of the UART section of the PZEM-004T
4. **GND** → to the GND of the microcontroller and the GND of the UART section of the PZEM-004T

### Communication (UART)
| Microcontroller | PZ IoT Power Meter |
|-----------------|-------------------|
| TX | RX+ |
| RX | RX- |
| VCC | VCC |
| GND | GND |

## Components
- AC-DC 220V → 5V (with galvanic isolation)
- 5V → 3.3V regulator (DC-DC converter from 5V to 3.3V or AMS1117-3.3 with supporting components)
- CB3S microcontroller (or ESP8266 with the required startup circuitry and config adjustments)
- PZEM-004T v3 or v4

## ⚠️ Safety Measures
- Insulate all 220V connections
- It is extremely important to use only an isolated AC-DC converter with galvanic isolation
- Check the logic levels of the meter (5V or 3.3V)
- Do not connect 220V to the UART section of the PZEM-004T

---

## 📂 Configuration Features
As a basis for building the configuration, the standard config for declaring sensors was used:

```yaml
modbus:

sensor:
  - platform: pzemac
    current:
      name: "PZEM-004T V3 Current"
    voltage:
      name: "PZEM-004T V3 Voltage"
    energy:
      name: "PZEM-004T V3 Energy"
    power:
      name: "PZEM-004T V3 Power"
    frequency:
      name: "PZEM-004T V3 Frequency"
    power_factor:
      name: "PZEM-004T V3 Power Factor"
    update_interval: 60s
```
Original source link: [Click.](https://esphome.io/components/sensor/pzemac/).

The configuration is built on ESPHome capabilities and includes a number of well-thought-out solutions:

- **Modbus communication:** uses the `modbus` and `uart` components to exchange data with the PZEM‑004T at 9600 baud.
- **Automatic counter reset:** a special `interval` checks the date every minute and automatically resets daily, weekly, and monthly values at the right time.
- **Data persistence:** accumulated energy data and meter reading offsets are stored in `globals` with restoration after reboot.
- **Convenient web interface:** all sensors and controls are divided into logical groups in the web server: main parameters, energy and cost, power quality, diagnostics, etc.
- **Spike protection:** filters have been added for all sensors to discard incorrect readings (for example, voltage 0 V or frequency 100 Hz).

## 🚀Getting Started

1. **Install ESPHome:** if you haven’t done it yet, install ESPHome (dashboard or command line).

2. **Copy the configuration:** take the ready-made YAML file from this repository.

3. **Set up secrets:** replace all `!secret` values with real data. Create a `secrets.yaml` file next to the configuration:

```yaml

wifi_ssid: "Your Wi-Fi network name"
wifi_password: "Password"
wifi_ssid2: "Backup network"
wifi_password2: "Backup network password"
api_encryption_key: "Generated encryption key"
```

4. Adjust parameters (optional):** if necessary, change `device_timezone` (time zone) and `electricity_tariff` (default tariff is 7.1 RUB/kWh).

5. **Compile and upload:** compile the firmware and upload it to the BK7231 board (via UART or over the air after the first flash).

6. **Open the web interface:** after connecting to Wi‑Fi, open the device IP address in your browser.

## 🔗Home Assistant Integration
The device integrates fully and seamlessly with Home Assistant thanks to the native ESPHome API.

## ⁉️Possible Difficulties You May Encounter

-  ❗Since in my configuration the device is based on CB3S (BK7231), the question will arise: how to flash it, because you will not be able to do this using ESPHome Flasher! In this case, you need to install BK7231Flasher or ltchiptool, and flash it via UART using one of them. After flashing, further updates will be possible through the web interface via OTA. To avoid compiling the firmware yourself, I have included ready-made flashing files in the Firmware folder.

-  ❗Another difficulty may arise during firmware compilation due to the hardware limitations of the device you are compiling on (for example, on my Raspberry Pi 4 with 4GB RAM, compilation ended with an error). The solution is as follows: you need a PC (or laptop, mini PC) with a sufficient amount of RAM (in my case Ryzen R5 5600X with 16GB, i5 12400 with 64GB — on these configurations everything compiles without problems). Install Docker Desktop on the device, then deploy an ESPHome container in Docker, and from there edit the config and compile the firmware file.
