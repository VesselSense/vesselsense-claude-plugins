---
name: signalk-kip
description: KIP dashboard automation, widget configuration, localStorage injection, browser validation via Chrome DevTools MCP.
triggers:
  - kip
  - dashboard
  - instrument panel
  - widget
  - gauge
  - MFD
---

# KIP Dashboard Automation

## Overview

KIP (@mxtommy/kip v4.5.0) is an Angular-based Multi-Function Display webapp bundled with Signal K Server. This skill enables fully automated dashboard development: natural language requests are translated into KIP configuration JSON, injected via browser localStorage, and validated through DOM queries.

## Architecture

```
User Request (natural language)
  |
  v
Skill: Translate to KIP Config JSON
  |
  v
Chrome DevTools MCP: evaluate_script -> localStorage.setItem()
  |
  v
Chrome DevTools MCP: navigate_page (reload)
  |
  v
Chrome DevTools MCP: evaluate_script -> DOM validation
  |
  v
(optional) Chrome DevTools MCP: take_screenshot -> visual spot-check
```

## KIP Access

- **URL**: `{SIGNALK_URL}/@mxtommy/kip` (ask the user for their Signal K server URL if not already known, e.g. `http://localhost:3000` or `http://nmea.local:3000`)
- **Config storage**: Browser `localStorage` (default) or Signal K Application Data API (shared mode)
- **Config version**: 12
- **Rendering**: Canvas-based (canvas-gauges, Chart.js) - DOM text queries return empty; validate structurally

## Configuration Model

KIP stores config across four localStorage keys:

| Key | Purpose |
|-----|---------|
| `connectionConfig` | Server URL, auth, subscribe settings |
| `appConfig` | Units, night mode, datasets, notifications |
| `dashboardsConfig` | Array of Dashboard objects with widget grids |
| `themeConfig` | Theme name |

### Critical Setting

`connectionConfig.signalKSubscribeAll` MUST be `true` for widgets to receive data. Always set this during injection.

## Automation Workflow

### 1. Inject Configuration

```javascript
// Via Chrome DevTools evaluate_script
() => {
  // Build config
  const dashboards = [/* ... */];
  localStorage.setItem('dashboardsConfig', JSON.stringify(dashboards));

  // Ensure data subscription
  const conn = JSON.parse(localStorage.getItem('connectionConfig'));
  conn.signalKSubscribeAll = true;
  localStorage.setItem('connectionConfig', JSON.stringify(conn));

  return JSON.stringify({ status: 'injected', count: dashboards.length });
}
```

### 2. Reload KIP

```
mcp__chrome-devtools__navigate_page({ type: 'reload' })
```

### 3. Validate (DOM-first, screenshots last resort)

```javascript
// Structural validation via evaluate_script
() => {
  const dashConfig = JSON.parse(localStorage.getItem('dashboardsConfig'));
  const gridItems = document.querySelectorAll('.grid-stack-item');
  const canvasEls = document.querySelectorAll('canvas');
  const currentDash = window.location.hash.match(/dashboard\/(\d+)/);
  const idx = currentDash ? parseInt(currentDash[1]) : 0;

  const widgets = [];
  gridItems.forEach(item => {
    const gsId = item.getAttribute('gs-id');
    const cfg = dashConfig[idx]?.configuration?.find(w => w.id === gsId);
    widgets.push({
      type: cfg?.input?.widgetProperties?.type,
      name: cfg?.input?.widgetProperties?.config?.displayName,
      rendered: item.querySelectorAll('canvas').length > 0,
      grid: {
        w: item.getAttribute('gs-w'),
        h: item.getAttribute('gs-h'),
        x: item.getAttribute('gs-x'),
        y: item.getAttribute('gs-y')
      }
    });
  });

  return JSON.stringify({
    dashboard: dashConfig[idx]?.name,
    expectedWidgets: dashConfig[idx]?.configuration?.length,
    renderedWidgets: gridItems.length,
    allCanvasPresent: widgets.every(w => w.rendered),
    widgets
  });
}
```

### 4. Navigate Between Dashboards

```
// Keyboard shortcut
mcp__chrome-devtools__press_key({ key: 'Shift+Control+ArrowDown' })  // next
mcp__chrome-devtools__press_key({ key: 'Shift+Control+ArrowUp' })    // prev
```

## Validation Strategy

KIP renders widgets to `<canvas>` elements, NOT DOM text nodes. Standard innerText queries return empty.

| What to validate | Method |
|-----------------|--------|
| Config persisted correctly | `localStorage.getItem()` via evaluate_script |
| Widget count matches config | `document.querySelectorAll('.grid-stack-item').length` |
| Widgets initialized | Each grid item has `<canvas>` child(ren) |
| Grid positions correct | `gs-w`, `gs-h`, `gs-x`, `gs-y` attributes on grid items |
| Widget types match | Map `gs-id` back to `dashboardsConfig[].configuration[].id` |
| Data flowing | Query Signal K REST API via curl: `curl {SIGNALK_URL}/signalk/v1/api/vessels/self/{path}/value` |
| Visual rendering | `take_screenshot` (sparingly - only for final spot-checks) |

## Reference Files

- Widget catalog: `signalk-kip/widgets.md`
- Config schema: `signalk-kip/config-schema.md`
- Layout patterns: `signalk-kip/layouts.md`
