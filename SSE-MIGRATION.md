# Migrating to SSE Output Format

The MQTT topic has changed its JSON format from polling mode (array of meter objects) to SSE mode (named object with per-phase data). Data now arrives ~1/second instead of every 30 seconds.

## Field Mapping (old → new)

| Old field | New field |
|-----------|-----------|
| `activePower` | `p` |
| `apparentPower` | `s` |
| `reactivePower` | `q` |
| `voltage` | `v` |
| `current` | `i` |
| `pwrFactor` | `pf` |
| `freq` | `f` |

## Access Pattern Change

### Old format (polling)

```json
[
  {
    "eid": 704643328,
    "activePower": 9054.577,
    "apparentPower": 9092.237,
    "reactivePower": -568.975,
    "pwrFactor": 0.996,
    "voltage": 241.503,
    "current": 37.664,
    "freq": 50.125
  },
  {
    "eid": 704643584,
    "activePower": -988.367,
    "apparentPower": 1376.75,
    "reactivePower": 47.465,
    "pwrFactor": -0.727,
    "voltage": 241.679,
    "current": 5.694,
    "freq": 50.125
  }
]
```

- Production power: `data[0]["activePower"]`
- Net-consumption power: `data[1]["activePower"]`

### New format (SSE)

```json
{
  "production": {
    "ph-a": {"p": 9056.6, "q": -568.9, "s": 9094.5, "v": 241.5, "i": 37.6, "pf": 1.0, "f": 50.0},
    "ph-b": {"p": 0.0, "q": 0.0, "s": 0.0, "v": 0.0, "i": 0.0, "pf": 0.0, "f": 0.0},
    "ph-c": {"p": 0.0, "q": 0.0, "s": 0.0, "v": 0.0, "i": 0.0, "pf": 0.0, "f": 0.0}
  },
  "net-consumption": {
    "ph-a": {"p": -1002.5, "q": 57.9, "s": -1392.2, "v": 241.7, "i": -5.7, "pf": -1.0, "f": 50.0},
    "ph-b": {"p": 0.0, "q": 0.0, "s": 0.0, "v": 0.0, "i": 0.0, "pf": 0.0, "f": 0.0},
    "ph-c": {"p": 0.0, "q": 0.0, "s": 0.0, "v": 0.0, "i": 0.0, "pf": 0.0, "f": 0.0}
  },
  "total-consumption": {
    "ph-a": {"p": 8054.1, "q": -511.0, "s": 7713.9, "v": 241.7, "i": 31.9, "pf": 1.0, "f": 50.0},
    "ph-b": {"p": 0.0, "q": 0.0, "s": 0.0, "v": 0.0, "i": 0.0, "pf": 0.0, "f": 0.0},
    "ph-c": {"p": 0.0, "q": 0.0, "s": 0.0, "v": 0.0, "i": 0.0, "pf": 0.0, "f": 0.0}
  }
}
```

- Production power: `data["production"]["ph-a"]["p"]`
- Net-consumption power: `data["net-consumption"]["ph-a"]["p"]`
- Total-consumption power: `data["total-consumption"]["ph-a"]["p"]` *(new — wasn't directly available before)*

For single-phase systems, always use `ph-a`. Unused phases have values of `0.0`.

## What's no longer available via SSE

The following fields from the polling format are **not present** in SSE output:

- `timestamp`
- `actEnergyDlvd` / `actEnergyRcvd` (cumulative energy delivered/received)
- `apparentEnergy`
- `reactEnergyLagg` / `reactEnergyLead`
- `channels` array (per-channel breakdowns)
- `eid` (meter identifiers)

If you need these fields, set `USE_SSE: false` and `PUBLISH_INTERVAL: 30`.
