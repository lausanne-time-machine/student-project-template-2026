---
title: Historical Maps (SwissTopo)
toc: false
---

# Historical Maps — SwissTopo "Voyage dans le temps"

SwissTopo publishes the **"Voyage dans le temps - Cartes"** layer (`ch.swisstopo.zeitreihen`) via a public WMTS service. It covers the entire country from the first Dufour map (1844) to 2021 — around 178 yearly snapshots of the Swiss national topographic map.

---

## Browse the historical layers

Use the slider to travel through time. Note that not every year has coverage for every area: the tile will simply be transparent if no map sheet exists for that year and location.

```js
const year = view(Inputs.range([1844, 2021], {step: 1, value: 1952, label: "Year"}));
```

```js
// Build the WMTS tile URL for a given year (timestamp = YYYYMMDD, always Dec 31)
function swisstopUrl(year) {
  return `https://wmts.geo.admin.ch/1.0.0/ch.swisstopo.zeitreihen/default/${year}1231/3857/{z}/{x}/{y}.png`;
}

// Create map with OSM base layer — runs ONCE (no dependency on `year`)
const mapDiv = display(document.createElement("div"));
mapDiv.style = "height: 560px; margin: 1em 0;";

const historicalMap = L.map(mapDiv).setView([46.8, 8.23], 8);

L.tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
  attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>',
  opacity: 0.3
}).addTo(historicalMap);

invalidation.then(() => historicalMap.remove());
```

```js
// Reactively swap the historical tile layer whenever `year` changes
{
  const tileLayer = L.tileLayer(swisstopUrl(year), {
    tms: false,
    opacity: 0.9,
    attribution: '&copy; <a href="https://www.swisstopo.admin.ch">swisstopo</a>'
  }).addTo(historicalMap);

  invalidation.then(() => historicalMap.removeLayer(tileLayer));
}
```

<div class="note">

The two-cell pattern is essential here: the first cell creates the map **once** (it has no dependency on `year`), while the second cell re-runs every time `year` changes and swaps the tile layer via `invalidation`.

</div>

---

## About the layer

The WMTS service is freely accessible without authentication. The tile URL template for EPSG:3857 (web Mercator, used by Leaflet) is:

```
https://wmts.geo.admin.ch/1.0.0/ch.swisstopo.zeitreihen/default/{YYYYMMDD}/3857/{z}/{x}/{y}.png
```

The time dimension always uses **December 31st** of the target year (`18441231` through `20211231`). The full list of 178 available timestamps is declared in the service's `WMTSCapabilities.xml` under the `<Dimension>` element for this layer.

| Parameter | Value |
|-----------|--------|
| Layer | `ch.swisstopo.zeitreihen` |
| Style | *(default)* |
| TileMatrixSet | `3857_18` (zoom 0–18) |
| Format | `image/png` |
| Period | 1844–2021 |
| License | [OGD – Open Government Data](https://www.swisstopo.admin.ch/fr/conditions-dutilisation-geodonnees-swisstopo) |

</div>
