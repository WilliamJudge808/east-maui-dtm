# East Maui at One Metre

A 1 m bare-earth terrain model (DTM) of **East Maui**, the wettest and steepest
quarter of the island — and the only part of Maui County with **no lidar coverage**.

The surface is *predicted*, not measured: a two-model neural ensemble reads Maxar
Vivid and NAIP aerial imagery on top of the Copernicus GLO-30 global DEM and
outputs a 1 m residual correction. It is **not lidar** and should not be used as a
substitute for surveyed elevations.

| | |
|---|---|
| Grid | 38,376 × 49,410 px at 1 m (EPSG:6634) |
| Elevation range | −26 → 3,054 m |
| Coverage | 100% of the 9,546-tile region |
| Tiles | z8–17 terrain-RGB PMTiles, 1.40 GB |

## Interactive map

`index.html` is a self-contained MapLibre viewer (3-D terrain, tilt, exaggeration
slider). It reads the PMTiles archive over HTTP range requests — set `PMTILES_URL`
at the top of the file to wherever the archive is hosted.

## Accuracy

Validated three ways. Errors are against airborne lidar unless noted.

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

### Known limitation

Under closed canopy the model retains a systematic **+0.93 m high bias** (the raw
prior's canopy ride-up is +4.90 m, so roughly 80% is removed, not all of it).
Treat texture finer than ~25 m wavelength as plausible rather than measured.

## Method notes

- Prediction is a residual on the global DEM, so the model corrects rather than
  invents the broad landform.
- Ablation controls: feeding the model *mismatched* imagery scores **worse than
  using no imagery at all**, which rules out "it just sharpens the prior."
- Imagery information is large-scale: scrambling imagery in 256-px blocks still
  destroys ~74% of the benefit, so the signal is geometric arrangement rather
  than local texture.

Full experimental record, pre-registrations, and literature review live in the
project's research notes.

## Credits

Prior: [Copernicus GLO-30](https://doi.org/10.5270/ESA-c5d3d65)
(© DLR/ESA). Imagery: Maxar Vivid, USDA NAIP. Validation: NASA/NSIDC
ICESat-2 ATL03/ATL08 via [SlideRule](https://slideruleearth.io).

## Why the tile archive ends in `.png`

`maui_dtm_z14.pmtiles.png` is an ordinary PMTiles archive, not an image. GitHub
Pages gzip-encodes `application/octet-stream` and then serves HTTP `Range`
requests over the *compressed* representation, so every PMTiles byte offset
lands in the wrong place and mid-file reads fail to decode. Files served as
`image/*` are passed through uncompressed, so ranges address real bytes. Rename
it back to `.pmtiles` for any other host.
