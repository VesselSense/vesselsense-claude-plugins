---
name: signalk-sensesp
description: SensESP framework, ESP32 custom sensors, PlatformIO, Arduino, WiFi, SignalK HTTP client, transforms, sensor paths.
---

# SensESP - ESP32 Sensor Development Framework

## Overview

SensESP is a framework for building custom marine sensors on ESP32/ESP8266 microcontrollers. Sensors connect to Signal K Server via WiFi WebSocket, appearing as additional data sources.

**Repository:** https://github.com/SignalK/SensESP
**Platform:** ESP32 (recommended), ESP8266 (legacy)
**Build System:** PlatformIO (Arduino framework)
**Current Version:** 3.x (pioarduino platform)

## Architecture

```
┌──────────────────────────┐     WiFi WebSocket     ┌──────────────────┐
│      ESP32 Device        │ ─────��────────────────▶ │  SignalK Server  │
│                          │    Delta messages       │                  │
│  Sensor → Transform      │                        │  Merges into     │
│         → SKOutput       │                        │  data model      │
└──────────────────────────┘                        └─────────��────────┘
```

SensESP devices:
1. Read physical sensors (I2C, analog, digital, 1-Wire)
2. Apply transforms (calibration, filtering, conversion)
3. Emit Signal K deltas to the server over WiFi WebSocket

## Common Sensor Types

### Temperature
| Sensor | Interface | Library | Use Case |
|--------|-----------|---------|----------|
| BME280 | I2C | Adafruit_BME280 | Ambient temp + humidity + pressure |
| DS18B20 | 1-Wire | sensesp/OneWire | Water/exhaust/engine/alternator temp |
| MAX31855 | SPI | Adafruit_MAX31855 | Exhaust gas (thermocouple) |
| SHT35 | I2C | — | High-accuracy temp + humidity |

### Air Quality
| Sensor | Interface | Measurements | Use Case |
|--------|-----------|--------------|----------|
| SEN55 | I2C (0x69) | PM1.0/2.5/4/10, VOC, NOx, temp, humidity | Engine room air quality |
| BME680 | I2C | VOC, temp, humidity, pressure | Cabin air quality |

### Engine Monitoring
| Measurement | Sensor/Method | Notes |
|-------------|---------------|-------|
| RPM | Digital pulse (W terminal) | Alternator or flywheel pickup, configurable pulses/rev |
| Oil pressure | MPX5700 (analog) | 0-5V sensor with voltage divider |
| Coolant temp | DS18B20 | 1-Wire on shared bus |
| Exhaust temp | DS18B20 or MAX31855 | 1-Wire or thermocouple |
| Alternator temp | DS18B20 | Mounted on alternator housing |
| Run hours | TimeCounter (software) | Accumulated when RPM > 0, persisted to SPIFFS |

### Tank Level
| Method | Sensor | Notes |
|--------|--------|-------|
| Resistive | 0-180 ohm sender | Standard marine fuel/water senders |
| Ultrasonic | DS1603L, JSN-SR04T | Non-contact level measurement |
| Capacitive | Custom | DIY tank sensing |

### Power Monitoring
| Sensor | Interface | Use Case |
|--------|-----------|----------|
| INA219 | I2C | Current/voltage/power per circuit |
| ADS1115 | I2C | 16-bit ADC for analog sensors |

## Project Structure

```
my-sensesp-project/
├── platformio.ini          # Build configuration
���── src/
│   ├── main.cpp            # Sensor setup and transform chains
│   ├── my_const.h          # Pin definitions, constants
│   ��── my_digital.h        # Digital sensor helpers
│   └── my_digital.cpp      # Digital sensor implementations
└── README.md
```

### platformio.ini (SensESP 3.x)

```ini
[env:esp32dev]
platform = https://github.com/pioarduino/platform-espressif32/releases/download/54.03.20/platform-espressif32.zip
board = esp32dev
framework = arduino
board_build.partitions = min_spiffs.csv
lib_deps =
    SignalK/SensESP @ ^3.0.0
    sensesp/OneWire @ ^3.0.1
    sensirion/Sensirion I2C SEN5X @ ^0.3.0
monitor_speed = 115200

[env:ota]
extends = env:esp32dev
upload_protocol = espota
upload_port = 192.168.x.x
upload_flags =
    --auth=my-ota-password
```

## SensESP 3.x Patterns

### RepeatSensor (Polling-Based Sensors)

For sensors that require explicit polling (not interrupt-driven):

```cpp
#include <sensesp/sensors/sensor.h>

// Global state updated by polling task
float pm1p0 = 0;

// Poll SEN55 every 100ms via event loop
event_loop()->onRepeat(100, []() {
  sen5x.readMeasuredValues(pm1p0, pm2p5, pm4p0, pm10p0,
                           humidity, temperature, vocIndex, noxIndex);
});

// RepeatSensor bridges polled data → SensESP observable (1000ms)
auto* pm1_sensor = new RepeatSensor<float>(1000, []() -> float {
  return pm1p0;
});

pm1_sensor->connect_to(
  new SKOutputFloat("environment.engineroom.massConcentrationPm1p0",
                    "/sensors/sen55/pm1p0",
                    new SKMetadata("ug/m3", "PM1.0 Mass Concentration"))
);
```

### TimeCounter (Run Hours with Persistence)

Accumulates duration while a condition is true, persists to SPIFFS:

```cpp
#include <sensesp/transforms/time_counter.h>

// engine_running is a bool observable (RPM > 0)
auto* engine_hours = new TimeCounter<bool>("/engine_hours/counter");

// Seed initial hours on first boot (e.g., 1100 hours = 3960000 seconds)
engine_hours->set_duration(3960000);

engine_running_observable
  ->connect_to(engine_hours)
  ->connect_to(new SKOutputFloat("propulsion.0.runTime",
                                  "/sensors/engine/hours",
                                  new SKMetadata("s", "Engine Hours")));
```

### LambdaTransform (Inline Conversion)

```cpp
#include <sensesp/transforms/lambda_transform.h>

// Convert Hz to boolean (engine running if RPM > 0)
auto* hz_to_running = new LambdaTransform<float, bool>(
  [](float hz) -> bool { return hz > 0; }
);

rpm_sensor->connect_to(hz_to_running);
```

### ConnectTachoSender (RPM Pattern)

```cpp
// Helper function for tachometer setup
void ConnectTachoSender(int pin, String name) {
  const float kTachoScale = 1.0 / 100.0;  // 100 pulses per revolution

  auto* tacho_input = new DigitalInputCounter(pin, INPUT_PULLUP, RISING, 500);

  tacho_input
    ->connect_to(new Frequency(kTachoScale, "/sensors/" + name + "/freq"))
    ->connect_to(new SKOutputFloat("propulsion.0.revolutions",
                                    "/sensors/" + name + "/sk",
                                    new SKMetadata("Hz", "Engine Revolutions")));
}
```

### Multiple DS18B20 on One Bus

```cpp
#include <sensesp/sensors/onewire_temperature.h>

DallasTemperatureSensors* dts = new DallasTemperatureSensors(ONEWIRE_PIN);

// Each sensor addressed by index on the bus
auto* alternator_temp = new OneWireTemperature(dts, 1000, "/sensors/alt_temp");
auto* exhaust_temp = new OneWireTemperature(dts, 1000, "/sensors/exh_temp");

alternator_temp->connect_to(
  new SKOutputFloat("propulsion.0.alternator.temperature",
                    "/sk/alt_temp",
                    new SKMetadata("K", "Alternator Temperature")));

exhaust_temp->connect_to(
  new SKOutputFloat("propulsion.0.exhaust.temperature",
                    "/sk/exh_temp",
                    new SKMetadata("K", "Exhaust Temperature")));
```

### SKOutput with Metadata

Always provide units and description for proper display in Signal K:

```cpp
new SKOutputFloat("propulsion.0.revolutions",
                  "/sensors/engine/rpm",
                  new SKMetadata("Hz", "Engine Revolutions"))
```

## NMEA 0183 Passthrough

ESP32 can bridge serial NMEA to TCP alongside SensESP:

```cpp
// Serial1 reads NMEA at 4800 baud
Serial1.begin(4800, SERIAL_8N1, RX_PIN, TX_PIN);

// TCP client forwards to SignalK server
WiFiClient tcpClient;
tcpClient.connect("signalk-server.local", 10110);

// In event loop: forward bytes
event_loop()->onRepeat(10, [&]() {
  while (Serial1.available()) {
    tcpClient.write(Serial1.read());
  }
});
```

## Transforms Reference

| Transform | Purpose |
|-----------|---------|
| `Linear` | Scale and offset (y = mx + b) |
| `CurveInterpolator` | Multi-point calibration curve |
| `Frequency` | Pulse count to Hz (with scale factor) |
| `MovingAverage` | Smooth noisy readings |
| `MedianFilter` | Remove spike outliers |
| `ChangeFilter` | Only emit on significant change |
| `TimeCounter<bool>` | Accumulate duration while true (SPIFFS persistence) |
| `LambdaTransform<In,Out>` | Inline custom conversion |
| `VoltageDivider` | Correct for voltage divider circuits |
| `AngleCorrection` | Apply offset to angle readings |

## Partition Tables

For OTA support, use `min_spiffs.csv`:

```
# Name,   Type, SubType, Offset,   Size
nvs,      data, nvs,     0x9000,   0x5000
otadata,  data, ota,     0xe000,   0x2000
app0,     app,  ota_0,   0x10000,  0x1E0000
app1,     app,  ota_1,   0x1F0000, 0x1E0000
spiffs,   data, spiffs,  0x3D0000, 0x20000
coredump, data, coredump,0x3F0000, 0x10000
```

SPIFFS stores persistent config (engine hours, calibration values).

## WiFi & Discovery

- SensESP creates WiFi AP on first boot for configuration
- Connects to Signal K server via mDNS discovery (`signalk-http._tcp`)
- Access request on first connection (approve in admin UI)
- OTA updates via `espota` protocol (password-protected)

## Logging

```cpp
#include <esp_log.h>
static const char* TAG = "main";

ESP_LOGI(TAG, "Engine RPM: %.1f Hz", rpm_value);
ESP_LOGW(TAG, "Sensor read failed, retrying");
```

## Hardware Boards

| Board | Flash | Features | Use Case |
|-------|-------|----------|----------|
| HALMET | 8MB | NMEA 2000 + 0183 + engine I/O | Engine monitoring |
| ESP32-DevKit | 4MB | General purpose, many GPIO | Prototyping |
| ESP32-C3 (Xiao) | 4MB | Compact, USB-C native | Space-constrained |
| ESP32-C6 | 4MB | WiFi 6, Thread/Zigbee | Future IoT |
