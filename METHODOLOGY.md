# Managed Aquifer Recharge Suitability Screening Methodology

## 1. Purpose and scope

This methodology describes a global, screening-level managed aquifer recharge (MAR) suitability tool implemented in Google Earth Engine. The tool is designed to support technically competent hydrogeologists, groundwater planners, water-resource managers, and WASH-sector specialists who need a transparent and repeatable first-pass spatial assessment of where MAR interventions may warrant further investigation.

The tool is not intended to identify final construction sites, define engineering designs, or replace field hydrogeology. It provides a structured geospatial screening framework that helps reduce the search space before more detailed assessment. All outputs should be interpreted as prioritisation layers, not as evidence that a site is technically, legally, socially, economically, or environmentally ready for MAR.

MAR is used here in the modern sense of intentional recharge of water into aquifers or aquifer systems for later use, drought resilience, environmental benefit, groundwater recovery, or conjunctive water-resource management.[^1] The tool is consistent with the broad logic of GIS-based multi-criteria decision analysis (GIS-MCDA), which is widely used in MAR suitability mapping because recharge suitability is controlled by several interacting factors rather than by a single variable.[^2][^3][^4]

The specific purpose is to classify each pixel within a selected country, ADM1, or ADM2 area into relative MAR suitability and broad intervention-setting classes. The tool focuses on hydrogeomorphic opportunity, physical feasibility, and screening-level constraints. It does not directly model recharge volumes, aquifer transmissivity, groundwater-level response, sediment clogging, groundwater quality, water rights, land tenure, environmental safeguards, or lifecycle costs.

## 2. Conceptual foundation

The tool is based on a process-oriented hydrogeological screening concept. A location is considered more promising for MAR when three conditions align:

1. A plausible recharge opportunity exists.
2. The physical setting is broadly feasible.
3. Constraints are limited or manageable.

Recharge opportunity refers to the landscape’s tendency to concentrate, retain, or permit controlled infiltration of water. It is represented by hydrogeomorphic indicators such as upstream drainage area, height above nearest drainage (HAND), valley position, convergence, footslope position, open land, and slope.

Physical feasibility refers to whether the terrain, land cover, soil-water storage proxy, depth-to-bedrock proxy, and drainage position make practical recharge plausible. The feasibility score is intentionally conservative and multiplicative because MAR suitability is often controlled by weak links. A site with strong runoff opportunity but excessive wetness, steep slopes, shallow storage, or built-up land should not score highly.

Constraint refers to conditions that may reduce suitability or require caution. The current model includes drainage wetness, very low HAND, steep slope, built-up land, and ridge settings. It does not include all important constraints. Protected areas, water quality, flood hazard, sediment load, legal restrictions, land tenure, and environmental-flow requirements should be added for operational use.

## 3. Model outputs

The tool produces the following outputs:

1. Combined suitability class.
2. Combined suitability continuous score.
3. Combined opportunity class.
4. Opportunity score.
5. Feasibility score.
6. Constraint score.
7. Confidence score.
8. Recommended MAR intervention setting.
9. Diagnostic input layers.

The default screening layer is the combined suitability class. The most interpretive layer is the recommended MAR intervention setting, which classifies the dominant hydrogeomorphic setting associated with a broad MAR intervention family.

## 4. MAR intervention settings

The model distinguishes four broad intervention settings. These are not engineering designs. They are hydrogeomorphic typologies intended to guide the next stage of investigation.

### 4.1 Riverbank / floodplain setting

This setting describes low-lying, drainage-adjacent areas, river corridors, floodplain-like settings, and alluvial recharge zones. Potential MAR approaches include:

- Riverbank filtration.
- Flood spreading.
- Infiltration basins.
- Alluvial recharge.
- Off-channel recharge basins.
- Seasonal floodwater recharge where water quality, sediment load, and land access permit.

The class is driven mainly by high upstream drainage area, low HAND, drainage wetness, valley landform, gentle slope, convergence, and open land.

This setting should be interpreted cautiously. A floodplain may offer water-source opportunity, but it may also be constrained by flood risk, fine sediments, clogging, groundwater contamination risk, land tenure complexity, environmental-flow needs, and infrastructure exposure.

### 4.2 Tributary / valley setting

This setting describes smaller valleys, tributary corridors, local drainage lines, valley margins, and small-catchment runoff pathways. Potential MAR approaches include:

- Check dams.
- Sand dams.
- Recharge weirs.
- Small valley spreading.
- Gully plugs where geomorphically appropriate.
- Tributary-scale infiltration and runoff-retention structures.

The class is driven mainly by moderate HAND, valley landforms, convergence, upstream drainage area, gentle slope, footslope influence, and open land.

This class is particularly relevant where decentralised structures may be technically simpler than large riverbank or injection schemes. It should still be validated against channel stability, sediment load, flood hydraulics, land rights, and local groundwater response.

### 4.3 Footslope / colluvial setting

This setting describes lower-slope, slope-transition, colluvial, and valley-margin positions where runoff and shallow subsurface flow may accumulate. Potential MAR approaches include:

- Contour trenches.
- Recharge bunds.
- Infiltration trenches.
- Managed runoff capture.
- Hillslope-to-valley recharge structures.
- Soil and water conservation measures that promote infiltration.

The class is driven by the footslope index, convergence, moderate HAND, slope landform, gentle slope, open land, and aridity modifier.

This setting is important in basement-complex and semi-arid rural environments where recharge may depend on weathered zones, fractures, lower-slope storage, and episodic runoff capture. It requires careful local validation because global data cannot reliably identify fracture connectivity, regolith hydraulic properties, or weathered-zone saturation.

### 4.4 Upland / distributed setting

This setting describes open, gentle, non-valley terrain suitable for diffuse recharge-supporting measures. Potential approaches include:

- Soil and water conservation.
- Contour bunds.
- Distributed infiltration.
- Farm ponds where suitable.
- Runoff retention.
- Landscape-scale recharge enhancement.

The class is driven by open land, gentle slope, low drainage concentration, position away from low-HAND drainage zones, field capacity, moderate aridity, and non-valley terrain.

This setting usually points to distributed land-management options rather than discrete engineered recharge infrastructure. It may be important for watershed restoration but is harder to translate into single investment sites.

## 5. Core datasets

The tool uses globally available datasets available in or compatible with Google Earth Engine.

### 5.1 Administrative boundaries

FAO GAUL 2015 provides country, ADM1, and ADM2 boundaries. These are used only for area selection and local normalisation.[^5]

### 5.2 FABDEM

FABDEM is used for slope derivation. FABDEM is a 30 m global elevation product produced by removing forest and building height biases from the Copernicus DEM.[^6] This is useful because slope and local terrain morphology can be distorted by vegetation and urban artefacts in standard surface models.

The model does not use FABDEM for hydrological flow routing. MERIT Hydro is used instead because it provides a hydrologically coherent global hydrography product.

### 5.3 MERIT Hydro

MERIT Hydro is the main hydrological backbone. It provides global hydrography at approximately 3 arc-second resolution, including upstream drainage area, HAND, and elevation.[^7] The model uses:

- `upa`: upstream drainage area.
- `hnd`: height above nearest drainage.
- `elv`: MERIT elevation.

MERIT UPA is used as a drainage-concentration proxy. MERIT HAND is used as a valley, drainage-adjacency, and wetness-risk proxy. MERIT elevation is used for the terrain-convergence index because it is more computationally stable for interactive global screening than focal processing over the FABDEM mosaic.

### 5.4 HAND concept

HAND normalises terrain by vertical distance above the nearest drainage. It is hydrologically meaningful because it captures local topographic position relative to drainage pathways rather than absolute elevation.[^8] In this tool, HAND is central to distinguishing riverbank/floodplain, tributary/valley, footslope, and upland settings.

### 5.5 ESA WorldCover

ESA WorldCover v200 provides a global 10 m land-cover product for 2021 derived from Sentinel-1 and Sentinel-2 data.[^9] The tool uses WorldCover only for:

- Open land.
- Built-up risk.

WorldCover water and wetland classes are not used as the main wetness predictor. In many semi-arid, seasonal, or ephemeral drainage systems, permanent mapped water is too sparse to represent recharge-relevant drainage corridors, dambos, floodplains, or episodic runoff pathways. MERIT HAND and UPA provide a more useful hydrogeomorphic wetness proxy.

### 5.6 ERGo SRTM landforms

The ERGo Global SRTM Landforms dataset provides landform classes derived from CHILI and multi-scale topographic position index.[^10] The tool uses this layer to identify broad valley, slope, and ridge settings. It is used as a supporting geomorphic classifier, not as a primary hydrological model.

### 5.7 OpenLandMap field capacity

OpenLandMap field-capacity data are derived from SoilGrids-based global soil predictions.[^11] The tool uses soil water content at 33 kPa as a field-capacity proxy for the upper 60 cm. This is a soil-water storage modifier, not a measured infiltration rate or saturated hydraulic conductivity.

### 5.8 Depth to bedrock

The model uses the user-provided Earth Engine asset:

`projects/washways/assets/BDTICMM250m`

This is interpreted as a depth-to-bedrock proxy and converted from centimetres to metres. It is used as a screening-level indicator of possible storage headroom and regolith/weathered-zone thickness. It is not a substitute for borehole logs, geophysical interpretation, fractured-bedrock mapping, transmissivity, storage coefficient, or groundwater-level data.

### 5.9 Aridity index

The model uses the user-provided Earth Engine asset:

`projects/washways/assets/AridityIndexv31yrFixed`

The aridity layer acts as a broad climate modifier. Moderate aridity is favoured in some classes because these areas may have MAR relevance while still potentially receiving episodic runoff. The layer is not used to estimate recharge volume, flood magnitude, or water availability.

## 6. Spatial scale and local normalisation

The model supports three operating contexts:

| Mode | Analysis scale | Render scale | Intended use |
|---|---:|---:|---|
| National | 500 m | 2000 m | Strategic screening |
| ADM1 | 250 m | 500 m | Regional prioritisation |
| ADM2 | 120 m | 150 m | District-scale exploration |

Several variables are normalised using the 2nd and 98th percentiles inside the selected AOI. This is done to reveal meaningful local variation and reduce outlier dominance.

This design choice has an important implication: outputs are relative to the selected area. A “very high” score in one ADM2 is not necessarily equivalent to a “very high” score in another ADM2. For inter-district comparison, users should run a common AOI or export and normalise consistently.

## 7. Derived input layers

### 7.1 Slope

Slope is derived from a smoothed FABDEM surface. The smoothing combines focal median and focal mean operations to reduce high-frequency noise before calculating slope. Slope is used to identify gentle terrain, steep constraints, and slope-transition areas.

### 7.2 MERIT terrain convergence

The convergence index compares MERIT elevation against local mean elevation at 250 m and 750 m radii. Pixels lower than their local surroundings receive higher convergence scores. This is a simplified indicator of local hollows, valley-bottom tendency, and convergent terrain.

It is not a full flow-routing model, topographic wetness index, or physically based runoff model. It should be interpreted as a broad terrain-position diagnostic.

### 7.3 Upstream drainage area

MERIT UPA is log-transformed and locally normalised. The log transformation is necessary because upstream drainage area is highly skewed. Without this transformation, large rivers would dominate the drainage signal and obscure tributary-scale opportunities.

### 7.4 HAND classes

The model derives several HAND-related layers:

- Very low HAND.
- Low HAND.
- Moderate HAND.
- Shallow HAND risk.

These are used differently across opportunity, feasibility, and constraints. Low HAND supports riverbank and valley settings, while very low HAND can also indicate flood or wetness constraint.

### 7.5 Footslope index

The footslope index is a synthetic layer based on:

- Local slope transition.
- MERIT convergence.
- Moderate HAND.
- ERGo slope landform.

It is intended to highlight lower-slope and slope-transition settings where runoff accumulation, colluvial deposits, and shallow lateral flow may support recharge interventions.

### 7.6 Open land

Open land is derived from WorldCover shrubland, grassland, cropland, and bare/sparse vegetation. It acts as a broad practicality indicator because many MAR interventions require accessible, non-built land. It is not a land-tenure or safeguards layer.

### 7.7 Built-up risk

Built-up risk is derived from the WorldCover built-up class. It reduces feasibility and increases constraint. It is a coarse global screen and may miss small settlements or informal development.

### 7.8 Field capacity

Field capacity is computed from OpenLandMap depths at 0, 10, 30, and 60 cm as a top-60 cm weighted average. The result is locally normalised and transformed into a moderate-favourability score. It should not be confused with hydraulic conductivity or infiltration capacity.

### 7.9 Drainage wetness

Drainage wetness is derived from:

- Very low HAND.
- Low HAND.
- UPA combined with low HAND.
- Convergence combined with low HAND.

It intentionally excludes WorldCover water/wetland as a main driver because mapped permanent water is often too sparse for MAR screening in ephemeral drainage environments.

## 8. Weighting rationale

The model weights were proposed by AI as transparent expert-judgement priors. They are not empirically calibrated coefficients and should not be presented as universal MAR suitability weights. Their rationale is based on hydrogeological process logic, the expected relevance of each predictor to each MAR setting, and the relative reliability of the available global datasets.

The weights follow five principles.

### 8.1 Primary process controls receive the highest weights

For each MAR setting, variables that describe the main physical control receive the highest weight. For riverbank/floodplain settings, upstream drainage area and low HAND dominate because that setting depends on drainage concentration and proximity to drainage corridors. For tributary/valley settings, moderate HAND, valley form, convergence, and UPA receive high weight because they represent valley-margin and small-catchment recharge opportunities. For footslope/colluvial settings, the footslope index receives the highest weight because the target process is runoff accumulation and infiltration at slope-transition zones. For upland/distributed settings, open land and gentle slope dominate because diffuse recharge-supporting interventions require accessible, low-gradient terrain.

### 8.2 Secondary controls receive moderate weights

Slope, convergence, open land, landform class, and field capacity support interpretation but should not dominate alone. Open land improves practical feasibility, but open land without runoff opportunity or storage potential is not sufficient for high suitability. Field capacity provides a useful soil-water modifier, but it is not a direct substitute for measured infiltration rate.

### 8.3 Contextual modifiers receive lower weights

Aridity and broad landform modifiers are retained as contextual variables but are deliberately not allowed to dominate. This avoids over-interpreting global layers that cannot capture local aquifer hydraulics, fracture connectivity, or seasonal runoff availability.

### 8.4 Constraint logic is conservative

The model penalises high wetness risk, very low HAND, built-up land, steep slope, and ridge settings. The use of multiplicative feasibility and nonlinear constraint logic reflects the fact that MAR suitability is often limited by weak links. A site with strong drainage opportunity but excessive flood risk, urban land cover, steep slopes, or poor practical infiltration conditions should not rank highly.

### 8.5 Type-level weights reflect a planning hypothesis

The type-level weights are:

- Riverbank / floodplain: 18 percent.
- Tributary / valley: 34 percent.
- Footslope / colluvial: 30 percent.
- Upland / distributed: 18 percent.

Tributary/valley and footslope/colluvial settings receive higher aggregate weights because the model is designed to be useful for decentralised MAR screening in semi-arid, basement-complex, rural, and data-limited contexts, where small valleys, tributary systems, lower slopes, colluvial zones, and runoff-retention settings may be more actionable than either large riverbank schemes or diffuse upland measures. This is a planning assumption, not a universal hydrogeological rule.

The weights should be tested through sensitivity analysis and recalibrated against local evidence, including borehole yield, groundwater-level response, geophysics, lithology, soil-infiltration testing, mapped alluvial aquifers, known successful recharge schemes, and failed MAR interventions.

## 9. MAR opportunity sub-models

### 9.1 Riverbank / floodplain

| Predictor | Weight |
|---|---:|
| UPA | 0.34 |
| Low HAND | 0.24 |
| Drainage wetness | 0.16 |
| Valley landform | 0.10 |
| Gentle slope | 0.07 |
| Convergence | 0.05 |
| Open land | 0.04 |

### 9.2 Tributary / valley

| Predictor | Weight |
|---|---:|
| Moderate HAND | 0.25 |
| Valley landform | 0.20 |
| Convergence | 0.20 |
| UPA | 0.15 |
| Gentle slope | 0.10 |
| Footslope index | 0.06 |
| Open land | 0.04 |

### 9.3 Footslope / colluvial

| Predictor | Weight |
|---|---:|
| Footslope index | 0.34 |
| Convergence | 0.20 |
| Moderate HAND | 0.18 |
| Slope landform | 0.12 |
| Gentle slope | 0.08 |
| Open land | 0.05 |
| Moderate aridity | 0.03 |

### 9.4 Upland / distributed

| Predictor | Weight |
|---|---:|
| Open land | 0.24 |
| Gentle slope | 0.22 |
| Inverse UPA | 0.15 |
| Inverse low HAND | 0.12 |
| Field capacity | 0.12 |
| Moderate aridity | 0.10 |
| Inverse valley landform | 0.05 |

## 10. Feasibility sub-model

### 10.1 Storage component

| Predictor | Weight |
|---|---:|
| Depth-to-bedrock usefulness | 0.30 |
| Moderate HAND | 0.22 |
| Convergence | 0.18 |
| UPA | 0.12 |
| Field capacity | 0.10 |
| Footslope index | 0.08 |

### 10.2 Infiltration component

| Predictor | Weight |
|---|---:|
| Gentle slope | 0.28 |
| Open land | 0.20 |
| Field capacity | 0.17 |
| Footslope index | 0.11 |
| Inverse drainage wetness | 0.14 |
| Inverse shallow HAND risk | 0.10 |

### 10.3 Combined feasibility

Feasibility is computed as:

```text
feasibility = storage^0.46 × infiltration^0.42 × (1 - builtRisk)^0.12
```

The formulation is deliberately multiplicative. It limits unrealistic compensation between unrelated factors.

## 11. Constraint sub-model

Constraint is calculated from:

- Drainage wetness.
- Very shallow HAND.
- UPA combined with low HAND.
- Steep slope.
- Built-up land.
- Ridge landform.

The model uses nonlinear union logic:

```text
constraint = 1 - Π(1 - partialConstraint)
```

This means a strong constraint remains influential and cannot be fully diluted by favourable values elsewhere.

## 12. Opportunity and suitability aggregation

The model combines the four MAR type scores using:

```text
opportunity = 0.50 × weightedMean + 0.50 × localBest
```

The weighted mean reflects balanced multi-setting opportunity. The local-best component ensures specialised settings remain visible.

Combined suitability is:

```text
combined = opportunity × feasibility^1.15 × penalty
```

where:

```text
penalty = 1 - constraint^1.2 × constraintPenaltyStrength
```

with a minimum penalty floor.

## 13. Classification

Continuous scores are converted into classes:

- Excluded.
- Low.
- Moderate.
- High.
- Very high.

Current thresholds are:

| Output | Low to moderate | Moderate to high | High to very high |
|---|---:|---:|---:|
| Opportunity | 0.18 | 0.40 | 0.65 |
| Suitability | 0.22 | 0.45 | 0.72 |

These thresholds are heuristic and should be calibrated locally.

## 14. Confidence score

Confidence is a model-internal clarity score. It is not a statistical confidence interval.

It combines:

- Margin between best and second-best intervention setting.
- Feasibility.
- Inverse constraint.

High confidence indicates that one intervention setting is clearly dominant and that feasibility and constraint scores are coherent. Low confidence may indicate ambiguity, mixed signals, or strong constraints.

## 15. Interpretation guidance

A high combined suitability class indicates that opportunity, feasibility, and low constraint align.

A high opportunity but low suitability score indicates a promising landscape position with practical or physical limitations.

A high feasibility but low opportunity score indicates generally favourable physical conditions but limited terrain or drainage reason to prioritise MAR.

A high constraint score should trigger caution.

The recommended MAR intervention setting should be treated as a typology for field planning, not as an engineering prescription.

## 16. Limitations

The model does not include:

- Groundwater depth.
- Seasonal groundwater fluctuation.
- Borehole yield.
- Aquifer transmissivity.
- Storage coefficient.
- Lithology.
- Fracture density.
- Weathered-zone hydraulic properties.
- Water quality.
- Sediment load.
- Surface-water reliability.
- Flood frequency.
- Protected areas.
- Land tenure.
- Environmental safeguards.
- Community acceptance.
- Engineering cost.
- O&M capacity.
- Legal and institutional feasibility.

## 17. Validation requirements

Operational use requires validation with:

- Borehole logs.
- Borehole yield and pumping-test data.
- Groundwater-level monitoring.
- Electrical resistivity or other geophysics.
- Soil infiltration tests.
- Field mapping of drainage, floodplain, dambo, and alluvial settings.
- Known successful and failed MAR structures.
- Water-quality sampling.
- Sediment-load assessment.
- Land-tenure and safeguards review.

## 18. Recommended improvements

Priority improvements include:

1. Add groundwater depth and seasonal fluctuation.
2. Add lithology and weathered-zone mapping.
3. Add borehole-yield calibration.
4. Add local aquifer transmissivity and storage data.
5. Add runoff availability by season.
6. Add flood hazard and sediment-load risk.
7. Add protected areas and land-tenure constraints.
8. Add access and construction-cost proxies.
9. Add calibration by hydrogeological province.
10. Add export functions for raster and administrative summaries.
11. Add sensitivity analysis for weights and thresholds.
12. Add validation dashboards using observed data.

## References

[^1]: Dillon, P. 2005. Future management of aquifer recharge. *Hydrogeology Journal* 13: 313-316. https://doi.org/10.1007/s10040-004-0413-6

[^2]: Russo, T. A., Fisher, A. T., and Lockwood, B. S. 2015. Assessment of managed aquifer recharge site suitability using a GIS and modeling. *Groundwater* 53(3): 389-400. https://doi.org/10.1111/gwat.12213

[^3]: Sallwey, J., Bonilla Valverde, J. P., Vásquez López, F., Junghanns, R., and Stefan, C. 2019. Suitability maps for managed aquifer recharge: a review of multi-criteria decision analysis studies. *Environmental Reviews* 27(2): 138-150. https://doi.org/10.1139/er-2018-0069

[^4]: Maghribi, A. A., Hashim, S. J. B., Aziz, N. A. B. A., and Akib, S. 2022. Geographic information system and multi-criteria decision analysis for managed aquifer recharge site suitability mapping: a review. *Water Supply* 22(9): 7027-7048. https://doi.org/10.2166/ws.2022.288

[^5]: FAO. 2015. Global Administrative Unit Layers (GAUL). FAO and Google Earth Engine Data Catalog. https://developers.google.com/earth-engine/datasets/catalog/FAO_GAUL_2015_level0

[^6]: Hawker, L., Uhe, P., Paulo, L., Sosa, J., Savage, J., Sampson, C., and Neal, J. 2022. A 30 m global map of elevation with forests and buildings removed. *Environmental Research Letters* 17: 024016. https://doi.org/10.1088/1748-9326/ac4d4f

[^7]: Yamazaki, D., Ikeshima, D., Sosa, J., Bates, P. D., Allen, G. H., and Pavelsky, T. M. 2019. MERIT Hydro: A high-resolution global hydrography map based on latest topography dataset. *Water Resources Research* 55: 5053-5073. https://doi.org/10.1029/2019WR024873

[^8]: Nobre, A. D., Cuartas, L. A., Hodnett, M., Rennó, C. D., Rodrigues, G., Silveira, A., Waterloo, M., and Saleska, S. 2011. Height Above the Nearest Drainage, a hydrologically relevant new terrain model. *Journal of Hydrology* 404(1-2): 13-29. https://doi.org/10.1016/j.jhydrol.2011.03.051

[^9]: Zanaga, D., Van De Kerchove, R., Daems, D., De Keersmaecker, W., Brockmann, C., Kirches, G., Wevers, J., Cartus, O., Santoro, M., Fritz, S., Lesiv, M., Herold, M., Tsendbazar, N. E., Xu, P., Ramoino, F., and Arino, O. 2022. ESA WorldCover 10 m 2021 v200. https://doi.org/10.5281/zenodo.7254221

[^10]: Conservation Science Partners. ERGo Global SRTM Landforms. Google Earth Engine Data Catalog. https://developers.google.com/earth-engine/datasets/catalog/CSP_ERGo_1_0_Global_SRTM_landforms

[^11]: Hengl, T., Mendes de Jesus, J. M., Heuvelink, G. B. M., Ruiperez Gonzalez, M., Kilibarda, M., Blagotić, A., Shangguan, W., Wright, M. N., Geng, X., Bauer-Marschallinger, B., et al. 2017. SoilGrids250m: global gridded soil information based on machine learning. *PLOS ONE* 12(2): e0169748. https://doi.org/10.1371/journal.pone.0169748
