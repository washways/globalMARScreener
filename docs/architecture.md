# Architecture Overview

## Purpose

The Global Managed Aquifer Recharge Screener is a screening-oriented Google Earth Engine application for identifying where MAR follow-up may be more promising. The current implementation is scale-aware and hydrogeology-conditioned, with hybrid suitability as the default screening frame.

This repository adds a lightweight public wrapper around the published Earth Engine app so users can access the map, supporting explanation, and documentation from one landing page.

## High-level structure

The project is organized into three layers.

1. Google Earth Engine app
   - Runs the screening model and map interface.
   - Integrates GLiM and GLHYMPS uploaded assets directly.
   - Produces absolute, contextual, and hybrid suitability outputs.

2. Public wrapper site
   - Embeds or links to the live Earth Engine app.
   - Provides a lightweight public entry point and interpretation guidance.

3. Documentation layer
   - README.md provides the practical implementation-oriented description.
   - METHODOLOGY.md provides the hydrogeology-oriented model explanation.
   - The docs folder provides supporting references and deployment notes.

## Conceptual model

The current screening logic combines:

- opportunity, expressed through four hydrogeomorphic intervention families;
- feasibility, built from storage and infiltration logic;
- constraint, including wetness, slope, built-up, ridge, and low-hydrogeology penalties;
- hydrogeology conditioning from GLiM and GLHYMPS;
- three suitability frames: absolute, contextual, and hybrid;
- intervention-specific suitability layers plus a dominant intervention setting;
- confidence as a screening confidence indicator.

## Suitability pathways

### Absolute pathway

Uses fixed physical suitability logic intended to remain more stable across scales.

### Contextual pathway

Uses AOI-based local normalization and percentile interpretation to identify the better relative areas within the selected geography.

### Hybrid pathway

Blends absolute and contextual suitability. This is the main default output.

## Scale-aware behavior

Contextual influence decreases as AOI size increases. This is intended to reduce misleading within-AOI ranking effects in very large heterogeneous administrative regions.

## Public wrapper flow

1. User opens the landing page.
2. The page embeds the live Earth Engine app in an iframe.
3. If embedding is blocked, the user can launch the full app directly.
4. The surrounding sections explain the current outputs and how to interpret them responsibly.

## Repository structure

- index.html: static wrapper page.
- site/: favicon and other visual assets.
- docs/: supporting reference material.
- README.md: practical product overview.
- METHODOLOGY.md: authoritative methodology description.

## Design principle

The wrapper and documentation should describe the current operational model as it exists now, not as a changelog of earlier terrain-led versions.
