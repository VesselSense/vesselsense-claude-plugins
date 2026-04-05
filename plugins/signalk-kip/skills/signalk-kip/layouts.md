# KIP Layout Patterns

## Grid System

- **24 columns** wide, rows auto-extend
- Position: `x` (column, 0-23), `y` (row, 0+)
- Size: `w` (width in columns), `h` (height in rows)
- Widgets cannot overlap; GridStack will auto-shift

## Standard Layout Templates

### Navigation Dashboard

4 numeric readouts top, 2 large gauges middle, 3 info panels bottom.

```
 x0        x6        x12       x18       x24
y0  [SOG 6x6] [STW 6x6] [Depth 6x6] [AWA 6x6]
y6  [  COG Compass 12x12 ] [ AWS Radial 12x12  ]
y18 [ VMG 8x6  ] [Position 8x6] [  XTE 8x6   ]
```

### Wind Dashboard

Large wind instrument left, chart and numerics right.

```
 x0           x12          x18       x24
y0  [  Windsteer 12x18  ] [ Data Chart 12x12  ]
                           [ TWS 6x6 ][ TWA 6x6]
y18                        [ SOG 6x6 ][ VMG 6x6]
```

### Engine Dashboard

RPM gauge, temperatures, pressures.

```
 x0        x6        x12       x18       x24
y0  [   RPM Radial 12x12    ] [Coolant 6x6][Oil P 6x6]
                               [Volts  6x6][Hours 6x6]
y12 [ Exhaust Chart 12x12   ] [Fuel L 12x12          ]
```

### Electrical Dashboard

Battery status, solar, shore power.

```
 x0        x6        x12       x18       x24
y0  [HouseV 8x6 ][HouseA 8x6 ][SOC  8x6  ]
y6  [Solar Chart 12x12      ] [Shore Chart 12x12   ]
y18 [SolarV 6x6][SolarA 6x6] [ChargeA 6x6][Load 6x6]
```

### Sailing Performance Dashboard

Wind, speed, VMG, polar performance.

```
 x0           x12          x18       x24
y0  [  Windsteer 12x18  ] [SOG 6x6  ][STW 6x6 ]
                           [VMG 6x6  ][TWS 6x6 ]
                           [AWA 6x6  ][TWA 6x6 ]
y18 [   Speed Chart 12x6              ][Heel 12x6]
```

### Overview / Status Dashboard

High-density information display.

```
 x0     x4     x8     x12    x16    x20    x24
y0 [SOG][STW][Depth][AWA ][AWS ][COG ]
y6 [Pos   8x6     ][VMG ][XTE ][Time  8x6  ]
y12[   Zones State Panel 24x6                ]
y18[Battery 8x6   ][Tank  8x6  ][Wind  8x6  ]
```

### Racing Dashboard

```
 x0           x12                    x24
y0  [Race Timer 12x6    ] [Start Line 12x6    ]
y6  [  Windsteer 12x12  ] [SOG 6x6  ][COG 6x6]
                           [VMG 6x6  ][TWA 6x6]
y18 [  Speed Chart 24x6                       ]
```

### Chartplotter Dashboard

Full-screen chart with minimal overlays.

```
 x0                                  x24
y0  [   Freeboard-SK 24x24                    ]
```

Or split mode with `splitShellEnabled: true` in appConfig.

## Layout Sizing Guidelines

| Widget Type | Minimum | Recommended | Maximum |
|------------|---------|-------------|---------|
| Numeric | 4x4 | 6x6 | 8x8 |
| Text | 4x4 | 6x6 | 12x6 |
| Position | 6x4 | 8x6 | 12x6 |
| Label | 4x2 | 4x4 | 24x4 |
| Radial Gauge | 8x8 | 12x12 | 16x16 |
| Compass | 8x8 | 12x12 | 16x16 |
| Linear Gauge | 4x6 | 6x8 | 12x12 |
| Simple Linear | 4x3 | 6x4 | 12x6 |
| Windsteer | 10x14 | 12x18 | 16x20 |
| Data Chart | 8x8 | 12x12 | 24x16 |
| Autopilot | 10x14 | 12x18 | 16x20 |
| AIS Radar | 10x14 | 12x18 | 24x24 |
| Boolean Switch | 4x4 | 6x6 | 8x8 |
| Iframe | 8x8 | 12x12 | 24x24 |
| Freeboard-SK | 12x12 | 24x24 | 24x24 |
| Zones Panel | 8x6 | 12x8 | 24x12 |
| Heel Gauge | 8x4 | 12x6 | 16x8 |
| Horizon | 8x8 | 12x12 | 16x16 |

## UUID Generation

Every widget and dashboard needs a UUID v4. Use this pattern in evaluate_script:

```javascript
function uuid() {
  return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, c => {
    const r = Math.random() * 16 | 0;
    return (c === 'x' ? r : (r & 0x3 | 0x8)).toString(16);
  });
}
```

## Widget Factory Pattern

Reusable function for evaluate_script injection:

```javascript
function makeWidget(type, x, y, w, h, config) {
  const id = uuid();
  return {
    w, h, x, y, id,
    selector: 'widget-host2',
    input: {
      widgetProperties: {
        type,
        uuid: id,
        config: config || {}
      }
    }
  };
}
```

## Dashboard Factory Pattern

```javascript
function makeDashboard(name, icon, widgets) {
  return {
    id: uuid(),
    name,
    icon: icon || 'dashboard-dashboard',
    collapseSplitShell: false,
    configuration: widgets
  };
}
```
