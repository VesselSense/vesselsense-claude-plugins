---
name: wilhelmsk-dashboard
description: WilhelmSK dashboard layout generation, SignalK path gauge mapping, customTemplates geometry, iPhone iPad layout, gauge className type, Application Data API.
---

# WilhelmSK Dashboard Layout Generator

## Overview

Generate WilhelmSK dashboard layouts from SignalK paths. Maps SignalK data paths to WilhelmSK gauge types, organizes them into pages, and produces the JSON structure that WilhelmSK reads from the SignalK Application Data API.

Works with any SignalK server — no vessel-specific knowledge required.

## Keyword Index

| Keyword | File |
|---------|------|
| gauge className, mapping rules, type field | gauge-mapping.md |
| customTemplates, geometry, pixel positions | templates.md |
| page categorization, path grouping | SKILL.md (Path Categorization) |
| template selection, page layout | SKILL.md (Template Selection) |
| JSON output, deployment, Application Data API | SKILL.md (Output & Deployment) |

## Workflow

### Prerequisites

The agent must first collect SignalK paths from the target server:
```
GET /signalk/v1/api/vessels/self
```
Walk the nested JSON tree to extract all paths with current values.

### Interactive Steps

1. **Ask device type**: iPhone (`Main`), iPad landscape (`iPad`), or both
2. **Ask layout name**: User picks a name (e.g., "Dashboard", "Remote Monitoring")
3. **Auto-categorize paths** into pages — present to user for approval/adjustment
4. **Map paths to gauges** using rules in `gauge-mapping.md`
5. **Select page templates** based on gauge count (see Template Selection below)
6. **Generate JSON** and instruct agent on deployment

## Path Categorization

Group discovered paths into pages by prefix:

| Path Prefix | Page Name |
|---|---|
| `electrical.grid.*` | AC Electrical |
| `electrical.batteries.*` | DC Electrical |
| `electrical.solar.*`, `electrical.venus.*` | DC Electrical |
| `electrical.switches.*` | Switches |
| `electrical.pumps.*` | Bilge |
| `environment.wind.*` | Sailing |
| `environment.depth.*` | Navigation |
| `environment.water.temperature` | Sailing |
| `environment.inside.*`, `environment.outside.*`, `environment.refrigerator.*` | Environment |
| `environment.engineroom.*` | Engine |
| `environment.bilge.*` | Bilge |
| `navigation.heading*`, `navigation.courseOverGround*` | Navigation |
| `navigation.speed*` | Navigation |
| `navigation.position` | Navigation |
| `navigation.attitude` | Sailing (expands to 3 gauges: Roll, Pitch, Yaw) |
| `navigation.courseGreatCircle.*`, `navigation.course.calcValues.*` | Navigation |
| `navigation.log`, `navigation.trip.*` | Navigation |
| `navigation.gnss.*` | System |
| `navigation.rateOfTurn` | Navigation |
| `propulsion.*` | Engine |
| `tanks.*` | Tanks |
| `steering.autopilot.*` | Switches |
| `computer.*` | System |
| `design.*`, `name`, `mmsi`, `communication.*` | Vessel Info |
| `notifications.*` | (skip) |
| `sensors.*` | (skip) |

**Rules:**
- Drop empty pages
- Merge pages with 1-2 gauges into a related page
- `navigation.attitude` → always 3 gauges (Roll, Pitch, Yaw) on the same page
- `steering.autopilot.*` → one `AutoPilotControl` gauge (not individual sub-paths)
- Skip pure metadata (`*.name`, `design.*`, `sensors.*`) unless user wants a Vessel Info page

Present proposed groupings to the user for confirmation before proceeding.

## Template Selection

### iPhone (`Main`)

| Gauge Count | Template | Grid | Notes |
|---|---|---|---|
| 1-3 | `Three` | 2 small + 1 hero | Good for switches + autopilot |
| 4-6 | `TwoByThree` | 2x3 | Medium gauges |
| 7-8 | `6x6` | 2x4 | Best general-purpose |
| 9-10 | `LargeBottomView` | 3x3 + hero | Last position = hero gauge |
| 11-12 | `BottomTextView` | 3x3 + bottom | 12 positions |
| 13-15 | `ThreeByFive` | 3x5 | Compact |
| 16-24 | `1` | 4x6 | Dense — avoid unless necessary |

### iPad (`iPad`)

| Gauge Count | Template | Grid | Notes |
|---|---|---|---|
| 1-15 | `Basic` | 5x3 + extras | General purpose |
| 16 | `FourByFour` | 4x4 | Clean grid |
| 17-49 | `3` | 7x7 | Full density |

### Hero Gauges

For templates with a hero position (`LargeBottomView`, `Three`, `BottomTextView`), the **last position** renders larger. Assign the most important gauge:
- DC Electrical → SOC
- Engine → RPM
- Sailing → AWA
- Switches → AutoPilotControl

### Empty Positions

Fill unused positions with `{"className": "EmptyGauge"}`. Position numbering starts at `"2"` (position "2" = gauges[0]).

## Output & Deployment

### JSON Structure

```json
{
  "deviceType": "Main",
  "layoutName": "User Chosen Name",
  "layout": {
    "pages": [
      {
        "pageLayout": "6x6",
        "config": {
          "2": {"className": "TextGaugeConfig", "title": "Grid Voltage", "path": "electrical.grid.1.l1.voltage", "type": 1},
          "3": {"className": "AmpsGauge", "title": "Grid Current", "path": "electrical.grid.1.l1.current", "type": 1},
          "9": {"className": "EmptyGauge"}
        }
      }
    ],
    "customTemplates": {}
  }
}
```

### Gauge Config Fields

- `className` (required): WilhelmSK gauge class
- `title` (required except EmptyGauge): Short label (max ~20 chars)
- `path` (required unless built-in default): Full SignalK path
- `type` (optional): 0=analog dial, 1=digital LCD, 4=bar/circle. Omit for class default.

Gauges with built-in defaults (no `path` needed): `AWAGauge`, `TWAGauge`, `AutoPilotControl`, `EmptyGauge`, `Map`

### customTemplates

Include only templates used by the layout's pages. Copy geometry from `templates.md`.

### Deployment

1. **GET** existing template: `/signalk/v1/applicationData/global/WilhelmSK/1.0.0`
2. **Merge** at `template.layouts[deviceType][layoutName] = generatedLayout`
3. **PUT** merged template back to same endpoint (requires JWT auth)
4. User must **force-quit and relaunch** WilhelmSK app

### Device Type Filtering

WilhelmSK filters layouts by device:
- iPhone sees only `Main.*`
- iPad landscape sees only `iPad.*`
- iPad portrait sees only `iPadPortrate.*`

