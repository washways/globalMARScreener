# Global Managed Aquifer Recharge Screener

## Quick links

- Live app: https://washways.projects.earthengine.app/view/globalmarscreener
- Wrapper landing page: https://washways.org/globalMARScreener/
- Repository: https://github.com/washways/globalMARScreener
- Full methodology: ./METHODOLOGY.md
- Architecture note: ./docs/architecture.md
- License: MIT, see ./LICENSE

## Overview

The Global Managed Aquifer Recharge Screener is a Google Earth Engine application for screening MAR potential across countries, ADM1 areas, and ADM2 areas. The current model is a scale-aware, hydrogeology-conditioned screening framework. It blends terrain, hydrography, land cover, soils, depth to bedrock, aridity, lithology, and hydrogeology into a three-frame suitability system.

The main operational output is now hybrid suitability. Hybrid suitability combines:

- absolute suitability, based on fixed physical logic intended to remain more comparable across larger or more heterogeneous regions;
- contextual suitability, based on within-AOI normalization and percentile interpretation to highlight the better relative areas inside the selected geography; and
- a direct hydrogeology modifier built from uploaded GLiM and GLHYMPS assets.

The current build depends on two uploaded hydrogeology layers:

- GLiM: `projects/washways/assets/glim_global_source`
- GLHYMPS: `projects/washways/assets/glhymps_global_source`

This means the tool no longer behaves as a mainly terrain-led contextual ranking app. It now integrates uploaded GLiM and GLHYMPS assets directly into the model and reduces contextual influence as AOI size increases.

The tool is for screening, not site confirmation. It does not replace field hydrogeology, aquifer testing, groundwater monitoring, engineering design, safeguards review, or local planning.

## Current outputs

The application exposes the following current layers. The default display is the hybrid suitability class.

### Primary screening outputs

- Hybrid suitability class, default
- Contextual suitability class
- Absolute suitability class
- Recommended MAR intervention setting
- Contextual opportunity class
- Absolute opportunity class
- Riverbank / floodplain suitability class
- Tributary / valley suitability class
- Footslope / colluvial suitability class
- Upland / distributed suitability class

### Continuous layers

- Hybrid suitability continuous, lightweight display
- Opportunity absolute continuous
- Opportunity contextual continuous
- Feasibility
- Constraint
- Confidence
- Hydrogeology modifier

### Hydrogeology debug layers

- Debug GLiM lithology score
- Debug GLHYMPS logK score
- Debug GLHYMPS porosity score
- Debug GLHYMPS GUM_K score
- Debug GLHYMPS description score

### Terrain and hydrology debug layers

- Debug MERIT UPA
- Debug MERIT very low HAND
- Debug MERIT low HAND
- Debug MERIT moderate HAND
- Debug MERIT terrain convergence
- Debug local slope transition
- Debug footslope index
- Debug OpenLandMap field capacity
- Debug MERIT drainage wetness

Only one model or diagnostic layer is shown at a time. The AOI boundary remains visible separately.

## Intended users

This README is written for GIS analysts, Earth Engine users, programmers, and technical planners. It is practical and implementation-oriented. The hydrogeological rationale is documented in ./METHODOLOGY.md.

## Data sources and required assets

The app combines global Earth Engine datasets with user-managed uploaded assets.

| Dataset | Earth Engine ID or asset | Role |
|---|---|---|
| GAUL level 0 | `FAO/GAUL/2015/level0` | Country selector |
| GAUL level 1 | `FAO/GAUL/2015/level1` | ADM1 selector |
| GAUL level 2 | `FAO/GAUL/2015/level2` | ADM2 selector |
| FABDEM | `projects/sat-io/open-datasets/FABDEM` | Slope derivation |
| MERIT Hydro | `MERIT/Hydro/v1_0_1` | HAND, UPA, elevation |
| ESA WorldCover | `ESA/WorldCover/v200` | Open land, built-up risk |
| ERGo SRTM landforms | `CSP/ERGo/1_0/Global/SRTM_landforms` | Valley, slope, ridge support |
| OpenLandMap field capacity | `OpenLandMap/SOL/SOL_WATERCONTENT-33KPA_USDA-4B1C_M/v01` | Soil-water storage proxy |
| Depth to bedrock | `projects/washways/assets/BDTICMM250m` | Storage headroom proxy |
| Aridity index | `projects/washways/assets/AridityIndexv31yrFixed` | Climate context |
| GLiM uploaded asset | `projects/washways/assets/glim_global_source` | Lithology feature collection |
| GLHYMPS uploaded asset | `projects/washways/assets/glhymps_global_source` | Hydrogeology feature collection |

The app expects the following uploaded user assets to exist and be accessible to the Earth Engine account running the script:

- `projects/washways/assets/glim_global_source`
- `projects/washways/assets/glhymps_global_source`

If those assets are missing or inaccessible, the hydrogeology-conditioned workflow is not fully operational.

## Model architecture

The Earth Engine script follows three scoring pathways and then exposes several derived products.

### Absolute pathway

The absolute pathway uses fixed physical suitability logic intended to remain more stable across scales. It combines hydrogeomorphic opportunity, feasibility, and constraint using non-contextual scores so that large or heterogeneous AOIs do not rely entirely on within-area ranking.

### Contextual pathway

The contextual pathway uses AOI-based local normalization and percentile interpretation to highlight the better relative areas inside the selected geography. This pathway is useful when the user wants to identify the locally strongest candidates within one district, province, or country selection.

### Hybrid pathway

The hybrid pathway blends absolute and contextual suitability and is the main screening output. Hybrid suitability is now the default class shown in the interface.

Around these pathways, the model also computes:

- four intervention-family opportunity surfaces;
- feasibility from storage and infiltration conditions;
- constraint from wetness, slope, built-up, ridge, and low-hydrogeology penalties;
- intervention-specific suitability layers;
- a recommended intervention setting based on the best-scoring intervention family; and
- confidence as a screening confidence indicator rather than a validation metric.

## Scale-aware logic

Administrative units vary massively in size across countries. A district, province, or region is not a hydrogeological unit. Pure contextual ranking can therefore become misleading over large heterogeneous areas, because the best percentile-ranked pixels inside a very large region are not necessarily good in absolute hydrogeological terms.

The current code addresses this by reducing contextual influence as AOI area increases. Contextual weight is determined from AOI area in square kilometres.

| AOI area | Contextual weight | Absolute weight |
|---|---:|---:|
| less than 2,500 km2 | 0.50 | 0.50 |
| 2,500 to less than 15,000 km2 | 0.40 | 0.60 |
| 15,000 to less than 75,000 km2 | 0.30 | 0.70 |
| 75,000 km2 and above | 0.20 | 0.80 |

In simple terms:

- small AOIs get more contextual influence;
- very large AOIs rely more on absolute suitability.

This is a pragmatic scaling rule intended to reduce overfitting of contextual ranking in large heterogeneous administrative regions. It is not claimed to be universally optimal.

## Hydrogeology integration

Hydrogeology conditioning is now part of the live model rather than a placeholder. The code uses two uploaded feature collections directly.

### GLiM contribution

The app reads lithology text from:

- `Litho`
- fallback `xx`

These text fields are classified heuristically into broad lithological favorability groups. The current logic distinguishes broadly favorable unconsolidated, alluvial, sandy, gravelly, volcanic, and some carbonate settings from more restrictive clay-rich, marl, shale, mudstone, or similar low-permeability settings.

### GLHYMPS contribution

The app reads the following GLHYMPS fields directly:

- `logK_Ferr_`
- `Porosity_x`
- `K_stdev_x1`
- `Transmissi`
- `GUM_K`
- `Descriptio`

The current implementation derives:

- a GLiM lithology score from text classification;
- a GLHYMPS description score from text classification;
- a permeability score from `logK_Ferr_`;
- a porosity score from `Porosity_x`;
- an uncertainty or stability modifier from `K_stdev_x1`;
- a transmissivity contextual score from `Transmissi`; and
- a class-based hydrogeology score from `GUM_K`.

These components are blended into a hydrogeology modifier, roughly constrained between 0.15 and 1.0. The modifier affects:

- storage;
- infiltration;
- feasibility;
- constraint;
- exclusions.

The modifier is physically motivated but still proxy-based. Text-based lithology interpretation from GLiM and GLHYMPS is heuristic and should be refined where better national hydrogeological data exist.

## Layer descriptions

### Suitability frames

| Layer | Interpretation |
|---|---|
| Hybrid suitability class | Main default screening output combining absolute and contextual suitability with scale-aware blending |
| Contextual suitability class | Best relative areas inside the selected AOI, with minimum absolute floors for higher classes |
| Absolute suitability class | Fixed-threshold physical suitability layer intended to be more comparable across AOIs |
| Hybrid suitability continuous, lightweight display | Byte-scaled display version of hybrid suitability for faster rendering |

### Opportunity, feasibility, and constraint

| Layer | Interpretation |
|---|---|
| Contextual opportunity class | Relative best opportunity areas within the AOI |
| Absolute opportunity class | Fixed-threshold opportunity class |
| Opportunity absolute continuous | Continuous absolute opportunity score |
| Opportunity contextual continuous | Continuous contextual opportunity score |
| Feasibility | Product of storage and infiltration logic, reduced by built-up risk |
| Constraint | Combined wetness, slope, built-up, ridge, and low-hydrogeology constraint logic |
| Confidence | Margin-based indicator of how clearly one intervention family dominates under current logic |
| Hydrogeology modifier | Combined geological and hydrogeological favorability proxy |

### Intervention-family outputs

| Layer | Interpretation |
|---|---|
| Recommended MAR intervention setting | Dominant intervention family at each pixel |
| Riverbank / floodplain suitability class | Suitability for drainage-adjacent and alluvial settings |
| Tributary / valley suitability class | Suitability for tributary and valley-margin runoff capture settings |
| Footslope / colluvial suitability class | Suitability for lower-slope and colluvial recharge settings |
| Upland / distributed suitability class | Suitability for distributed recharge-supporting measures |

### Diagnostic layers

The debug layers expose major components of the hydrogeology, terrain, soil, wetness, and hydrology logic so users can inspect why a location received a given score.

## UI behavior

The interface is organized around a left sidebar and a map panel.

- Users select country, ADM1, and ADM2 from GAUL boundaries.
- The model runs in national, ADM1, or ADM2 mode depending on the selection depth.
- Only one model or diagnostic layer is shown at a time through a controlled layer selector.
- The AOI boundary remains visible separately.
- The opacity slider adjusts the selected layer only.
- The status line reports AOI area, contextual weight, absolute weight, analysis scale, and render scale.

The app is designed for Earth Engine Code Editor use and Earth Engine App deployment.

## Limitations

Despite direct hydrogeology integration, the current model still does not directly include:

- measured groundwater levels;
- groundwater seasonal fluctuation;
- recharge water availability;
- water quality;
- abstraction pressure;
- legal or environmental constraints;
- national groundwater body maps;
- local field validation.

Additional limitations are important:

- GLiM and GLHYMPS text interpretation is heuristic.
- Lithological text matching is coarse and global.
- The hydrogeology modifier is a proxy, not a substitute for local aquifer maps.
- Administrative units are not hydrogeological units.
- The model should not be presented as calibrated or validated against known MAR performance unless that work has been done separately.

## Suggested next improvements

- Replace heuristic lithology parsing with nationally harmonized hydrogeology classes where available.
- Integrate groundwater body or aquifer map boundaries.
- Add recharge water availability and groundwater-need layers.
- Add groundwater stress or abstraction pressure terms.
- Calibrate against known MAR sites and national studies.
- Allow normalization by basin, groundwater body, or hydrogeological province rather than only by administrative unit.

## Interpretation note

This README describes the current operational model and should be treated as the authoritative high-level description of the live screening workflow. It is not a changelog. Older wording that framed the tool as mainly terrain-based, contextual-only, or hydrogeology-placeholder logic is no longer current.
