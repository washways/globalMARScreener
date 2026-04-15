# Architecture Overview

## Purpose

The Global Managed Aquifer Recharge Screener is a screening-oriented Google Earth Engine application for exploring where managed aquifer recharge may be more suitable for follow-up investigation.

This repository adds a lightweight public wrapper around the published Earth Engine app so users can access the map, supporting explanation, and documentation from a single landing page.

## High-level structure

The project is organised into three layers:

1. Google Earth Engine app
   - The interactive screening tool runs on Earth Engine infrastructure.
   - Users open the live app directly or through the embedded iframe on the landing page.

2. Public wrapper site
   - The root landing page explains the model in plain language.
   - It links to the methodology, repository, and deployment notes.
   - It is designed for simple static hosting at a WASHways path or via GitHub Pages.

3. Documentation layer
   - README.md provides the product overview and technical summary.
   - METHODOLOGY.md provides the scientific and hydrogeological basis.
   - The docs folder provides deployment, architecture, and dataset references.

## Conceptual model

The screening logic combines several ideas:

- Opportunity: whether the hydrogeomorphic setting suggests plausible recharge concentration or runoff capture potential.
- Feasibility: whether broad physical conditions make practical recharge more plausible.
- Constraint: whether steep slopes, drainage wetness, ridges, or built-up settings reduce suitability.
- Confidence: how cautiously results should be interpreted.
- Intervention setting: which broad MAR typology best matches the terrain setting.

## Public wrapper flow

1. User opens the landing page.
2. The page embeds the live Earth Engine app in an iframe.
3. If embedding is blocked, the user can launch the full app directly.
4. The surrounding sections explain what the outputs mean and how to interpret them responsibly.

## Repository structure

- index.html: static wrapper page.
- site/: favicon and visual assets.
- docs/: supporting reference material.
- README.md and METHODOLOGY.md: core written documentation.

## Design principle

The wrapper keeps the app easy to find and easy to understand without duplicating or misrepresenting the underlying science.
