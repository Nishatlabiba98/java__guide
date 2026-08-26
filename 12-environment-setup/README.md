# 12 — Environment Setup

How TapRoot's environments differ and how to work with each one.

- **Local/dev** — running the app on your own machine, local config, hitting local or shared dev dependencies
- **SBX (sandbox)** — shared testing environment, how it differs from local, who else uses it
- **UAT** — user acceptance testing, what gates a change getting promoted here
- **PROD** — production, access restrictions, deploy process, why nothing skips UAT

Ties directly into [14 — Spring](../14-spring/README.md)'s config/profiles section — each environment above typically maps to a Spring profile (`application-dev.properties`, `application-uat.properties`, etc.) that swaps things like DB connection strings and feature flags without changing code.
