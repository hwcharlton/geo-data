# geo-data — Tokyo 23-ward area packs (TopoJSON, ODbL)

Free, public **[ODbL-1.0](./LICENSE)** geographic **area packs** in
[TopoJSON](https://github.com/topojson/topojson) format for **Tokyo's 23
special wards** (東京都区部). Derived from **OpenStreetMap** via the
`@hwcharlton/geo-*` pipeline.

> **Attribution:** **© OpenStreetMap contributors** — data licensed under the
> **Open Database License (ODbL) v1.0**.

These packs are small, pre-simplified vector boundaries ready to drop straight
into a web map, a D3 visualisation, or any TopoJSON-aware renderer. They are
served for free over a CDN — see [Usage](#usage).

---

## Usage (jsDelivr CDN)

Files are served directly from this GitHub repo via
[jsDelivr](https://www.jsdelivr.com/)'s `/gh` endpoint. CORS is open
(`access-control-allow-origin: *`) and HTTP Range requests are supported.

**URL pattern** (always pin a release tag — never use `@latest` in production):

```
https://cdn.jsdelivr.net/gh/hwcharlton/geo-data@v1.0.0/packs/<ward>/<layer>/<tier>.topo.json
```

Each pack has a sibling **manifest** with full provenance:

```
https://cdn.jsdelivr.net/gh/hwcharlton/geo-data@v1.0.0/packs/<ward>/<layer>/<tier>.manifest.json
```

> ⚠️ **Pin the `@tag`.** Use `@v1.0.0` (or a later released tag), **not**
> `@latest` and **not** a bare branch ref. Pinning gives you immutable,
> permanently-cached, predictable bytes; `@latest` can change under you and is
> rate-limit-prone.

### Example

```js
// shibuya ward boundary, high detail
const url =
  "https://cdn.jsdelivr.net/gh/hwcharlton/geo-data@v1.0.0/packs/shibuya/admin/high.topo.json";

const topology = await fetch(url).then((r) => r.json());
// topology.type === "Topology"
// topology.objects["shibuya-admin"]  ← the GeometryCollection for this pack
```

A top-level [`index.json`](./index.json) catalogs every available
`{ward → layer → tiers}` for discovery.

---

## Catalog

- **Wards (24):** the 23 Tokyo special wards plus `tokyo-23` (all wards merged
  into a single boundary set).

  ```
  adachi    arakawa    bunkyo    chiyoda    chuo       edogawa
  itabashi  katsushika kita      koto       meguro     minato
  nakano    nerima     ota       setagaya   shibuya    shinagawa
  shinjuku  suginami   sumida    taito      toshima    tokyo-23
  ```

- **Layers:**
  - `admin` — administrative boundaries (ward outline)
  - `road` — road network
  - `water` — waterways / water bodies

- **Tiers** (level of simplification — pick the smallest that looks good at your
  zoom):
  - `high` — most detail, largest file
  - `med` — medium
  - `low` — least detail, smallest file

**Coverage notes**

- The 23 individual wards each provide all three layers (`admin`, `road`,
  `water`) at all three tiers (`high`, `med`, `low`).
- `tokyo-23` (the merged 23-ward set) currently provides the `admin` layer only,
  at all three tiers.

The authoritative, machine-readable catalog is [`index.json`](./index.json).

---

## PLATEAU authoritative building tier (3D) — `packs/plateau/`

Since **v1.2.0** this repo also ships an **authoritative building tier** derived
from **[Project PLATEAU](https://www.mlit.go.jp/plateau/)** (国土交通省 / MLIT),
the Japanese government's open 3D city model: **1,410,692 buildings** across the
23 special wards, each with its **real surveyed footprint + height**
(`measuredHeight`, tallest ≈ 332.9 m) — not the OSM tier's sparse/estimated
heights.

> **Attribution (this tier):** **出典：国土交通省 PLATEAU（加工して作成）**, licensed
> under **ODbL-1.0**. This is a **separate source** from the OSM area packs; when
> you render the PLATEAU buildings *alongside* the OSM tier, credit **both**
> (`出典：国土交通省 PLATEAU（加工して作成）　© OpenStreetMap contributors`).

PLATEAU is taken under its officially-selectable **ODbL** option, so it
co-mingles with the OSM ODbL tier and is freely redistributable here.

### Structure — per-mesh packs (viewport-culled rendering)

The whole 23-ku (~1.4M buildings) would blow past any WebGL renderer's per-frame
polygon budget, so the tier is **split into 572 packs, one per Japanese
3rd-level mesh (~1 km, 8-digit code)** — the unit a viewport culler loads and
drops as you pan/zoom. A top-level **mesh index** lists every mesh with its
bounding box and pack path:

```
https://cdn.jsdelivr.net/gh/hwcharlton/geo-data@v1.2.0/packs/plateau/index.json
```

```js
const base = "https://cdn.jsdelivr.net/gh/hwcharlton/geo-data@v1.2.0";
const index = await fetch(`${base}/packs/plateau/index.json`).then((r) => r.json());
// index.meshes: [{ mesh, bbox:[minLon,minLat,maxLon,maxLat], pack, count, height_max }]
// load only the meshes whose bbox intersects your viewport:
const entry = index.meshes[0];
const packUrl = `${base}/packs/${entry.pack}`; // plateau/<mesh>/building/flat.<hash>.topo.json.br
```

Each per-mesh pack is **brotli-only** (`flat.<hash>.topo.json.br`) — decode it
client-side (the platform `DecompressionStream("br")` or a wasm brotli decoder).
Every building Feature carries a numeric `height` (metres). The
`@hwcharlton/geo-client` + `@hwcharlton/geo-canvas` libraries provide a
ready-made fetch + worker-decode + viewport-cull + extrusion path.

---

## Provenance

- **Source data:** OpenStreetMap, via **Geofabrik** extract
  `asia/japan/kanto` (`kanto-latest.osm.pbf`).
- **Input snapshot date:** **2026-06-12**.
- **Toolchain:** baked with [`osmium`](https://osmcode.org/osmium-tool/),
  [`mapshaper`](https://github.com/mbloch/mapshaper),
  [`osmtogeojson`](https://github.com/tyrasd/osmtogeojson) and the
  `@hwcharlton/geo-*` pipeline (TopoJSON quantization + Visvalingam
  simplification).

Every pack ships a sibling `*.manifest.json` recording the exact source,
license, attribution, tool versions, simplification parameters and content
hash for that specific file. **The manifests are the full source of truth for
provenance** — see e.g.
[`packs/shibuya/admin/high.manifest.json`](./packs/shibuya/admin/high.manifest.json).

---

## Licensing

The data in this repository is licensed under the **Open Database License (ODbL)
v1.0** (full text in [`LICENSE`](./LICENSE)). It draws on **two sources**, each
credited where it appears:

- the **area packs** (`packs/<ward>/…`) and the OSM building tier are
  **© OpenStreetMap contributors**;
- the **PLATEAU building tier** (`packs/plateau/…`, since v1.2.0) is
  **出典：国土交通省 PLATEAU（加工して作成）** (国土交通省 / MLIT Project PLATEAU, taken
  under its ODbL option).

Both are ODbL-1.0; render a combined view and you must show **both** credits.

You are free to copy, distribute, use, and adapt this data, provided you (a)
**attribute** OpenStreetMap and (b) keep any *derivative database* you publicly
distribute under the **same ODbL** license. In short:

> **Attribution:** **© OpenStreetMap contributors**, ODbL-1.0.

### What ODbL share-alike actually requires here

The downloadable **raw vector packs in this repo are a Derivative Database** of
OpenStreetMap, and so they are themselves licensed under **ODbL** — this public,
openly-licensed repository is exactly what **discharges the share-alike
obligation** for that derivative database.

That distinction matters for *you*, the consumer:

- A **Produced Work** is anything you *make from* the data — a rendered/baked
  map image, a styled tile, a baked background, a chart, a screenshot. A
  Produced Work needs **attribution only** (`© OpenStreetMap contributors`); it
  does **not** trigger share-alike, so you are free to license your own
  rendered output however you like.
- A **Derivative Database** is the data itself, modified or built upon and then
  **publicly distributed as data**. If you redistribute these packs (or a
  transformed *database* derived from them) publicly, that redistribution stays
  under **ODbL** and must remain openly available.

So: **draw whatever you want and license the picture as you please — just keep
the attribution.** Only if you redistribute the underlying *data* does the
ODbL share-alike term follow it.

*(This corresponds to the upstream pipeline's HG-6 / ADR-006: raw vector packs
= Derivative Database → ODbL; rendered/baked output = Produced Work →
attribution only.)*

---

## Files

```
LICENSE                      Full ODbL-1.0 license text
README.md                    This file
index.json                   Machine-readable catalog of wards/layers/tiers (+ the plateau tier)
packs/<ward>/<layer>/<tier>.topo.json            TopoJSON area pack (OSM, ODbL)
packs/<ward>/<layer>/<tier>.manifest.json        Per-pack provenance manifest
packs/plateau/index.json                         PLATEAU mesh index (572 meshes)
packs/plateau/<mesh>/building/flat.<hash>.topo.json.br   PLATEAU building pack (brotli)
packs/plateau/<mesh>/building/flat.manifest.json         PLATEAU per-mesh provenance
```
