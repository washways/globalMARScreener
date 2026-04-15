# Global MARS Screener

## Quick links

- Live app: https://washways.projects.earthengine.app/view/globalmarscreener
- Wrapper landing page: https://washways.org/globalMARScreener/
- Repository: https://github.com/washways/globalMARScreener
- Full methodology: ./METHODOLOGY.md

## 1. Overview

The Global MARS Screener is a Google Earth Engine application for screening managed aquifer recharge potential across countries, ADM1 areas, and ADM2 areas. It combines global terrain, hydrography, land-cover, soil, aridity, and depth-to-bedrock datasets to generate relative suitability layers and broad recommended MAR intervention settings.

The tool is designed as a screening application. It is not a final site-selection model and does not replace field investigation, aquifer testing, geophysics, groundwater monitoring, water-quality assessment, engineering design, safeguards review, or cost analysis.

## 2. Main outputs

The application generates the following layers:

- Combined suitability class.
- Recommended MAR intervention setting.
- Combined opportunity class.
- Combined suitability continuous score.
- Opportunity.
- Feasibility.
- Constraint.
- Confidence.
- Diagnostic layers.

Only one model or diagnostic layer is visible at a time. The administrative boundary remains visible separately.

## 3. Intended users

The README is written for GIS analysts, programmers, technical hydrogeologists, Earth Engine users, and application maintainers. The companion methodology file provides the hydrogeological and academic justification for the model.

## 4. Application architecture

The script is a single Earth Engine JavaScript file. It contains:

1. Configuration.
2. Dataset loading.
3. UI style helpers.
4. UI construction.
5. Layer control and legend logic.
6. General helper functions.
7. Administrative selector functions.
8. Input layer construction.
9. MAR model construction.
10. Main update function.
11. UI event handlers.
12. Startup routine.

The code is designed for Earth Engine Code Editor use and can be adapted for Earth Engine Apps.

## 5. Required datasets

The tool uses the following datasets:

| Dataset | Earth Engine ID or asset |
|---|---|
| GAUL level 0 | `FAO/GAUL/2015/level0` |
| GAUL level 1 | `FAO/GAUL/2015/level1` |
| GAUL level 2 | `FAO/GAUL/2015/level2` |
| FABDEM | `projects/sat-io/open-datasets/FABDEM` |
| MERIT Hydro | `MERIT/Hydro/v1_0_1` |
| ESA WorldCover | `ESA/WorldCover/v200` |
| ERGo SRTM landforms | `CSP/ERGo/1_0/Global/SRTM_landforms` |
| OpenLandMap field capacity | `OpenLandMap/SOL/SOL_WATERCONTENT-33KPA_USDA-4B1C_M/v01` |
| Depth to bedrock | `projects/washways/assets/BDTICMM250m` |
| Aridity index | `projects/washways/assets/AridityIndexv31yrFixed` |

The two user assets must be accessible to the Earth Engine account running the script.

## 6. Main configuration values

The script defines analysis and rendering scales:

```javascript
var SCALE_NATIONAL = 500;
var SCALE_ADM1 = 250;
var SCALE_ADM2 = 120;

var RENDER_SCALE_NATIONAL = 2000;
var RENDER_SCALE_ADM1 = 500;
var RENDER_SCALE_ADM2 = 150;
```

The default country is:

```javascript
var DEFAULT_COUNTRY = 'Malawi';
```

The model uses two user-provided assets:

```javascript
var DTB_ASSET = 'projects/washways/assets/BDTICMM250m';
var AI_ASSET = 'projects/washways/assets/AridityIndexv31yrFixed';
```

## 7. Weighting configuration

The default type-level weights are:

```javascript
var settings = {
  riverWeight: 18,
  valleyWeight: 34,
  footWeight: 30,
  uplandWeight: 18,
  weightedMeanShare: 0.50,
  localBestShare: 0.50,
  constraintPenaltyStrength: 0.96,
  constraintPenaltyFloor: 0.03,
  oppClassT1: 0.18,
  oppClassT2: 0.40,
  oppClassT3: 0.65,
  suitClassT1: 0.22,
  suitClassT2: 0.45,
  suitClassT3: 0.72
};
```

These weights were proposed by AI as transparent expert-judgement priors. They are not empirically calibrated coefficients. They reflect hydrogeological process reasoning, expected predictor relevance, and global dataset reliability. They should be recalibrated with local hydrogeological evidence when available.

The current settings favour tributary/valley and footslope/colluvial settings because these are often actionable for decentralised MAR screening in semi-arid, basement-complex, rural, and data-limited contexts. This is a starting hypothesis, not a universal rule.

## 8. User interface

The UI is built with Earth Engine `ui` components.

The app has:

- A fixed left sidebar.
- A map panel.
- Area selectors.
- Run and refresh buttons.
- One-layer selector.
- Opacity slider.
- Dynamic legend.
- Status text.

The native Earth Engine layer list is disabled:

```javascript
mapPanel.setControlVisibility({
  all: true,
  layerList: false,
  mapTypeControl: true,
  zoomControl: true,
  scaleControl: true
});
```

This avoids confusion because the script implements its own one-layer selector.

## 9. Styling

Earth Engine UI styling is limited compared with HTML/CSS. The code improves readability using:

- Cleaner font stack.
- Stronger spacing.
- Lighter sidebar background.
- White cards for layer and legend sections.
- Black button text.
- Clear section headings.
- Simple colour palettes.

True CSS border radius for rounded buttons is not supported in the standard Earth Engine UI API.

## 10. Area selection logic

The tool uses GAUL boundaries. Users select:

1. Country.
2. ADM1.
3. ADM2.

Special selector values are used:

```javascript
var ALL_ADM1 = '__ALL_ADM1__';
var ALL_ADM2 = '__ALL_ADM2__';
```

The model mode is determined as follows:

- Country only: national mode.
- Country + ADM1: ADM1 mode.
- Country + ADM1 + ADM2: ADM2 mode.

## 11. Scale logic

The function `getScaleForMode()` returns the analysis scale:

```javascript
function getScaleForMode(mode) {
  if (mode === 'National') return SCALE_NATIONAL;
  if (mode === 'ADM1') return SCALE_ADM1;
  return SCALE_ADM2;
}
```

The function `getRenderScale()` returns the display scale. This keeps national rendering lighter while retaining more detail for ADM2 analysis.

## 12. Projection functions

The code uses two projection helper functions.

### 12.1 FABDEM projection helper

```javascript
function toProj(img, scaleUse, aoi) {
  return ee.Image(img)
    .setDefaultProjection(DEM.projection())
    .reproject({crs: DEM.projection(), scale: scaleUse})
    .clip(aoi);
}
```

This is used mainly for FABDEM-derived slope processing.

### 12.2 MERIT projection helper

```javascript
function toMeritProj(img, scaleUse, aoi) {
  return ee.Image(img)
    .setDefaultProjection(MERIT_ELV.projection())
    .reproject({crs: MERIT_ELV.projection(), scale: scaleUse})
    .clip(aoi);
}
```

This is used for MERIT elevation, UPA, HAND, and most final rendered layers.

## 13. Local normalisation

The function `normalizeByArea()` normalises an image using the 2nd and 98th percentiles within the selected AOI.

```javascript
function normalizeByArea(img, bandName, aoi, scaleUse) {
  var stats = ee.Dictionary(ee.Image(img).rename(bandName).reduceRegion({
    reducer: ee.Reducer.percentile([2, 98]),
    geometry: aoi,
    scale: Math.max(scaleUse * 3, 300),
    bestEffort: true,
    maxPixels: 1e12,
    tileScale: 4
  }));

  var p2 = ee.Number(stats.get(bandName + '_p2'));
  var p98 = ee.Number(stats.get(bandName + '_p98'));
  var span = p98.subtract(p2);
  var safeSpan = ee.Number(ee.Algorithms.If(span.abs().lt(1e-6), 1, span));

  return ee.Image(img).subtract(p2).divide(safeSpan).clamp(0, 1);
}
```

This improves local map contrast and reduces the influence of outliers. The trade-off is that scores are relative to the selected area.

## 14. Visualisation functions

The app uses two visualisation helpers:

```javascript
function vizContinuous(img, aoi, palette, gamma)
```

and:

```javascript
function vizClass(img, aoi, palette)
```

These functions:

- Clip the image to the AOI.
- Unmask to zero.
- Apply optional gamma correction.
- Set MERIT projection.
- Reproject to the render scale.
- Return a visualised RGB image.

## 15. Layer management

Layers are stored in the `appLayers` object. They are added with:

```javascript
addManagedLayer(key, label, eeObj, shownByDefault, legendType)
```

Each layer object stores:

- label.
- Earth Engine map layer.
- legend type.
- shown status.
- default status.

Only one layer can be selected at a time.

## 16. Opacity control

The opacity slider controls the active layer only. When the user selects another layer, the script applies the current opacity value to the new active layer.

Boundary opacity is not controlled by the slider because the boundary is added separately outside the managed-layer system.

## 17. Legend system

The legend is rebuilt when the selected layer changes. The script supports the following legend types:

- `suitabilityClass`
- `intervention`
- `continuousGreen`
- `continuousOrange`
- `continuousRed`
- `continuousPurple`
- `debugBlue`
- `debugGreen`
- `debugRed`

The recommended intervention setting has a categorical legend with explanatory text.

## 18. Input layer construction

The main input function is:

```javascript
function buildInputs(aoi, scaleUse)
```

It creates all layers required by the model.

### 18.1 FABDEM slope

FABDEM is smoothed before slope is calculated:

```javascript
var demSlope = demRaw
  .focal_median({radius: 45, units: 'meters'})
  .focal_mean({radius: 60, units: 'meters'})
  .rename('demSlope');
```

The app derives:

- `slope`
- `slopeGentle`
- `slopeSteep`

### 18.2 MERIT convergence

MERIT elevation is used for convergence:

```javascript
var meritMean250 = meritElv.focal_mean({radius: 250, units: 'meters'});
var meritMean750 = meritElv.focal_mean({radius: 750, units: 'meters'});
```

The convergence layer is a broad indicator of local low terrain relative to surrounding mean elevation.

### 18.3 MERIT UPA

UPA is log-transformed and locally normalised:

```javascript
var upaLog = meritUpa.log10().rename('upaLog');
var upaN = normalizeByArea(upaLog, 'meritUpaLog', aoi, scaleUse).rename('upaN');
```

### 18.4 MERIT HAND

The app derives:

- `handVeryLow`
- `handLow`
- `handModerate`
- `handShallowRisk`

These represent relative vertical position above drainage.

### 18.5 ERGo landforms

Landform classes are grouped into:

- `lfValley`
- `lfSlope`
- `lfRidge`

### 18.6 Footslope index

The footslope index combines:

- local slope transition.
- convergence.
- moderate HAND.
- slope landform.

This helps identify lower-slope transition zones.

### 18.7 Depth to bedrock

Depth-to-bedrock is transformed into `dtbUseful`.

The transformation favours intermediate-to-deep storage/headroom conditions and reduces favourability at very shallow and very deep extremes.

### 18.8 Aridity

Aridity is transformed into `aiModerate`, favouring moderate aridity.

### 18.9 Land cover

WorldCover is used to derive:

- `openLand`
- `builtRisk`

Open land includes shrubland, grassland, cropland, and bare/sparse vegetation.

### 18.10 Drainage wetness

Drainage wetness combines:

- very low HAND.
- low HAND.
- UPA × low HAND.
- convergence × low HAND.

This avoids dependence on sparse permanent water/wetland classes.

### 18.11 Field capacity

The function `top60MeanOpenLand()` calculates a weighted top-60 cm average from the OpenLandMap field-capacity bands.

## 19. Model construction

The main model function is:

```javascript
function buildModel(x, aoi)
```

It calculates:

- `river`
- `valley`
- `foot`
- `upland`
- `storage`
- `infiltration`
- `feasibility`
- `wetConstraint`
- `constraint`
- `hardExclusion`

## 20. Opportunity scores

The four MAR setting scores are computed with weighted sums and clamped to 0-1.

### 20.1 Riverbank / floodplain

Dominant predictors:

- UPA.
- low HAND.
- drainage wetness.
- valley landform.

### 20.2 Tributary / valley

Dominant predictors:

- moderate HAND.
- valley landform.
- convergence.
- UPA.

### 20.3 Footslope / colluvial

Dominant predictors:

- footslope index.
- convergence.
- moderate HAND.
- slope landform.

### 20.4 Upland / distributed

Dominant predictors:

- open land.
- gentle slope.
- inverse UPA.
- inverse low HAND.
- field capacity.
- aridity.

## 21. Feasibility model

Feasibility combines:

- storage.
- infiltration.
- inverse built-up risk.

The formula is:

```text
feasibility = storage^0.46 × infiltration^0.42 × (1 - builtRisk)^0.12
```

## 22. Constraint model

Constraint combines:

- wetness.
- shallow HAND.
- UPA × low HAND.
- steep slope.
- built-up land.
- ridge landform.

The model uses nonlinear union logic.

## 23. Combined opportunity and suitability

The type-level weights are normalised by `getWeights()`.

Opportunity is:

```text
opportunity = 0.50 × weightedMean + 0.50 × localBest
```

Combined suitability is:

```text
combined = opportunity × feasibility^1.15 × penalty
```

## 24. Confidence score

Confidence is based on:

- margin between best and second-best intervention setting.
- feasibility.
- inverse constraint.

It is a model clarity score, not a statistical probability.

## 25. Recommended MAR intervention setting

The code calculates the highest of the four setting scores:

- river.
- valley.
- foot.
- upland.

The categorical result is displayed as:

1. Riverbank / floodplain.
2. Tributary / valley.
3. Footslope / colluvial.
4. Upland / distributed.

## 26. Main update function

The main runtime function is:

```javascript
function updateSuitabilityMap()
```

It performs the full model run after the user presses the Run button.

## 27. Startup routine

At startup, the app:

1. Populates the country selector.
2. Centres the map on the default country.
3. Displays a ready status message.

The model does not run automatically at startup. This prevents unnecessary computation.

## 28. Technical limitations

The current implementation has the following limitations:

- Large national AOIs may be slow.
- Local percentile normalisation requires server-side reductions.
- Earth Engine UI styling is limited.
- The app does not export rasters or tables.
- The model is not modularised into separate library files.
- Only one layer can be viewed at a time.
- The boundary layer is always visible and not controlled by the opacity slider.
- The model uses global layers that may not reflect local conditions.

## 29. Scientific limitations

The model does not include:

- groundwater depth.
- groundwater-level fluctuation.
- borehole yield.
- transmissivity.
- lithology.
- fracture density.
- water quality.
- runoff volume.
- flood frequency.
- sediment load.
- land tenure.
- protected areas.
- legal constraints.
- cost.
- operation and maintenance capacity.

## 30. Recommended programming improvements

Potential development improvements include:

1. Add export to GeoTIFF.
2. Add export to CSV by admin area.
3. Add summary statistics panel.
4. Add charted histograms.
5. Add downloadable model metadata.
6. Add editable weights through the UI.
7. Add preset scenarios.
8. Add sensitivity-analysis mode.
9. Add local validation-point upload.
10. Add external layer upload.
11. Add optional protected-area masks.
12. Add modular code structure.
13. Add precomputed national assets for performance.
14. Add automated report generation.

## 31. Recommended hydrogeological improvements

Potential scientific improvements include:

1. Add depth-to-groundwater data.
2. Add groundwater-level monitoring.
3. Add borehole-yield calibration.
4. Add aquifer transmissivity.
5. Add lithology.
6. Add weathered-zone thickness.
7. Add lineament or fracture indicators where validated.
8. Add infiltration-test data.
9. Add runoff and rainfall intensity data.
10. Add flood hazard.
11. Add sediment-clogging risk.
12. Add water-quality risk.
13. Add land-tenure and safeguards constraints.

## 32. Running the tool

To run the tool:

1. Open Google Earth Engine Code Editor.
2. Paste the full JavaScript code.
3. Confirm access to:
   - `projects/washways/assets/BDTICMM250m`
   - `projects/washways/assets/AridityIndexv31yrFixed`
4. Click Run.
5. Select country, ADM1, and ADM2.
6. Click Run current selection.
7. Select a layer from the sidebar.
8. Adjust opacity if needed.

## 33. Interpreting results

Use the combined suitability class for first-pass screening.

Use the recommended MAR intervention setting to understand which intervention family is most consistent with the landscape.

Use feasibility, constraint, confidence, and diagnostic layers to understand why an area scores as it does.

Do not use the output as final investment evidence without field validation.

## 34. References

[^1]: Dillon, P. 2005. Future management of aquifer recharge. *Hydrogeology Journal* 13: 313-316. https://doi.org/10.1007/s10040-004-0413-6

[^2]: Russo, T. A., Fisher, A. T., and Lockwood, B. S. 2015. Assessment of managed aquifer recharge site suitability using a GIS and modeling. *Groundwater* 53(3): 389-400. https://doi.org/10.1111/gwat.12213

[^3]: Sallwey, J., Bonilla Valverde, J. P., Vásquez López, F., Junghanns, R., and Stefan, C. 2019. Suitability maps for managed aquifer recharge: a review of multi-criteria decision analysis studies. *Environmental Reviews* 27(2): 138-150. https://doi.org/10.1139/er-2018-0069

[^4]: Maghribi, A. A., Hashim, S. J. B., Aziz, N. A. B. A., and Akib, S. 2022. Geographic information system and multi-criteria decision analysis for managed aquifer recharge site suitability mapping: a review. *Water Supply* 22(9): 7027-7048. https://doi.org/10.2166/ws.2022.288

[^5]: Yamazaki, D., Ikeshima, D., Sosa, J., Bates, P. D., Allen, G. H., and Pavelsky, T. M. 2019. MERIT Hydro: A high-resolution global hydrography map based on latest topography dataset. *Water Resources Research* 55: 5053-5073. https://doi.org/10.1029/2019WR024873

[^6]: Hawker, L., Uhe, P., Paulo, L., Sosa, J., Savage, J., Sampson, C., and Neal, J. 2022. A 30 m global map of elevation with forests and buildings removed. *Environmental Research Letters* 17: 024016. https://doi.org/10.1088/1748-9326/ac4d4f

[^7]: Nobre, A. D., Cuartas, L. A., Hodnett, M., Rennó, C. D., Rodrigues, G., Silveira, A., Waterloo, M., and Saleska, S. 2011. Height Above the Nearest Drainage, a hydrologically relevant new terrain model. *Journal of Hydrology* 404(1-2): 13-29. https://doi.org/10.1016/j.jhydrol.2011.03.051

[^8]: Zanaga, D., Van De Kerchove, R., Daems, D., De Keersmaecker, W., Brockmann, C., Kirches, G., Wevers, J., Cartus, O., Santoro, M., Fritz, S., Lesiv, M., Herold, M., Tsendbazar, N. E., Xu, P., Ramoino, F., and Arino, O. 2022. ESA WorldCover 10 m 2021 v200. https://doi.org/10.5281/zenodo.7254221

[^9]: Hengl, T., Mendes de Jesus, J. M., Heuvelink, G. B. M., Ruiperez Gonzalez, M., Kilibarda, M., Blagotić, A., Shangguan, W., Wright, M. N., Geng, X., Bauer-Marschallinger, B., et al. 2017. SoilGrids250m: global gridded soil information based on machine learning. *PLOS ONE* 12(2): e0169748. https://doi.org/10.1371/journal.pone.0169748
