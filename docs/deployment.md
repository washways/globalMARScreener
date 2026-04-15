# Deployment Guide

This repository is designed to act as a lightweight public wrapper around the published Global MARS Screener Earth Engine app.

## Canonical public links

- Live app: https://washways.projects.earthengine.app/view/globalmarscreener
- Repository: https://github.com/washways/globalMARScreener
- Preferred landing page: https://washways.org/globalMARScreener/
- Optional GitHub Pages fallback: https://washways.github.io/globalMARScreener/

## Option A: Host at a WASHways site path

Recommended for the public-facing version.

1. Upload the repository contents or the static site files to the desired web root path.
2. Serve the root landing page from the folder path for globalMARScreener.
3. Ensure the site preserves the relative links to the site and docs folders.
4. Confirm that the embedded Earth Engine iframe loads publicly.

## Option B: GitHub Pages

This repository is also structured to work on GitHub Pages.

1. Push the repository to GitHub.
2. Open repository settings.
3. Enable Pages.
4. Set the source to deploy from the main branch and the root directory.
5. Confirm that the published site loads the root index page.

## Verification checklist

Before announcing the site publicly, verify the following:

- The landing page loads correctly.
- The Open App button launches the live Earth Engine app.
- The iframe embed is visible in supported browsers.
- README and methodology links open successfully.
- GitHub links point to the correct repository.
- Social sharing links resolve to the intended landing page URL.

## Maintenance

When the public app URL changes, update it in:

- index.html
- README.md
- Any GitHub repository homepage setting

If the Earth Engine source is later added to this repository, update this guide with the exact publish workflow for the script as well.
