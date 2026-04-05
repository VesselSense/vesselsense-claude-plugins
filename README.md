# VesselSense Claude Plugins

Claude Code plugin marketplace for [Signal K](https://signalk.org/) marine development.

## Installation

Add the marketplace (one time):

```
/plugin marketplace add VesselSense/vesselsense-claude-plugins
```

Then install the plugins you need:

```
/plugin install signalk-kip@vesselsense-claude-plugins
/plugin install signalk-sensesp@vesselsense-claude-plugins
/plugin install wilhelmsk-dashboard@vesselsense-claude-plugins
```

## Plugins

### signalk-kip

KIP dashboard automation. Translates natural language requests into KIP widget configurations, injects them via Chrome DevTools MCP, and validates rendering through DOM queries.

**Requires:** Chrome DevTools MCP server

**Skills:** `signalk-kip` (model-invoked on triggers: kip, dashboard, widget, gauge, MFD)
**Commands:** `/kip` (autonomous 7-phase dashboard workflow)

### signalk-sensesp

SensESP framework reference for building custom marine sensors on ESP32 microcontrollers. Covers sensor types, transform chains, PlatformIO configuration, and Signal K integration patterns.

**Skills:** `signalk-sensesp`

### wilhelmsk-dashboard

WilhelmSK dashboard layout generation. Maps SignalK data paths to WilhelmSK gauge types, organizes them into pages, and produces JSON layouts for iPhone and iPad via the Signal K Application Data API.

**Skills:** `wilhelmsk-dashboard`

## Contributing

To add a new plugin:

1. Create a directory under `plugins/` with a `.claude-plugin/plugin.json` manifest
2. Add skills under `skills/` and commands under `commands/`
3. Register the plugin in `.claude-plugin/marketplace.json`
4. Open a pull request

## License

MIT
