# Heatmapper: Digital Transformation with Third-Party APIs

A talk for QAware Engineering Camp 26 (EC26) by Charlie Harding.

## About

This talk traces the evolution of [Heatmapper](https://github.com/c-harding/heatmapper), a personal web application for visualizing Strava activities on interactive maps. What started as a digital continuation of hand-drawn routes on a paper map grew into a practical case study covering API integration, data management, GDPR compliance, cost optimization, and technical trade-offs.

Through concrete challenges — from incomplete APIs and rate limits to new compliance requirements and browser storage limits — the talk shows how robust, future-proof software can emerge from a small hobby project, and how those learnings transfer directly to enterprise digital transformation.

## Topics

1. **Motivation** — From analog paper maps to a digital solution
2. **From files to APIs** — GPX exports, Strava API, encoded polylines, GDPR as an architectural driver
3. **Visualization** — Slippy maps to vector tiles, Mapbox, heatmap rendering, globe mode & 3D terrain
4. **Data management** — Performance vs. cost vs. API limits, LocalStorage, delta updates, IndexedDB migration
5. **Governance & compliance** — Strava API review, branding requirements, webhooks, privacy without personal data in the backend

## Getting Started

- `pnpm install`
- `pnpm dev`
- Visit <http://localhost:3030>

Learn more about Slidev at the [documentation](https://sli.dev/).
