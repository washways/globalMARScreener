# Managed Aquifer Recharge Suitability Screening Methodology

## Purpose and scope

This methodology describes the current Google Earth Engine implementation of the Global Managed Aquifer Recharge Screener. The model should be understood as a scale-aware, hydrogeology-conditioned MAR screening framework that combines hydrogeomorphic opportunity, physical feasibility, constraint logic, and geological or hydrogeological suitability.

The current implementation uses two uploaded hydrogeology source layers directly:

- GLiM: `projects/washways/assets/glim_global_source`
- GLHYMPS: `projects/washways/assets/glhymps_global_source`

The purpose is to support technically competent hydrogeologists, groundwater planners, GIS analysts, and water-resource specialists who need a transparent first-pass screen of where MAR interventions may deserve further investigation. The model is not intended to identify final construction sites, replace field hydrogeology, define engineering designs, estimate recharge volumes, or demonstrate operational feasibility without further work.

All outputs should be read as screening and prioritization layers. They do not represent calibration, validation, permitting readiness, economic viability, or environmental clearance.

## Conceptual framework

The current model separates seven linked concepts.

1. Opportunity: hydrogeomorphic opportunity for runoff concentration, retention, or infiltration.
2. Feasibility: broad physical plausibility of recharge based on storage, infiltration, land cover, and hydrogeology-conditioned factors.
3. Constraint: screening-level penalties from wetness, steep slopes, built-up land, ridges, and poor hydrogeological conditions.
4. Hydrogeology conditioning: lithological and aquifer-property proxies from GLiM and GLHYMPS.
5. Absolute suitability: fixed-threshold screening intended to preserve comparability across larger or heterogeneous regions.
6. Contextual suitability: within-AOI interpretation intended to identify the best relative areas inside the selected geography.
7. Hybrid suitability: the operational screening output that blends absolute and contextual information.

The model also produces intervention-specific suitability outputs for four broad MAR setting families: riverbank or floodplain, tributary or valley, footslope or colluvial, and upland or distributed.

This framing replaces the older terrain-led description. Terrain remains important, but the active logic now combines terrain, hydrography, land cover, soils, depth to bedrock, aridity, lithology, and hydrogeology.

## Datasets

The model uses globally available base layers and two required uploaded user assets.

### Administrative layers

- FAO GAUL 2015 level 0, level 1, and level 2 are used for AOI selection only.

### Terrain and hydrography

- FABDEM is used for smoothed slope derivation.
- MERIT Hydro provides upstream drainage area, height above nearest drainage, and elevation.
- ERGo landforms provide broad valley, slope, and ridge classes.

### Land cover, soils, and climatic context

- ESA WorldCover provides open-land and built-up screening classes.
- OpenLandMap field capacity provides a soil-water storage proxy.
- A user asset for depth to bedrock is used as a storage headroom proxy.
- A user asset for aridity index provides climatic context.

### Hydrogeology source layers

The current implementation requires two uploaded assets to exist and be accessible:

- `projects/washways/assets/glim_global_source`
- `projects/washways/assets/glhymps_global_source`

GLiM is used as a lithology text source. GLHYMPS is used as a hydrogeological property and descriptive-text source. These layers are integrated directly into the model rather than treated as placeholders.

## Hydrogeomorphic indicators

The hydrogeomorphic backbone of the model remains important because MAR suitability often depends on where runoff accumulates, where drainage-adjacent storage is plausible, and where topographic setting supports either focused or distributed recharge.

The main derived indicators are:

- slope, gentle slope, and steep slope;
- terrain convergence from MERIT elevation;
- upstream drainage area, log-transformed and normalized within the AOI;
- several HAND-derived layers representing very low, low, moderate, and shallow-risk drainage position;
- valley, slope, and ridge landform indicators;
- a footslope or slope-break index built from local slope transition, convergence, moderate HAND, and slope landform;
- open-land and built-up masks;
- field-capacity-derived soil favorability;
- drainage wetness risk.

These indicators drive the opportunity families, support feasibility, and contribute to constraint logic.

## Hydrogeology conditioning using GLiM and GLHYMPS

Hydrogeology conditioning is now an essential part of the model.

### GLiM contribution

The model uses an uploaded GLiM feature collection and reads lithological text from `Litho`, with fallback to `xx`. The lithology text is interpreted heuristically into broad favorability classes. The current logic distinguishes categories such as:

- unconsolidated, alluvial, sandy, and gravelly settings;
- carbonate and karst-related settings;
- crystalline, basement, and metamorphic settings;
- clay-rich, marl, shale, mudstone, and other lower-permeability settings.

This produces a GLiM lithology score.

### GLHYMPS contribution

The model uses an uploaded GLHYMPS feature collection and reads the following fields directly:

- `logK_Ferr_`
- `Porosity_x`
- `K_stdev_x1`
- `Transmissi`
- `GUM_K`
- `Descriptio`

From these, the model derives:

- a permeability score from estimated log hydraulic conductivity;
- a porosity score from `Porosity_x`;
- an uncertainty or stability modifier from `K_stdev_x1`;
- a contextual transmissivity score from `Transmissi`;
- a class-based hydrogeology score from `GUM_K`;
- a description-based score from the GLHYMPS descriptive text.

### Hydrogeology modifier construction

The GLiM and GLHYMPS components are blended into a hydrogeology modifier that is constrained roughly between 0.15 and 1.0. In the current implementation, hydraulic conductivity, descriptive classification, lithology text, GUM_K class, porosity, transmissivity context, and uncertainty all contribute, with stronger emphasis on conductivity and descriptive or lithological favorability.

The modifier is used:

- positively in storage;
- positively in infiltration;
- indirectly in feasibility;
- as a low-hydrogeology penalty in constraint; and
- as part of hard exclusion when hydrogeological suitability is extremely low.

This modifier is physically motivated, but it remains a proxy. It is not a substitute for local aquifer maps, pumping tests, borehole stratigraphy, groundwater levels, or fracture-network characterization.

## Opportunity model

The opportunity component represents hydrogeomorphic potential for runoff concentration, drainage-adjacent recharge, slope-transition infiltration, or distributed recharge-supporting measures. The model retains four intervention-family opportunity surfaces.

### Riverbank or floodplain opportunity

This surface emphasizes high upstream drainage area, low HAND, drainage wetness, valley landform, gentle slopes, convergence, and open land. It is intended to capture river corridors, floodplain-like settings, and alluvial recharge opportunities.

### Tributary or valley opportunity

This surface emphasizes moderate HAND, valley landforms, convergence, upstream drainage area, gentle slopes, slope-break support, and open land. It is intended to capture smaller valleys, tributary corridors, and valley-margin runoff capture settings.

### Footslope or colluvial opportunity

This surface emphasizes the slope-break or footslope index, convergence, moderate HAND, slope landforms, gentle slopes, open land, and moderate aridity support. It is intended to highlight lower-slope positions where runoff and shallow subsurface flow may accumulate.

### Upland or distributed opportunity

This surface emphasizes open land, gentle slope, reduced drainage concentration, lower dependence on low HAND settings, field capacity support, moderate aridity, and non-valley positions. It is intended to identify more distributed recharge-supporting landscapes.

For both contextual and absolute logic, these four opportunity surfaces are aggregated through a fixed family weighting scheme and a best-family component. The fixed family weights remain constant in the current model. Scale adaptation happens later through the blending of absolute and contextual suitability, not by changing the opportunity family weights.

## Feasibility model

Feasibility is no longer only terrain-plus-soil driven. It now includes explicit hydrogeology conditioning.

### Storage component

The storage component combines:

- depth-to-bedrock usefulness;
- moderate HAND;
- convergence;
- upstream drainage area;
- field capacity;
- slope break;
- hydrogeology modifier.

The intent is to identify places where some storage headroom, convergent setting, and hydrogeological favorability align.

### Infiltration component

The infiltration component combines:

- gentle slope;
- open land;
- field capacity;
- slope break;
- inverse wetness;
- inverse shallow HAND risk;
- hydrogeology modifier.

The intent is to favor areas where infiltration is more plausible and less likely to be dominated by immediate waterlogging, excessive slope, or highly unfavorable geological settings.

### Feasibility aggregation

Storage and infiltration are combined multiplicatively, with a further reduction from built-up land. This preserves the screening logic that weak links matter. Strong hydrogeomorphic opportunity alone is not enough if storage or infiltration plausibility is poor.

## Constraint model

Constraint is now conditioned partly by hydrogeology and is not limited to wetness and terrain penalties.

The current constraint logic includes:

- wetness risk;
- steep slopes;
- built-up land;
- ridge setting;
- a low-hydrogeology penalty.

The low-hydrogeology term is derived from the inverse of the hydrogeology modifier. This means hydrogeological unsuitability can both reduce feasibility and increase constraint.

The model also applies hard exclusion where conditions are extremely unfavorable, including very high wetness, very steep slopes, very strong built-up presence, or extremely low hydrogeological suitability.

Hydrogeology conditioning therefore influences both feasibility and constraint rather than sitting outside the core scoring logic.

## Absolute, contextual, and hybrid suitability

### Why the three-frame logic was added

Administrative units are not hydrogeological units. Pure within-AOI normalization can distort large, heterogeneous regions because the best-ranked pixels in a large administrative region may still be poor in absolute hydrogeological terms. Large regions, including cases such as French regional units, can therefore yield misleading relative rankings if only contextual logic is used.

The current model addresses this by separating absolute, contextual, and hybrid suitability.

### Absolute suitability

Absolute suitability is intended to preserve broader comparability across larger or heterogeneous regions. It uses fixed physical suitability logic based on absolute opportunity, absolute feasibility, and absolute constraint penalty.

### Contextual suitability

Contextual suitability is intended to identify the best relative areas within the selected AOI. It uses AOI-based normalization and percentile interpretation. Higher contextual classes also require minimum absolute suitability floors so that locally high percentile ranks do not automatically imply strong physical suitability.

### Hybrid suitability

Hybrid suitability is the operational screening output. It blends absolute and contextual suitability. This is the main default output shown to users.

The blend is area-aware. Contextual weight is reduced as AOI size increases, using AOI area in square kilometres. Small AOIs receive more contextual influence. Very large AOIs rely more on absolute suitability. This is presented as a pragmatic scaling rule intended to reduce overfitting of contextual ranking in large heterogeneous administrative regions, not as a universal hydrological law.

## Intervention-specific suitability

The model now exposes intervention-family suitability outputs directly rather than only inferring a dominant type.

The four current intervention-family suitability outputs are:

- Riverbank / floodplain
- Tributary / valley
- Footslope / colluvial
- Upland / distributed

Each intervention-specific suitability surface combines the corresponding intervention-family opportunity signal with absolute feasibility and absolute penalty logic. The dominant intervention setting is then taken as the highest of the four suitability surfaces at each pixel.

This means the intervention setting is no longer just a categorical byproduct. It sits alongside explicit intervention-specific suitability layers that users can inspect separately.

## Interpretation guidance

Hybrid suitability should be used as the default screening layer because it balances broader physical comparability with local relevance.

Absolute suitability is most useful when comparing larger or heterogeneous AOIs, reviewing national or regional patterns, or checking whether contextual highlights remain strong under fixed physical logic.

Contextual suitability is most useful when users want to highlight the best relative areas within one selected geography. It should not be treated as directly comparable between unrelated AOIs.

Feasibility and constraint should be read together. High opportunity with low feasibility indicates a promising hydrogeomorphic position but weak storage, infiltration, land-use, or hydrogeological support. High opportunity with high constraint indicates a potentially attractive setting that may still be unsuitable because of wetness, slope, built-up land, ridge position, or poor hydrogeology.

The hydrogeology modifier should be interpreted as a screening proxy. It strengthens the realism of the model but does not remove the need for local hydrogeological review.

## Limitations

The current model has important limitations.

- GLiM and GLHYMPS text interpretation is heuristic.
- Lithological text matching is coarse and may not capture national stratigraphic nuance.
- The hydrogeology modifier is still a proxy, not a substitute for local aquifer maps.
- No national groundwater-body delineations are included yet.
- No explicit recharge water availability term is included yet.
- No groundwater stress or abstraction term is included yet.
- Measured groundwater levels are not included.
- Groundwater seasonal fluctuation is not included.
- Water quality is not included.
- Legal and environmental constraints are not included.
- Local field validation is not included.
- The model has not yet been calibrated against known MAR sites or national case-study inventories.

These limits should be stated plainly. The model should not be described as calibrated or validated unless that work is done separately.

## Recommended future calibration

The most important next improvements are:

- replace heuristic lithology parsing with nationally harmonized hydrogeology classes where available;
- integrate groundwater body or aquifer map boundaries;
- add recharge water availability and groundwater-need layers;
- add groundwater stress or abstraction terms;
- calibrate against known MAR sites and national studies;
- allow normalization by basin, groundwater body, or hydrogeological province rather than only by administrative unit.

These improvements would make the framework less dependent on administrative boundaries and more aligned with hydrogeological decision units.
