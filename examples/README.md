# Integration Examples

This directory contains ready-to-use configuration examples for Home Assistant and OpenHAB, organised by data mode.

## Which mode am I using?

| Setting | Mode | Data source |
|---------|------|-------------|
| `USE_SSE: true` (default) | **SSE** | `/stream/meter` — real-time event stream |
| `USE_SSE: false` | **Polling** | `/ivp/meters/readings` or `/ivp/livedata/status` |

## Directory layout

```
examples/
├── sse/                        # USE_SSE: true (default, recommended)
│   ├── home-assistant.yaml     # HA sensor templates
│   └── openhab/
│       └── solar.things        # OpenHAB MQTT thing definition
└── polling/                    # USE_SSE: false
    ├── home-assistant.yaml     # HA sensor templates (meters/readings + livedata)
    └── openhab/
        └── solar.things        # OpenHAB MQTT thing definition (livedata format)
```

## SSE mode (`examples/sse/`)

SSE streams data from the Envoy at ~1 event/second with negligible load on the device. The JSON structure uses named keys:

```json
{
  "production":        { "ph-a": { "p": 451.2, "v": 244.5, "i": 2.1, "pf": 0.87, "f": 50.0 } },
  "net-consumption":   { "ph-a": { "p": -362.2, "v": 244.4, ... } },
  "total-consumption": { "ph-a": { "p": 89.1, "v": 244.4, ... } }
}
```

Values are in real units (watts, volts, amps). No conversion needed.

## Polling mode (`examples/polling/`)

Polling fetches data on a schedule (see `PUBLISH_INTERVAL`, minimum 30 s recommended). Two variants exist depending on your firmware and battery setup:

**Without batteries** — `/ivp/meters/readings` returns an array:
```json
[
  { "activePower": 451.2, "voltage": 244.5, "current": 2.1, "pwrFactor": 0.87, ... },
  { "activePower": 89.1,  "voltage": 244.4, ... }
]
```

**With batteries** — `/ivp/livedata/status` returns an object with milliwatt values:
```json
{
  "meters": {
    "pv":      { "agg_p_mw": 451279 },
    "grid":    { "agg_p_mw": -362175 },
    "storage": { "agg_p_mw": 0 }
  }
}
```

The polling `home-assistant.yaml` includes both variants (Option B is commented out — uncomment it if you have batteries).

## OpenHAB shared files

The OpenHAB items, rules, and sitemap files in [`openhab/`](../openhab/) are designed for the **polling** (livedata/milliwatt) format. If you use SSE mode, the SSE `solar.things` maps values directly in watts, so the milliwatt-to-watt conversion rule can be simplified or removed.

## Three-phase systems

All examples show single-phase (`ph-a`). For three-phase, duplicate the sensors for `ph-b` and `ph-c` following the same pattern.
