---
description: Configure KIP dashboard visualizations from natural language. Creates, modifies, or replaces dashboard layouts with browser-validated widgets.
---

## User Request

```text
$ARGUMENTS
```

## Instructions

You are automating KIP (@mxtommy/kip) dashboard configuration. Follow this workflow precisely.

### Prerequisites

Load the skill knowledge base by reading these files:
1. `.claude/skills/signalk-kip/SKILL.md` - Architecture and validation strategy
2. `.claude/skills/signalk-kip/widgets.md` - Widget catalog with config templates
3. `.claude/skills/signalk-kip/layouts.md` - Layout patterns and grid system
4. `.claude/skills/signalk-kip/config-schema.md` - Full config schema reference

### Workflow

#### Phase 1: Discover Available Data

Query the local Signal K server to find available paths:

```bash
curl -s http://localhost:3000/signalk/v1/api/vessels/self 2>/dev/null | python3 -c "
import json, sys
data = json.load(sys.stdin)
def get_paths(obj, prefix=''):
    paths = []
    for k, v in obj.items():
        if k in ('meta', '\$source', 'timestamp', 'sentence', 'pgn'): continue
        path = f'{prefix}.{k}' if prefix else k
        if isinstance(v, dict) and 'value' in v: paths.append(path)
        elif isinstance(v, dict): paths.extend(get_paths(v, path))
    return paths
for p in sorted(get_paths(data)): print(p)
"
```

#### Phase 2: Translate Request to Config

Map the user's natural language request to KIP widget configurations using:
- `widgets.md` for widget type selection and config templates
- `layouts.md` for grid positioning and sizing
- Available Signal K paths from Phase 1

#### Phase 3: Ensure KIP Is Loaded

Check if KIP is already open in Chrome:

```
mcp__chrome-devtools__list_pages
```

If not at `/@mxtommy/kip`, navigate there:

```
mcp__chrome-devtools__navigate_page({ type: 'url', url: 'http://localhost:3000/@mxtommy/kip' })
```

#### Phase 4: Read Current Config

```javascript
// Via evaluate_script
() => {
  return JSON.stringify({
    dashboards: JSON.parse(localStorage.getItem('dashboardsConfig')),
    app: JSON.parse(localStorage.getItem('appConfig')),
    conn: JSON.parse(localStorage.getItem('connectionConfig'))
  });
}
```

Decide whether to:
- **Add** a new dashboard to existing config
- **Modify** widgets in an existing dashboard
- **Replace** entire dashboard config

#### Phase 5: Inject Configuration

Build the complete config JSON and inject via `evaluate_script`. Always ensure `signalKSubscribeAll: true`.

#### Phase 6: Reload and Validate

1. Reload KIP: `navigate_page({ type: 'reload' })`
2. Wait 3 seconds for Angular to initialize
3. Run structural validation via `evaluate_script`:

```javascript
() => {
  const dashConfig = JSON.parse(localStorage.getItem('dashboardsConfig'));
  const gridItems = document.querySelectorAll('.grid-stack-item');
  const idx = (window.location.hash.match(/dashboard\/(\d+)/) || [,0])[1];
  const widgets = [];
  gridItems.forEach(item => {
    const gsId = item.getAttribute('gs-id');
    const cfg = dashConfig[idx]?.configuration?.find(w => w.id === gsId);
    widgets.push({
      type: cfg?.input?.widgetProperties?.type,
      name: cfg?.input?.widgetProperties?.config?.displayName,
      rendered: item.querySelectorAll('canvas').length > 0,
      grid: { w: item.getAttribute('gs-w'), h: item.getAttribute('gs-h') }
    });
  });
  return JSON.stringify({
    dashboard: dashConfig[idx]?.name,
    expected: dashConfig[idx]?.configuration?.length,
    rendered: gridItems.length,
    allRendered: widgets.every(w => w.rendered),
    widgets
  });
}
```

4. If validating multiple dashboards, navigate between them with:
   `press_key({ key: 'Shift+Control+ArrowDown' })` and re-validate each.

#### Phase 7: Report

Report the results:
- Dashboards created/modified
- Widgets placed with types and bound paths
- Validation status (all rendered, grid positions correct)
- Any issues found

### Validation Rules

- **DOM-first**: Use `evaluate_script` for all validation. Screenshots only as final spot-check if explicitly requested.
- **Canvas rendering**: KIP widgets render to `<canvas>`, not DOM text. Validate by canvas element presence, not text content.
- **Config source of truth**: `localStorage.getItem('dashboardsConfig')` is always authoritative.
- **Data flow**: Confirm paths exist via `curl localhost:3000/signalk/v1/api/vessels/self/{path}/value`.

### Error Recovery

If validation fails (widget count mismatch, missing canvases):
1. Re-read localStorage to check if config persisted
2. Check browser console for errors: `list_console_messages({ types: ['error'] })`
3. If config is correct but rendering failed, try another reload
4. If config is wrong, re-inject and reload
