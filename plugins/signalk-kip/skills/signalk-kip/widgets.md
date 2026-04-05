# KIP Widget Catalog

## Widget Structure

Every widget in `dashboardsConfig[].configuration[]` follows this shape:

```json
{
  "w": 6, "h": 6, "x": 0, "y": 0,
  "id": "<uuid>",
  "selector": "widget-host2",
  "input": {
    "widgetProperties": {
      "type": "<widget-type>",
      "uuid": "<same-uuid-as-id>",
      "config": { /* IWidgetSvcConfig */ }
    }
  }
}
```

**Grid**: 24 columns wide, rows auto-expand. Minimum widget size is typically 4x4.

## Widget Types

### Numeric Display (`widget-numeric`)

Displays a single numeric value with optional minichart.

**Default size**: 6w x 6h
**Common paths**: Any numeric Signal K path

```json
{
  "type": "widget-numeric",
  "config": {
    "displayName": "SOG",
    "filterSelfPaths": true,
    "showLabel": true,
    "numDecimal": 1,
    "numInt": 3,
    "paths": {
      "numericPath": {
        "description": "Numeric Value",
        "path": "navigation.speedOverGround",
        "source": "default",
        "pathType": "number",
        "isPathConfigurable": true,
        "convertUnitTo": "knots",
        "sampleTime": 500
      }
    }
  }
}
```

### Text Display (`widget-text`)

Displays text/string values.

**Default size**: 6w x 6h
**Common paths**: String-type Signal K paths

```json
{
  "type": "widget-text",
  "config": {
    "displayName": "Status",
    "filterSelfPaths": true,
    "showLabel": true,
    "paths": {
      "stringPath": {
        "description": "String Value",
        "path": "navigation.state",
        "source": "default",
        "pathType": "string",
        "isPathConfigurable": true,
        "sampleTime": 1000
      }
    }
  }
}
```

### Position Display (`widget-position`)

GPS coordinates.

**Default size**: 8w x 6h

```json
{
  "type": "widget-position",
  "config": {
    "displayName": "Position",
    "filterSelfPaths": true,
    "showLabel": true,
    "paths": {
      "positionPath": {
        "description": "Position",
        "path": "navigation.position",
        "source": "default",
        "pathType": "number",
        "isPathConfigurable": true,
        "sampleTime": 1000
      }
    }
  }
}
```

### Date/Time (`widget-datetime`)

Timezone-aware date/time display.

**Default size**: 6w x 6h

```json
{
  "type": "widget-datetime",
  "config": {
    "displayName": "UTC Time",
    "showLabel": true,
    "paths": {
      "datetimePath": {
        "description": "DateTime",
        "path": "navigation.datetime",
        "source": "default",
        "pathType": "Date",
        "isPathConfigurable": true,
        "sampleTime": 1000
      }
    }
  }
}
```

### Static Label (`widget-label`)

Static text label for dashboard organization.

**Default size**: 4w x 4h

```json
{
  "type": "widget-label",
  "config": {
    "displayName": "Navigation",
    "showLabel": true
  }
}
```

### Radial Gauge (`widget-gauge-ng-radial`)

Radial/dial gauge for numeric data.

**Default size**: 12w x 12h
**Subtypes**: `basDefault` (standard), `basDonut` (donut)

```json
{
  "type": "widget-gauge-ng-radial",
  "config": {
    "displayName": "AWS",
    "filterSelfPaths": true,
    "showLabel": true,
    "numDecimal": 1,
    "paths": {
      "numericPath": {
        "description": "Numeric Value",
        "path": "environment.wind.speedApparent",
        "source": "default",
        "pathType": "number",
        "isPathConfigurable": true,
        "convertUnitTo": "knots",
        "sampleTime": 500
      }
    },
    "displayScale": {
      "lower": 0,
      "upper": 50,
      "type": "linear"
    },
    "gauge": {
      "type": "ngRadial",
      "subType": "basDefault",
      "enableTicks": true,
      "enableNeedle": true
    }
  }
}
```

### Linear Gauge (`widget-gauge-ng-linear`)

Horizontal or vertical bar gauge.

**Default size**: 6w x 8h

```json
{
  "type": "widget-gauge-ng-linear",
  "config": {
    "displayName": "Tank Level",
    "filterSelfPaths": true,
    "showLabel": true,
    "numDecimal": 0,
    "paths": {
      "numericPath": {
        "description": "Numeric Value",
        "path": "tanks.freshWater.0.currentLevel",
        "source": "default",
        "pathType": "number",
        "isPathConfigurable": true,
        "convertUnitTo": "ratio",
        "sampleTime": 1000
      }
    },
    "displayScale": {
      "lower": 0,
      "upper": 100,
      "type": "linear"
    },
    "gauge": {
      "type": "ngLinear",
      "subType": "basDefault",
      "enableTicks": true
    }
  }
}
```

### Compass Gauge (`widget-gauge-ng-compass`)

Marine compass card.

**Default size**: 12w x 12h

```json
{
  "type": "widget-gauge-ng-compass",
  "config": {
    "displayName": "COG",
    "filterSelfPaths": true,
    "showLabel": true,
    "numDecimal": 0,
    "paths": {
      "numericPath": {
        "description": "Numeric Value",
        "path": "navigation.courseOverGroundTrue",
        "source": "default",
        "pathType": "number",
        "isPathConfigurable": true,
        "convertUnitTo": "deg",
        "sampleTime": 500
      }
    },
    "gauge": {
      "type": "ngRadial",
      "subType": "basCompass",
      "compassUseNumbers": false,
      "enableTicks": true
    }
  }
}
```

### Steel Gauge (`widget-gauge-steel`)

Classic vintage-style gauge (steelseries library).

**Default size**: 12w x 12h

```json
{
  "type": "widget-gauge-steel",
  "config": {
    "displayName": "RPM",
    "filterSelfPaths": true,
    "showLabel": true,
    "paths": {
      "numericPath": {
        "description": "Numeric Value",
        "path": "propulsion.main.revolutions",
        "source": "default",
        "pathType": "number",
        "isPathConfigurable": true,
        "convertUnitTo": "rpm",
        "sampleTime": 500
      }
    },
    "displayScale": { "lower": 0, "upper": 4000, "type": "linear" },
    "gauge": { "type": "steel", "subType": "radial" }
  }
}
```

### Simple Linear Gauge (`widget-simple-linear`)

Compact horizontal bar with large value label.

**Default size**: 6w x 4h

```json
{
  "type": "widget-simple-linear",
  "config": {
    "displayName": "Battery",
    "filterSelfPaths": true,
    "showLabel": true,
    "numDecimal": 1,
    "paths": {
      "numericPath": {
        "description": "Numeric Value",
        "path": "electrical.batteries.house.voltage",
        "source": "default",
        "pathType": "number",
        "isPathConfigurable": true,
        "convertUnitTo": "V",
        "sampleTime": 1000
      }
    },
    "displayScale": { "lower": 10, "upper": 15, "type": "linear" }
  }
}
```

### Wind Steering (`widget-windsteer`)

Combined wind instrument display.

**Default size**: 12w x 18h
**Paths**: Multiple required paths for full functionality.

```json
{
  "type": "widget-windsteer",
  "config": {
    "displayName": "Wind",
    "filterSelfPaths": true,
    "showLabel": true,
    "windSectorEnable": true,
    "windSectorWindowSeconds": 5,
    "compassModeEnabled": false,
    "paths": {
      "headingPath": {
        "description": "Heading",
        "path": "navigation.courseOverGroundTrue",
        "source": "default",
        "pathType": "number",
        "isPathConfigurable": true,
        "convertUnitTo": "deg",
        "sampleTime": 500
      },
      "trueWindAngle": {
        "description": "True Wind Angle",
        "path": "environment.wind.angleTrueWater",
        "source": "default",
        "pathType": "number",
        "isPathConfigurable": true,
        "convertUnitTo": "deg",
        "sampleTime": 500
      },
      "apparentWindAngle": {
        "description": "Apparent Wind Angle",
        "path": "environment.wind.angleApparent",
        "source": "default",
        "pathType": "number",
        "isPathConfigurable": true,
        "convertUnitTo": "deg",
        "sampleTime": 500
      },
      "appWindSpeed": {
        "description": "App Wind Speed",
        "path": "environment.wind.speedApparent",
        "source": "default",
        "pathType": "number",
        "isPathConfigurable": true,
        "convertUnitTo": "knots",
        "sampleTime": 500
      },
      "trueWindSpeed": {
        "description": "True Wind Speed",
        "path": "environment.wind.speedTrue",
        "source": "default",
        "pathType": "number",
        "isPathConfigurable": true,
        "convertUnitTo": "knots",
        "sampleTime": 500
      }
    }
  }
}
```

### Data Chart (`widget-data-chart`)

Historical/realtime time-series chart (Chart.js).

**Default size**: 12w x 12h
**Note**: Requires `appConfig.dataSets` to be configured for historical data. Without datasets, shows live streaming only.

```json
{
  "type": "widget-data-chart",
  "config": {
    "displayName": "Speed Trend",
    "filterSelfPaths": true,
    "showLabel": true,
    "animateGraph": true,
    "showTimeScale": true,
    "showYScale": true,
    "startScaleAtZero": true,
    "timeScale": "minute",
    "period": 5,
    "paths": {
      "numericPath": {
        "description": "Numeric Value",
        "path": "navigation.speedOverGround",
        "source": "default",
        "pathType": "number",
        "isPathConfigurable": true,
        "convertUnitTo": "knots",
        "sampleTime": 1000
      }
    }
  }
}
```

### Boolean Switch (`widget-boolean-switch`)

Toggle switch for PUT-enabled boolean paths.

**Default size**: 6w x 6h

```json
{
  "type": "widget-boolean-switch",
  "config": {
    "displayName": "Anchor Light",
    "filterSelfPaths": true,
    "showLabel": true,
    "putEnable": true,
    "paths": {
      "booleanPath": {
        "description": "Boolean Value",
        "path": "electrical.switches.anchorLight.state",
        "source": "default",
        "pathType": "boolean",
        "isPathConfigurable": true,
        "sampleTime": 1000
      }
    }
  }
}
```

### Autopilot Head (`widget-autopilot`)

Remote autopilot control.

**Default size**: 12w x 18h

```json
{
  "type": "widget-autopilot",
  "config": {
    "displayName": "Autopilot",
    "showLabel": true,
    "autopilot": {
      "apiVersion": 2,
      "instanceId": "default"
    }
  }
}
```

### AIS Radar (`widget-ais-radar`)

AIS target display with range rings.

**Default size**: 12w x 18h

```json
{
  "type": "widget-ais-radar",
  "config": {
    "displayName": "AIS Radar",
    "showLabel": true,
    "ais": {
      "viewMode": "radar",
      "rangeRings": true
    }
  }
}
```

### Heel Gauge (`widget-heel-gauge`)

Dual-scale heel angle indicator.

**Default size**: 12w x 6h

```json
{
  "type": "widget-heel-gauge",
  "config": {
    "displayName": "Heel",
    "filterSelfPaths": true,
    "showLabel": true,
    "paths": {
      "numericPath": {
        "description": "Numeric Value",
        "path": "navigation.attitude.roll",
        "source": "default",
        "pathType": "number",
        "isPathConfigurable": true,
        "convertUnitTo": "deg",
        "sampleTime": 500
      }
    }
  }
}
```

### Horizon (Pitch & Roll) (`widget-horizon`)

Attitude indicator.

**Default size**: 12w x 12h

```json
{
  "type": "widget-horizon",
  "config": {
    "displayName": "Attitude",
    "filterSelfPaths": true,
    "showLabel": true,
    "paths": {
      "pitchPath": {
        "description": "Pitch",
        "path": "navigation.attitude.pitch",
        "source": "default",
        "pathType": "number",
        "isPathConfigurable": true,
        "convertUnitTo": "deg",
        "sampleTime": 500
      },
      "rollPath": {
        "description": "Roll",
        "path": "navigation.attitude.roll",
        "source": "default",
        "pathType": "number",
        "isPathConfigurable": true,
        "convertUnitTo": "deg",
        "sampleTime": 500
      }
    }
  }
}
```

### Zones State Panel (`widget-zones-state-panel`)

Multi-sensor health monitoring.

**Default size**: 12w x 8h

```json
{
  "type": "widget-zones-state-panel",
  "config": {
    "displayName": "Systems",
    "filterSelfPaths": true,
    "showLabel": true,
    "paths": {
      "numericPath": {
        "description": "Numeric Value",
        "path": "electrical.batteries.house.voltage",
        "source": "default",
        "pathType": "number",
        "isPathConfigurable": true,
        "sampleTime": 1000
      }
    }
  }
}
```

### Embedded Webpage (`widget-iframe`)

Embed external web apps (Grafana, Node-RED, etc.).

**Default size**: 12w x 12h

```json
{
  "type": "widget-iframe",
  "config": {
    "displayName": "Grafana",
    "showLabel": false,
    "widgetUrl": "http://localhost:4000/d/abc123/dashboard?orgId=1&kiosk",
    "allowInput": true
  }
}
```

### Freeboard-SK (`widget-freeboardsk`)

Chart plotter integration.

**Default size**: 24w x 24h (full screen recommended)

```json
{
  "type": "widget-freeboardsk",
  "config": {
    "displayName": "Chart",
    "showLabel": false
  }
}
```

## Common Unit Conversions

| Measurement | Signal K Unit (SI) | Common Display Unit | convertUnitTo |
|------------|-------------------|-------------------|--------------|
| Speed | m/s | knots | `"knots"` |
| Speed | m/s | km/h | `"km/h"` |
| Temperature | K | Celsius | `"celsius"` |
| Temperature | K | Fahrenheit | `"fahrenheit"` |
| Depth | m | meters | `"m"` |
| Depth | m | feet | `"ft"` |
| Angle | rad | degrees | `"deg"` |
| Pressure | Pa | hPa | `"hPa"` |
| Pressure | Pa | mmHg | `"mmHg"` |
| Voltage | V | volts | `"V"` |
| Current | A | amps | `"A"` |
| Power | W | watts | `"W"` |
| Volume | m3 | liters | `"liter"` |
| Distance | m | nm | `"nm"` |
| Frequency | Hz | rpm | `"rpm"` |
| Ratio | ratio | percent | `"percent"` |

## Natural Language Mapping

| User says | Widget type | Key config |
|-----------|------------|------------|
| "speed gauge" / "speedometer" | widget-gauge-ng-radial | SOG or STW path |
| "show my speed" / "speed number" | widget-numeric | SOG path, knots |
| "compass" / "heading" | widget-gauge-ng-compass | COG or heading path |
| "wind display" / "wind instrument" | widget-windsteer | Wind paths |
| "depth" / "depthsounder" | widget-numeric | depth.belowTransducer |
| "position" / "GPS" / "lat/lon" | widget-position | navigation.position |
| "chart" / "trend" / "graph" / "history" | widget-data-chart | Specified path |
| "tank level" / "fuel" / "water" | widget-gauge-ng-linear | tanks.* path |
| "battery" / "voltage" | widget-simple-linear or widget-numeric | electrical.batteries.* |
| "switch" / "toggle" / "light" | widget-boolean-switch | electrical.switches.* |
| "autopilot" / "AP" | widget-autopilot | autopilot config |
| "AIS" / "radar" / "traffic" | widget-ais-radar | AIS config |
| "chartplotter" / "map" | widget-freeboardsk | none |
| "embed" / "grafana" / "webpage" | widget-iframe | widgetUrl |
