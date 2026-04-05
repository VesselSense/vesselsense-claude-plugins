# Gauge Mapping Rules

## Type Field

| type | Visual Style |
|------|-------------|
| 0 | Analog dial (compass, arc gauge) |
| 1 | Digital LCD (segment display) |
| 4 | Bar or Circle gauge |
| (omit) | className default rendering |

Values 2 and 3 have not been observed.

## Specific Path Mappings (highest priority)

Apply in order — first match wins.

| Path | className | type | Title Hint |
|---|---|---|---|
| `navigation.attitude` | `RollGauge` | 0 | Roll |
| `navigation.attitude` | `PitchGauge` | 0 | Pitch |
| `navigation.attitude` | `YawGauge` | 0 | Yaw |
| `navigation.headingMagnetic` | `MagneticHeadingGauge` | 0 | Heading Mag |
| `navigation.headingTrue` | `HeadingGauge` | 0 | Heading True |
| `navigation.courseOverGroundTrue` | `COGGauge` | 0 | COG |
| `navigation.speedOverGround` | `SOGGauge` | 0 | SOG |
| `navigation.speedThroughWater` | `SpeedGauge` | 1 | STW |
| `navigation.position` | `PositionGauge` | (omit) | Position |
| `navigation.log` | `TextGaugeConfig` | 1 | Log |
| `navigation.trip.log` | `TTWGauge` | (omit) | Trip |
| `navigation.rateOfTurn` | `TextGaugeConfig` | 1 | Rate of Turn |
| `navigation.courseGreatCircle.nextPoint.distance` | `DTWGauge` | 1 | DTW |
| `navigation.courseGreatCircle.*.estimatedTimeOfArrival` | `ETATimeGauge` | 1 | ETA |
| `navigation.course.calcValues.bearingMagnetic` | `MagneticHeadingGauge` | 0 | Bearing Mag |
| `environment.wind.angleApparent` | `AWAGauge` | (omit) | AWA |
| `environment.wind.angleTrueWater` | `TWAGauge` | (omit) | TWA |
| `environment.wind.speedApparent` | `WindSpeedGauge` | 0 | AWS |
| `steering.autopilot.*` | `AutoPilotControl` | (omit) | Autopilot |

## Pattern-Based Mappings

| Path Pattern | className | type | Title Hint |
|---|---|---|---|
| `environment.depth.*` | `DepthGauge` | 1 | Depth |
| `environment.*.temperature` | `WaterTempGauge` | 4 | {zone} Temp |
| `environment.*.humidity` | `RatioGauge` | 4 | {zone} Humidity |
| `environment.*.pressure` | `PressureGauge` | (omit) | {zone} Pressure |
| `environment.*.flood` | `RatioGauge` | 1 | {zone} Flood |
| `propulsion.*.revolutions` | `RPMGauge` | 0 | RPM |
| `propulsion.*.oilPressure` | `OilPressureGauge` | 0 | Oil Pressure |
| `propulsion.*.coolantTemperature` | `CoolantTemperatureGauge` | 0 | Coolant Temp |
| `propulsion.*.exhaust.temperature` | `CoolantTemperatureGauge` | 0 | Exhaust Temp |
| `propulsion.*.alternator.temperature` | `CoolantTemperatureGauge` | 0 | Alternator Temp |
| `electrical.batteries.*.voltage` | `BatteryTwoGauge` | 0 | {name} Voltage |
| `electrical.batteries.*.current` | `AmpsGauge` | 4 | {name} Current |
| `electrical.batteries.*.power` | `WattsGauge` | 1 | {name} Power |
| `electrical.batteries.*.capacity.stateOfCharge` | `RatioGauge` | 4 | {name} SOC |
| `electrical.batteries.*.capacity.timeRemaining` | `TextGaugeConfig` | 1 | Time Remaining |
| `electrical.solar.*.current` | `AmpsGauge` | 4 | {name} Solar A |
| `electrical.solar.*.voltage` | `TextGaugeConfig` | 1 | {name} Solar V |
| `electrical.solar.*.panelPower` | `WattsGauge` | 1 | {name} Solar W |
| `electrical.solar.*.yieldToday` | `TextGaugeConfig` | 1 | {name} Yield |
| `electrical.grid.*.voltage` | `TextGaugeConfig` | 1 | Grid Voltage |
| `electrical.grid.*.current` | `AmpsGauge` | 1 | Grid Current |
| `electrical.grid.*.power` | `WattsGauge` | 1 | Grid Power |
| `electrical.switches.*.state` | `SwitchGauge` | (omit) | {name} Switch |
| `electrical.switches.*.on` | `LEDGauge` | (omit) | {name} Status |
| `electrical.switches.*.temperature` | `WaterTempGauge` | 4 | {name} Temp |
| `electrical.switches.*.current` | `AmpsGauge` | 1 | {name} Current |
| `electrical.switches.*.voltage` | `TextGaugeConfig` | 1 | {name} Voltage |
| `electrical.pumps.*.cycles*` | `TextGaugeConfig` | 1 | {metric} |
| `electrical.pumps.*.lastCycle*` | `TextGaugeConfig` | 1 | Last Cycle |
| `electrical.pumps.*.totalRunTime` | `TextGaugeConfig` | 1 | Total Runtime |
| `electrical.venus.totalPanelCurrent` | `AmpsGauge` | 4 | Solar Total A |
| `electrical.venus.totalPanelPower` | `WattsGauge` | 1 | Solar Total W |
| `electrical.venus.dcPower` | `WattsGauge` | 1 | DC Power |
| `electrical.venus.state` | `TextGaugeConfig` | 1 | Charge State |
| `tanks.*.currentLevel` | `RatioGauge` | 1 | {tank} Level |
| `computer.*.temperature` | `WaterTempGauge` | 0 | CPU Temp |
| `computer.*.utilisation` | `RatioGauge` | 0 | {component} |

## Fallback Rules (lowest priority)

| Path ending | className | type |
|---|---|---|
| `*.voltage` | `TextGaugeConfig` | 1 |
| `*.current` | `AmpsGauge` | 4 |
| `*.power` | `WattsGauge` | 1 |
| `*.temperature` | `WaterTempGauge` | 4 |
| `*.humidity` | `RatioGauge` | 4 |
| `*.pressure` | `PressureGauge` | (omit) |
| `*.state` (boolean) | `SwitchGauge` | (omit) |
| Value between 0-1 | `RatioGauge` | 4 |
| Everything else | `TextGaugeConfig` | 1 |

## Multiple Battery Handling

- First battery → `BatteryTwoGauge` (type 0) — typically house/main
- Second battery → `BatteryOneGauge` (type 0) — typically starter
- Third+ → `TextGaugeConfig` (type 1)

## Title Generation

- Extract meaningful segment: `environment.inside.temperature` → "Inside Temp"
- Use `name` property if available for batteries/solar, otherwise the ID
- Max ~20 characters
- Capitalize first letter of each word

## Known Gauge Classes

**Analog Dials:** `AWAGauge`, `AWACOGGauge`, `TWAGauge`, `GWAGauge`, `WindSpeedGauge`, `HeadingGauge`, `MagneticHeadingGauge`, `HeadWindCOGGauge`, `COGGauge`, `RollGauge`, `PitchGauge`, `YawGauge`, `RPMGauge`, `OilPressureGauge`, `CoolantTemperatureGauge`, `BatteryOneGauge`, `BatteryTwoGauge`, `Battery24VGauge`, `SpeedGauge`, `SOGGauge`, `DepthGauge`

**Digital Displays:** `TextGaugeConfig`, `AmpsGauge`, `WattsGauge`, `WaterTempGauge`, `RatioGauge`, `PercentGauge`, `PressureGauge`

**Distance/Time:** `DTWGauge`, `TTWGauge`, `ETATimeGauge`, `XTEGauge`

**Control/Status:** `SwitchGauge`, `LEDGauge`, `AutoPilotControl`, `FuelGauge`

**Navigation:** `PositionGauge`, `Map`

**Special:** `EmptyGauge` (spacer), `SetGauge` (current set), `DriftGauge` (current drift)

This list is derived from observed usage. WilhelmSK may support additional classes. `TextGaugeConfig` with `type: 1` is a safe fallback for unknown paths.
