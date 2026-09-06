# Maui at One Metre

A 1 m bare-earth terrain model of Maui, published as an interactive 3-D map:
**https://williamjudge808.github.io/east-maui-dtm/**

Two very different things are shown as one seamless surface, which is the
point and also the hazard:

* **Surveyed** — NOAA 2022 airborne lidar over West Maui, the isthmus, the
  upper mountain and Kahoʻolawe. Measured ground.
* **Modelled** — East Maui, the wettest and steepest quarter of the island,
  which has **no lidar at all**. A two-model neural ensemble reads Maxar Vivid
  and NAIP imagery over the Copernicus GLO-30 global DEM and outputs a 1 m
  residual correction. It is **not** a survey and must not be used as one.
* **Gap-filled** — 14.3 km² that neither covers. The NOAA survey has a real
  hole in it on Haleakalā's south flank, west of where the model's footprint
  begins; left alone it renders as a sea-level crater in the side of a
  3,000 m mountain. It is filled with Copernicus, levelled to the surrounding
  ground (+3.68 m). The grain visibly changes there: that patch is 30 m data.

The dashed teal outline on the map is the edge of the lidar survey — including
around that hole — and the cursor readout says which side of it you are on.

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

| Measure | Copernicus prior | This model |
|---|---:|---:|
| Median absolute error (held-out chips) | 1.27 m | **0.55 m** |
| RMSE | 6.40 m | **4.87 m** |
| NMAD | 1.62 m | **1.07 m** |
| Spectral effective resolution (wet forest, paired) | 63.4 m | **27.5 m** |

**On East Maui itself**, where no lidar exists, the model was checked against
ICESat-2 satellite laser altimetry (6,901 ATL08 ground-classified segments,
35 overpasses, 2018–2026): median |error| **1.10 m** vs the prior's 1.52 m, a
paired improvement of **+0.40 m** [0.27, 0.64], concentrated on steep ground.

**At the seam.** Lidar and prediction overlap on 7.3% of the tiled grid, and
lidar always wins there. Across those 88.9 M pixels the model reads **+0.660 m
high** of the lidar, mean absolute difference 1.843 m — so the join is a
sub-metre step, invisible at map scale but real.

### Known limitation

Under closed canopy the model retains a systematic **+0.93 m high bias** (the
raw prior's canopy ride-up is +4.90 m, so roughly 80% is removed, not all of
it). Treat texture finer than ~25 m wavelength as plausible rather than
measured.

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
| `maui_surface_z14.pmtiles.png` | z8–14 | 69 MB | lidar + model, merged |
| `maui_prior_z12.pmtiles.png` | z8–12 | 10 MB | Copernicus GLO-30, raw |
| `lidar_coverage.geojson` | — | 4 kB | outline of what was surveyed |

Both archives are Mapbox terrain-RGB (`rio rgbify -b -10000 -i 0.1`) on a
2.1447 m Web Mercator grid — 2× oversampled for z14's 4.47 m/px.

The surface tiles are **lossless WebP**, not PNG: identical pixels, 112 MB →
69 MB, which is what puts a whole-island archive under GitHub's 100 MiB
per-file limit. The opening view costs about 1.6 MB over 34 range requests;
only one of the two archives is ever rendered at a time, so the toggle does
not double tile traffic.

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

## Credits

Lidar: NOAA NOS 2022 Kahoʻolawe/Lānaʻi/Maui/Molokaʻi/Oʻahu topobathymetric DEM.
Prior: [Copernicus GLO-30](https://doi.org/10.5270/ESA-c5d3d65) (© DLR/ESA).
Imagery: Maxar Vivid, USDA NAIP. Validation: NASA/NSIDC ICESat-2 ATL03/ATL08
via [SlideRule](https://slideruleearth.io).
