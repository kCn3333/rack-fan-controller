# 𖣘 ESP32 PWM Fan Controller with Temperature Monitoring

<div align="center">

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![ESPHome](https://img.shields.io/badge/ESPHome-2025.12.3-blue.svg)
![Platform](https://img.shields.io/badge/platform-ESP32-blue.svg)

</div>
ESPHome-based fan controller designed for my homelab rack enclosure. This project automates thermal management using ESP32, DS18B20 temperature sensors, and PWM-controlled fans, fully integrated with Home Assistant.

## Features

- Automatic fan control based on **maximum temperature**
- Configurable temperature thresholds and fan speeds
- Manual fan control via Home Assistant
- Web UI (ESPHome Web Server)
- OTA updates over WiFi
- Stop & fan test mode


## Hardware used:

| Component | Description |
|-----------|-------------|
| ESP32 DevKit | ESP32-WROOM-32D/32U, 30-pin, Type-C | 
| DS18B20 Sensor | 2x with terminal adapter (4.7kΩ resistor) | 
| PWM Fans | 2x Arctic P12 PWM 4-pin  | 
| 12V Power Supply | 1.5A |
| USB Power Supply | 5V 1A with USB Type-C cable | 
| Dupont Cables | |


## Wiring Diagram

![diagram](wiring-diagram.png)

**⚠️ CRITICAL:** Connect 12V PSU GND to ESP32 GND (common ground required for PWM signal!)

## Software Setup

### Prerequisites

- [ESPHome](https://esphome.io/) installed
- [Home Assistant](https://www.home-assistant.io/) (recommended but optional)
- USB cable for initial firmware upload

### Installation Steps

#### 1. Clone this repository

```bash
git clone https://github.com/YOUR_USERNAME/esp32-fan-controller.git
cd esp32-fan-controller
```

#### 2. Configure WiFi credentials

Edit `fan-controller.yaml` and update WiFi settings:

```yaml
wifi:
  ssid: "YOUR_WIFI_SSID"
  password: "YOUR_WIFI_PASSWORD"
```

Also update web server credentials:

```yaml
web_server:
  auth:
    username: admin
    password: YOUR_PASSWORD
```

#### 3. Install firmware

**Option A: ESPHome Dashboard (recommended)**

1. Add device in ESPHome Dashboard
2. Copy contents of `fan-controller.yaml`
3. Click Install → Plug into this computer
4. Select COM port
5. Wait for installation

**Option B: ESPHome CLI**

```bash
esphome run fan-controller.yaml
```

**Option C: Web Flasher (first time only)**

1. Visit https://web.esphome.io/
2. Connect ESP32 via USB
3. Upload compiled `.bin` file
4. After first flash, use OTA for updates

#### 4. Add to Home Assistant

After successful installation:

1. Go to Settings → Devices & Services
2. ESPHome integration should auto-discover device
3. Click Configure
4. Enter encryption key (found in compilation logs)
5. Done!

## Configuration

### Temperature Thresholds & Fan Speeds

Configure via Home Assistant or web interface:

| Setting | Default | Description |
|---------|---------|-------------|
| Temp Threshold 1 | 30°C | Minimum temperature to start fans |
| Speed 1 (Low) | 30% | Fan speed when temp > Threshold 1 |
| Temp Threshold 2 | 40°C | Medium temperature |
| Speed 2 (Medium) | 60% | Fan speed when temp > Threshold 2 |
| Temp Threshold 3 | 50°C | High temperature |
| Speed 3 (High) | 100% | Fan speed when temp > Threshold 3 |

### Control Logic

The system uses **two temperature sensors** (TOP and BOTTOM) and controls both fans based on the **maximum temperature** reading:

```
temp_max = max(temp_top, temp_bottom)

Temperature < 30°C → Fans OFF (0%)
Temperature 30-40°C → Fans at 30%
Temperature 40-50°C → Fans at 60%
Temperature > 50°C → Fans at 100%
```

**Why max(temp_top, temp_bottom)?**
- If either sensor detects high temperature, both fans respond
- Prevents hot spots in the rack
- More reliable than single sensor monitoring

All values are configurable via sliders in Home Assistant.

## Usage

### Auto Mode

1. Enable "Auto Control" switch
2. System automatically adjusts fan speed based on temperature
3. Configure thresholds and speeds via sliders

### Manual Mode

1. Disable "Auto Control" switch
2. Use individual fan controls or master ON/OFF switch
3. Set desired fan speed (0-100%)

### Available Controls

**Switches:**
- `Auto Control` - Enable/disable automatic temperature-based control
- `Fans ON/OFF` - Master switch for all fans (50% default)

**Buttons:**
- `Test Fans` - Run both fans at 100% for 5 seconds
- `STOP Fans` - Emergency stop (disables auto mode)
- `Restart Controller` - Restart ESP32

**Sensors:**
- `Temperature TOP` - Top sensor temperature reading
- `Temperature BOTTOM` - Bottom sensor temperature reading
- `Temperature MAX` - Maximum of both sensors (used for control)
- `Speed TOP` - Current speed of top fan (%)
- `Speed BOTTOM` - Current speed of bottom fan (%)
- `WiFi Signal` - WiFi signal strength
- `Uptime` - Device uptime

## Home Assistant Dashboard Example

Example dashboard configuration in `dashboard-example.yaml`:

```yaml
type: vertical-stack
cards:
  - type: history-graph
    title: Temperature History
    hours_to_show: 24
    entities:
      - entity: sensor.temperatura_rack
  
  - type: entities
    title: Fan Controller
    entities:
      - sensor.temperatura_rack
      - switch.auto_sterowanie
      - switch.wentylatory_on_off
      - fan.wentylator_top
      - fan.wentylator_down
```

See full example in `dashboard-example.yaml`.

## OTA Updates

### Via ESPHome Dashboard (WiFi)

```bash
esphome run fan-controller.yaml --device fan-controller.local
```

### Via Web Interface

1. Open `http://fan-controller.local`
2. Login with credentials
3. Look for update/upload button
4. Select new `.bin` file

### Via Home Assistant

1. Settings → Devices & Services → ESPHome
2. Find `fan-controller`
3. Click Update (if available)

## Troubleshooting

### Fans not spinning

**Check:**
- 12V power supply connected and ON
- PWM wires connected to correct GPIO pins (25, 26)
- Common ground between 12V PSU and ESP32
- Fans support PWM control (4-pin connectors)
- Fan speed > 20% (minimum startup speed)

### Temperature sensor not detected

**Check:**
- Both DS18B20 sensors connected to GPIO4 (parallel)
- 4.7kΩ pull-up resistor present (usually on terminal board)
- Correct wiring: Red→3.3V, Yellow→GPIO4, Black→GND
- Check ESPHome logs for "Found DS18B20" message (should see 2 devices)
- Sensors have unique addresses

**Finding sensor addresses:**
Check logs after first boot:
```
[one_wire] Found DS18B20 device 0x1c0000031edd2a28
[one_wire] Found DS18B20 device 0x233c01f096fff428
```

### WiFi connection issues

**Solutions:**
- Check SSID and password in configuration
- Ensure 2.4GHz WiFi (ESP32 doesn't support 5GHz)
- Device creates fallback hotspot "Fan-Controller-Fallback" (password: fancontrol123)
- Connect to fallback and reconfigure WiFi

### Auto mode not working

**Check:**
- "Auto Control" switch is enabled
- Temperature thresholds configured correctly
- Check ESPHome logs for auto control messages
- Ensure temperature sensor is working

## Advanced Configuration

### Identifying Sensor Addresses

After first boot with both sensors connected, check logs:

```
[one_wire] Found DS18B20 device 0x1c0000031edd2a28
[one_wire] Found DS18B20 device 0x233c01f096fff428
```

The sensors are automatically assigned:
- `index: 0` → First sensor found (TOP)
- `index: 1` → Second sensor found (BOTTOM)

If you need to swap them, physically swap the sensor probes or use explicit addresses in YAML:

```yaml
sensor:
  - platform: dallas_temp
    address: 0x1c0000031edd2a28  # Specific address for TOP
    name: "Temperature TOP"
```

### Using Single Sensor

If you only have one sensor, the system still works:
- Comment out the second sensor in YAML
- `temp_max` will use the single sensor value
- Both fans controlled by one temperature reading

### Adjusting PWM Frequency

Change in YAML (default 25kHz):

```yaml
output:
  - platform: ledc
    frequency: 25000 Hz  # Adjust if needed
```

### Custom Fan Curves

Modify lambda function in `on_value` section for custom temperature/speed curves.



