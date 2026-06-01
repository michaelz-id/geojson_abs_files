# LGA Boundaries Interactive Map — Design Spec

**Date:** 2026-06-01  
**Status:** Approved

---

## Overview

A single-page interactive map website that renders Australian Local Government Area (LGA) boundaries from a GeoJSON file. Users can zoom, hover to see LGA details in a popup, and search by LGA name to zoom directly to a region.

---

## Inputs

| File | Location | Size | Key properties |
|------|----------|------|----------------|
| `lga_boundaries.geojson` | `C:\Users\michaelz\source\repos\claude\demographic-indicators\` | ~4.7 MB | `lga_code`, `lga_name`, `state_code`, `state_name`, `area_sqkm` |

The GeoJSON is fetched at runtime via `fetch()` from the same directory as `index.html`. It is not bundled into the HTML.

---

## Deployment

- **Output:** Single `index.html` file
- **Dependencies:** Leaflet JS + CSS via CDN (no install, no build step)
- **Runtime:** Open directly in a browser, or serve from any static web server
- **Offline:** Works offline once loaded (no tile server dependency)

---

## Layout

**Top bar** (fixed height, dark `#2c3e50` background):
- Left: site title "🗺 Australian LGA Map"
- Centre: search input ("Search LGA name…") with magnifier icon
- Right: LGA count label ("548 LGAs · Australia")

**Map** (fills remaining viewport height):
- Leaflet map initialised to Australia's bounding box
- No base tile layer — polygons render on a plain dark background (`#1a2b3c`)
- Zoom controls (Leaflet default, top-left)
- State legend control (bottom-right)

---

## Colour Scheme

Each LGA polygon is coloured by `state_code`. Hover highlight colour is `#f39c12` (orange). Border strokes are a lighter tint of the fill.

| State/Territory | `state_code` | Fill colour |
|-----------------|-------------|-------------|
| NSW | 1 | `#3498db` |
| VIC | 2 | `#27ae60` |
| QLD | 3 | `#e67e22` |
| SA | 4 | `#e74c3c` |
| WA | 5 | `#8e44ad` |
| TAS | 6 | `#e91e8c` |
| NT | 7 | `#16a085` |
| ACT | 8 | `#7f8c8d` |

---

## Hover Popup

Triggered on `mouseover`, dismissed on `mouseout`. Content:

```
[lga_name]
LGA Code: [lga_code]
[state_name] · [area_sqkm] km²
```

Implemented as a Leaflet `Popup` bound via `onEachFeature`. The hovered polygon's fill switches to the highlight colour; it reverts on `mouseout`.

---

## Search

- Input in the top bar listens on `input` event (real-time, no submit required)
- Case-insensitive substring match against `lga_name` across all GeoJSON layers
- On match: `map.fitBounds(layer.getBounds())` zooms to the LGA and opens its popup
- No match: no visual change (input remains active)
- Clearing the input resets the map view to Australia's full bounding box

---

## State Legend

Leaflet `Control` positioned `bottomright`. Shows a colour swatch + label for each of the 8 states/territories. Dark semi-transparent background to contrast against the map.

---

## Components (all in `index.html`)

| Section | Responsibility |
|---------|---------------|
| `<head>` | CDN links for Leaflet JS + CSS |
| `<style>` | Top bar layout, search input styling, legend styling |
| `#map` div | Leaflet mount point, fills viewport below top bar |
| `initMap()` | Creates Leaflet map, sets bounds, adds GeoJSON layer and legend |
| `styleFeature(feature)` | Returns Leaflet path options based on `state_code` |
| `onEachFeature(feature, layer)` | Binds hover handlers and popup to each LGA polygon |
| `initSearch()` | Wires search input to layer scan + fitBounds |
| DOMContentLoaded | `fetch()` GeoJSON → `initMap()` → `initSearch()` |

---

## Data Flow

```
Page load
  → fetch('lga_boundaries.geojson')
  → parse JSON
  → L.geoJSON(data, { style: styleFeature, onEachFeature })
  → add to map

Search input (oninput)
  → scan all layers for lga_name match (case-insensitive)
  → match found: map.fitBounds + openPopup
  → input cleared: map.fitBounds(australiaBounds)
```

---

## Out of Scope

- Base tile layer (aerial/street imagery)
- Filtering by state
- Clicking to select/lock an LGA
- Mobile-specific touch handling beyond Leaflet defaults
