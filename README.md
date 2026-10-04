# Maui at One Metre

A 1 m bare-earth terrain model of Maui, published as an interactive 3-D map:
**https://williamjudge808.github.io/east-maui-dtm/**

Two very different things are shown as one seamless surface, which is the
point and also the hazard:

* **Surveyed** — NOAA 2022 airborne lidar over West Maui, the isthmus, the
  upper mountain and Kahoʻolawe. Measured ground.
* **Modelled** — East Maui, the wettest and steepest quarter of the island,
  which has **no lidar at all**. A six-model neural ensemble reads Pictometry,
  Maxar Vivid and NAIP imagery over the Copernicus GLO-30 global DEM and
  outputs a 1 m residual correction. It is **not** a survey and must not be
  used as one.
* **Gap-filled** — 14.3 km² that neither covers. The NOAA survey has a real
  hole in it on Haleakalā's south flank, west of where the model's footprint
  begins; left alone it renders as a sea-level crater in the side of a
  3,000 m mountain. It is filled with Copernicus, levelled to the surrounding
  ground (+3.68 m). The grain visibly changes there: that patch is 30 m data.

The dashed teal outline on the map is the edge of the lidar survey — including
around that hole — and the cursor readout says which side of it you are on.

## Versions

**v3 (October 2026), this map.** Six models: v2's four plus two new ones that are told where the
coastline and the absolute elevation are, and that are trained to put the sea at sea level. On
East Maui, v3 is checked against ICESat-2 under a rule written down before the check ran; see Accuracy.

**v2 (October 2026), at [/v2/](v2/).** Retrained on Princeton's A100 cluster:
- **Ensemble:** four models (two SegFormer mit_b2, two mit_b5), trained 3× longer.
- **Grid fix:** each tile is predicted at 16 sub-pixel shifts and averaged. This removes a 4 m / 2 m grid the network
  drew into the September map.
- **Coastline:** the Maui County 2017 Detailed Coastline, plus 2013 USACE coastal lidar. It replaces Copernicus's coast,
  which is tens of metres off in places and put false land in coves (e.g. Huelo).

**v1 (September 2026), at [/v1/](v1/).** Also tagged `v1-september` in this repository.

## What you can do with it

| Control | |
|---|---|
| **Surface** | Flip between the 1 m surface and the Copernicus 30 m prior it was built from. `Space` toggles. |
| **View** | Shaded relief, hypsometric elevation tint, or contours. |
| **Relief** | Vertical exaggeration, 1–3×. |
| **3-D terrain** | Right-drag to tilt. The readout gives elevation under the cursor. |

The prior toggle is the honest comparison: it is the *raw* GLO-30, with no
datum shift applied, because that is exactly the surface the model was fed.

Contours are generated in the browser from the elevation tiles already loaded
(via [maplibre-contour](https://github.com/onthegomap/maplibre-contour)), so
switching to that view hosts and downloads no extra data.

## Accuracy

Errors are against airborne lidar unless noted.

| Measure | Copernicus prior | v1 (Sept) | v2 | **v3 (this map)** |
|---|---:|---:|---:|---:|
| Median absolute error (held-out chips) | 1.27 m | 0.55 m | 0.47 m | **0.46 m** |
| RMSE | 6.40 m | 4.87 m | not re-measured | not re-measured |
| NMAD | 1.62 m | 1.07 m | not re-measured | not re-measured |
| Spectral effective resolution (wet forest, paired) | 63.4 m | 27.5 m | not re-measured | not re-measured |

**On East Maui itself**, where no lidar exists, every version is checked against
ICESat-2 satellite laser altimetry. The setup:
- the same 6,732 ground segments from 35 overpasses (2018–2026), and the same
  pre-registered protocol as the September check;
- heights debiased per overpass, with a bootstrap clustered by overpass.

| | Copernicus | v1 | v2 | **v3** |
|---|---:|---:|---:|---:|
| Median absolute error | 1.50 m | 1.09 m | 1.08 m | **1.08 m** |
| 90th-percentile error | 7.48 m | 4.01 m | 3.73 m | **3.68 m** |

v3 is better than Copernicus by **+0.40 m** per segment [0.26, 0.64], and by
+0.67 m on the steepest quarter of the ground. Against v1 the typical point is
unchanged, but the large errors and steep ground improve: v2 vs v1 is +0.05 m
[0.03, 0.11] on steep ground.

**Against other bare-earth DEMs, on the same lidar.** FABDEM and GEDTM30 are global products that also
correct Copernicus toward bare earth. They were scored on the same 1,107 held-out chips, against NOAA
lidar published after both products. The rule was written down before scoring.

| | Copernicus | FABDEM | GEDTM30 | **this model** |
|---|---:|---:|---:|---:|
| Median per-chip error, 1 m | 1.27 m | 1.25 m | 2.22 m | **0.46 m** |
| Median per-chip error, averaged to 30 m | 1.15 m | 1.13 m | 1.98 m | **0.35 m** |

The model beats FABDEM on 99% of chips, by +1.09 m [1.02, 1.17] per chip. On a rainforest region a
version of the model never trained on (wet east Molokai), it scores 2.57 m against FABDEM's 4.85 m.
FathomDEM was not tested.

**At the seam.** Lidar and prediction overlap on 7.3% of the tiled grid, and
lidar always wins there. Across those 88.9 M pixels v3 reads **+0.399 m high**
of the lidar, mean absolute difference **1.515 m** (v2: +0.339 m, 1.513 m;
v1: +0.660 m, 1.843 m).
Much of that overlap is ground the models were trained on, so read it as a
consistency check, not a held-out score.

### Known limitation

Under closed canopy the v1 model retained a systematic **+0.93 m high bias** (the
raw prior's canopy ride-up is +4.90 m, so roughly 80% is removed, not all of
it); not yet re-measured for v2. Treat texture finer than ~25 m wavelength as
plausible rather than measured.

## Method notes

- Prediction is a residual on the global DEM, so the model corrects rather
  than invents the broad landform.
- Feeding the model *mismatched* imagery scores **worse than using no imagery
  at all**, which rules out "it just sharpens the prior."
- Imagery information is large-scale: scrambling imagery in 256-px blocks
  still destroys ~74% of the benefit, so the signal is geometric arrangement
  rather than local texture.

## The tile archives

| File | Zooms | Size | Contents |
|---|---|---:|---|
| `maui_surface_z15.pmtiles.png` | z8–15 | 102 MB | lidar + model, merged |
| `maui_prior_z12.pmtiles.png` | z8–12 | 10 MB | Copernicus GLO-30, raw |
| `lidar_coverage.geojson` | — | 310 kB | outline of what was surveyed, clipped to the coastline |

The coverage outline is traced from the mosaic's **valid pixels**, not from
the survey's tile index. NOAA ships tiles that are wholly or mostly nodata
inside a delivery block, so "the tile exists" is not "the ground was flown" —
and along the south shore the survey is a topobathy ribbon only a few hundred
metres wide, which a coarse simplification tolerance bulldozed inland. Tested
on a 0.01° grid across East Maui against measured pixel coverage: the outline
now over-claims 7 cells of 514 (all straddling the 50%-coverage threshold)
and under-claims 3, down from 56 over-claims. Since v2 it is also clipped to
the Maui County coastline, because 100 m cells drew whole coves inside the
outline although the survey has no returns from the water.

Both archives are Mapbox terrain-RGB (`rio rgbify -b -10000 -i 0.1`) on a
2.1447 m Web Mercator grid. The surface tiles are **lossless WebP**, not PNG:
identical pixels, 112 MB → 70 MB at z14.

**The pyramid is deliberately lopsided.** z15 (2.24 m/px — essentially the
native grid) exists *only* over the model's footprint, and z14 was dropped
*outside* it. The reconstruction is what this map exists to show, so that is
where the bytes go; the lidar tops out a level earlier. PMTiles is sparse and
MapLibre falls back to the parent tile, so West Maui simply gets less detail
at high zoom rather than holes — verified at screen z14, 0% blank frame.

That trade is what keeps the archive at 97.6 MiB (v3), under GitHub's 100 MiB
per-file cap, with z15 included at all.

The source is declared `tileSize: 256` although the tiles really are 512 px.
That only changes which zoom MapLibre reaches for at a given camera — one
level deeper, so the hillshade is computed from twice as fine a DEM. Measured:
**1.53× the high-frequency detail** for 2.1× the bytes, elevations unchanged.
The opening view costs about 3.8 MB over 51 range requests. Only one of the
two archives is ever rendered, so the surface/prior toggle does not double
tile traffic.

## Page security

The page is static, sets no cookies, stores nothing, and has no analytics —
it makes no third-party request except for its four library files. Those are
pinned by **Subresource Integrity** hash, so a compromised CDN cannot swap the
code, and a **Content-Security-Policy** names `cdn.jsdelivr.net` as the only
permitted script origin and restricts `connect-src` to this origin alone.

Ocean is encoded as elevation 0, never as nodata. rio-rgbify casts NaN
straight to `uint8`, which decodes to −10000 m; the previous version of this
map had corner tiles that were 33% −10000 m, tearing 10 km-deep holes in the
mesh at the edge of the footprint.

### Why the archives end in `.png`

They are ordinary PMTiles archives, not images. GitHub Pages gzip-encodes
`application/octet-stream` and then serves HTTP `Range` over the *compressed*
representation, so every PMTiles byte offset lands in the wrong place and
mid-file reads fail to decode. Files served as `image/*` are passed through
uncompressed, so ranges address real bytes. Rename to `.pmtiles` for any other
host.

## Credits and required notices

Lidar: NOAA NOS 2022 Kahoʻolawe/Lānaʻi/Maui/Molokaʻi/Oʻahu topobathymetric DEM
(US federal, public domain). Coastline: County of Maui Detailed Coastline
(2017, County Enterprise GIS; provided without warranty) and the 2013 USACE
NCMP topobathy lidar (public domain). Imagery used to *train* the model, none of it
redistributed here: Maxar Vivid 2022 via the State of Hawaiʻi Statewide GIS
Program, USDA NAIP 2021 via NOAA Digital Coast, Pictometry 2023 via Maui
County. Validation: NASA/NSIDC ICESat-2 ATL03/ATL08 via
[SlideRule](https://slideruleearth.io).

The Copernicus DEM notices below are **required verbatim** by Article 6 of the
[Copernicus DEM licence](https://doi.org/10.5270/ESA-c5d3d65), not offered as
a courtesy credit. The 1 m surface is an adapted product; the prior layer is
the data itself.

> produced using Copernicus WorldDEM-30 © DLR e.V. 2010-2014 and © Airbus
> Defence and Space GmbH 2014-2018 provided under COPERNICUS by the European
> Union and ESA; all rights reserved

> © DLR e.V. 2010-2014 and © Airbus Defence and Space GmbH 2014-2018 provided
> under COPERNICUS by the European Union and ESA; all rights reserved

Article 6(c) also requires that anyone receiving this data understands that
neither the licensor nor any other party warrants it. Nothing here is
warranted, and the East Maui surface in particular is a model's estimate, not
a survey — see the accuracy and limitation sections above.
