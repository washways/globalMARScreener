# Dataset Summary

The Global MARS Screener uses global geospatial inputs available in or compatible with Google Earth Engine, plus two WASHways-managed assets.

## Core inputs

| Dataset | Earth Engine ID or asset | Main role |
|---|---|---|
| GAUL level 0 | FAO/GAUL/2015/level0 | Country selector |
| GAUL level 1 | FAO/GAUL/2015/level1 | ADM1 selector |
| GAUL level 2 | FAO/GAUL/2015/level2 | ADM2 selector |
| FABDEM | projects/sat-io/open-datasets/FABDEM | Slope and terrain processing |
| MERIT Hydro | MERIT/Hydro/v1_0_1 | Elevation, HAND, upstream drainage area, hydrographic context |
| ESA WorldCover | ESA/WorldCover/v200 | Open land and built-up masking |
| ERGo landforms | CSP/ERGo/1_0/Global/SRTM_landforms | Valley, slope, and ridge setting |
| OpenLandMap field capacity | OpenLandMap/SOL/SOL_WATERCONTENT-33KPA_USDA-4B1C_M/v01 | Soil-water storage proxy |
| Depth to bedrock | projects/washways/assets/BDTICMM250m | Storage headroom / regolith proxy |
| Aridity index | projects/washways/assets/AridityIndexv31yrFixed | Broad climate modifier |

## Important caveats

- The two WASHways assets must be accessible to the Earth Engine account running the model.
- Several inputs are proxies, not direct measurements of infiltration, transmissivity, recharge volume, or water quality.
- Local interpretation remains essential because global data cannot fully capture aquifer complexity.

## Interpretation note

These datasets are suitable for first-pass screening and prioritisation. They are not a substitute for field hydrogeology, aquifer testing, groundwater monitoring, water-quality analysis, or engineering design.
