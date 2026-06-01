# LGA Map — Lessons Learnt

**Project:** Interactive Australian LGA Boundaries Map  
**Completed:** 2026-06-01  
**Stack:** Static HTML, Leaflet 1.9.4 (CDN), vanilla JS

---

## What Worked Well

**Single-file architecture** — Keeping everything in one `index.html` with no build step meant fast iteration and zero deployment friction. For a data exploration tool serving a small internal audience, this was the right call.

**Leaflet for polygon/choropleth maps** — Native GeoJSON support, simple event API (mouseover/mouseout/bindPopup), and a clean Leaflet control system for the legend made all five features straightforward to implement.

**State colour keyed by ABS state_code** — Using the ABS standard codes ('1'–'8') as dictionary keys worked cleanly. `String(feature.properties.state_code)` coercion guarded against numeric vs. string type inconsistency in GeoJSON exports.

**Fetch + remove loading indicator pattern** — Removing the `#loading` div inside the `.then()` block (rather than toggling visibility) kept the DOM clean and avoided any CSS complexity.

---

## Issues Found and Fixed

| Issue | Root Cause | Fix |
|-------|-----------|-----|
| `defer` on Leaflet CDN broke map init | Inline scripts execute during parsing, before deferred scripts run — `L` was undefined | Removed `defer`; Leaflet must load synchronously when using inline scripts |
| `#loading` positioning broken | `position: absolute` with no positioned ancestor | Added `position: relative` to `body` |
| Missing accessibility on search | No `aria-label` on input; emoji not hidden from screen readers | Added `aria-label="Search LGA name"` and `aria-hidden="true"` on emoji spans |
| Search stuttered on fast typing | No debounce — `fitBounds` fired on every keystroke across 548 layers | Added 250ms `setTimeout` debounce |
| Popup clipped at map edges | `autoPan: false` prevented Leaflet from panning to show full popup | Changed to `autoPan: true` |

---

## Known Limitations

- **Search popup vs. hover popup conflict** — A popup opened by search will close when the user moves the mouse over and off that polygon (mouseout fires `closePopup`). A future "pinned" popup state would fix this.
- **Search returns only the first match** — No feedback when no LGA matches the query. A status message would improve UX.
- **No base tile layer** — Map shows polygons on a plain dark background. Good for clean data focus; not suitable if geographic context (roads, coastline, place names) is needed.
- **`file://` protocol blocked** — `fetch()` requires a web server. Use `python -m http.server 8080` or VS Code Live Server.

---

## Future Features

### 1. Selectable data in hover popup
Show additional demographic indicators on hover, toggleable by the user:
- Population (total, density)
- Median age
- Median household income
- Socio-economic score (SEIFA / IRSD)

**Approach:** Join attribute data to the GeoJSON layer by `lga_code` at load time. Add a toggle control (radio buttons or a dropdown) in the top bar to switch the active metric. Update popup content and optionally drive polygon fill opacity/shade from the metric value (choropleth).

### 2. Slide-out quick stats dashboard
A panel that slides in from the right when an LGA is clicked, showing a richer set of demographic and economic indicators:
- Population breakdown (age groups, gender)
- Household and dwelling statistics
- Employment and income
- Historical trend sparklines

**Approach:** Click handler on GeoJSON layer opens a `<aside>` panel with CSS transition. Data loaded from a pre-built JSON lookup keyed by `lga_code`.

### 3. LGA benchmarking tool
Compare the selected LGA against similar LGAs across a chosen metric — e.g. "show me all LGAs with similar population size" or "compare this LGA's median income against state peers".

**Approach:** Sidebar or modal showing a ranked table of comparable LGAs. Highlight comparison LGAs on the map simultaneously. Similarity could be defined by population quintile, urbanity classification, or state.

---

## Technical Notes for Future Sessions

- GeoJSON source: `C:\Users\michaelz\source\repos\claude\demographic-indicators\lga_boundaries.geojson` (4.7 MB, gitignored)
- Serve with: `python -m http.server 8080` from project root
- State code lookup: ABS codes 1=NSW, 2=VIC, 3=QLD, 4=SA, 5=WA, 6=TAS, 7=NT, 8=ACT
- All JS is inline in `index.html` — no modules, no bundler
- `geoJsonLayer` is module-scope so all functions can reference it
