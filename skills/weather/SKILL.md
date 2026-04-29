---
name: weather
description: Weather lookup skill using curl and wttr.in API for fetching current weather and forecasts.
version: 1.0.0
---

# Weather Lookup (curl + wttr.in)

## How to Use

### 1. Simple Weather (Current Conditions + Forecast)
```bash
curl wttr.in/LOCATION
```

**Example:** `curl wttr.in/Canberra`  → Returns detailed ASCII weather report

### 2. JSON Format (Parsed Data)
```bash
curl wttr.in/LOCATION?format=json
```

Use `jq` to parse: `curl wttr.in/Canberra?format=json | jq`

### 3. Specific Location Formats

| Query | Example |
|-------|---------|
| City name | `curl wttr.in/Paris` |
| City+Country | `curl wttr.in/Moscow` |
| City+State | `curl wttr.in/Kansas+City` (quote if spaces) |
| City+Region | `curl wttr.in/California` |
| Airport code | `curl wttr.in/KYUL` (YVR, SYD, etc.) |
| Zip code | `curl wttr.in/10001` |
| Lat/Lon | `curl wttr.in/-35.28,149.13` |
| Current location | `curl wttr.in` (auto-detect) |

### 4. Customization Options

| Option | Description | Example |
|--------|-------------|---------|
| No icons | Clean text output | `curl wttr.in/London\?o` |
| Meter | Metric (default) | Already standard |
| Fahrenheit | Temp in °F | `curl wttr.in/London\?F` |
| 5-day forecast | Extended forecast | `curl wttr.in/London\?5` |
| Hourly | Hourly forecast | `curl wttr.in/London\?u` |

### 5. Best Practices

```bash
# For detailed weather report (recommended)
curl -s wttr.in/Canberra

# For machine-readable JSON
curl -s "wttr.in/Canberra?format=json"

# With verbose debugging (if needed)
curl -v wttr.in/Canberra
```

### 6. Sample Output
```
Weather report: canberra

  Partly cloudy          +13°C
  ↑ 15 km/h
  0.0 mm

Today: Sunny morning, 9-16°C
Tomorrow: Sunny all day, 11-19°C
```

## When to Use This Skill

✅ Weather queries (current conditions, forecasts)  
✅ Temperature for specific locations  
✅ Climate/weather planning  
✅ Trip preparation and packing advice  
✅ Seasonal weather expectations  

## Notes

- wttr.in is free, requires no API key
- Returns ASCII art + detailed weather data
- Most endpoints support JSON output
- Quote locations with spaces: `"Los+Angeles"`
