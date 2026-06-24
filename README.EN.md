# Korea Immigration Transparency Dashboard

[한국어](README.md) · **English** · [日本語](README.ja.md)

A single-file HTML dashboard that processes the Korean Ministry of Justice (MOJ) immigration statistics into a map of the **service area** and estimated **visa processing time** for the country's 17 immigration offices. The goal is to let a foreign resident see, on one screen, which office covers their district, where to submit documents (main office vs. branch office), and roughly how long it takes.

Processing times after a visa extension or change are not published, so applicants have little visibility into the status. This dashboard estimates those times from public statistics and shows the jurisdiction and submission points alongside.

## Features

- **Processing-time estimator** — choose office, visa type, application month → estimated days, expected completion date (ETA), 95% confidence interval
- **Service-area map (17 offices)** — 251 districts merged into office territories by official jurisdiction, colored by estimated wait. Hovering an office shows its districts, submission points (main/branch offices), and processing stats. A grid-based **Tile view** is provided for users who find the map hard to read
- **Regional inflow/outflow** — provincial monthly net change (new arrivals − departures) choropleth
- **Volume trend** — 52 months of actuals (2022.01–2026.04) + 6-month OLS forecast
- **By-visa analysis · visa mix · seasonality · office ranking**
- Korean/English/Japanese · dark/light (system-aware) · responsive (mobile/tablet/desktop)

## Data

| Item | Source | As of |
|------|--------|-------|
| Registered foreigners by office | Monthly report, "Registered foreigners by district" | Apr 2026 (1,623,256 nationwide) |
| Office jurisdiction | Immigration office service areas (MOJ official) | district level |
| Regional inflow/outflow | "Foreign inflow/outflow statistics" (MOJ Foreigner Big-Data Team) | May 2026 |
| Volume trend & seasonality | Immigration policy monthly report | 2022.01–2026.04 |

All data from MOJ public releases (Korea Open Government License, Type 1).

## Jurisdiction mapping

The 17 office territories are derived from the **official immigration-office service areas** published by the MOJ.

1. Extract registered foreigners per district from "Registered foreigners by district" (Apr 2026, 255 rows)
2. Assign each district to one of 17 offices per the official table. Branch offices roll up to their parent, but **cross-assignments** follow the official record — e.g. Gyeongju (Gyeongbuk) → Ulsan; Gimhae·Miryang·Yangsan (Gyeongnam) → Busan; Cheorwon (Gangwon) → Yangju; Gimpo (Gyeonggi) → Incheon; Anyang·Gwangmyeong (Gyeonggi) → Seoul
3. Verify **zero unmapped districts and a matching total** (1,623,256)
4. Dissolve district polygons by office → Mercator projection → bake into lightweight SVG paths

This corrected about 10 cross-jurisdiction errors in an earlier web-research-based mapping. The office tooltip also lists the per-branch submission jurisdiction, so a resident can tell where to submit documents versus which office processes them.

## Forecast model

Monthly processing volume is modeled with an OLS trend plus seasonal decomposition. R² 0.958, MAE 5.1%, RMSE 16,048, trend +2,802/month, August peak (+44,948).

Per-office, per-visa estimate:

```
days = base_days × load^0.5
load = (district registered foreigners ÷ capacity tier) ÷ national mean
capacity tier = main bureau 1.30 / office 1.00 / branch 0.55   (staffing not public; tier-approximated)
```

Measured values (REAL) and estimates (EST) are distinguished on screen. The model's only assumption is the office capacity tier; the other inputs (registered foreigners, jurisdiction, inflow/outflow) are sourced and measured. If staffing or throughput figures become public, this assumption can be replaced with measured data.

## Architecture

- Single HTML file. No build step, no server — open it in a browser
- Map: self-contained SVG. District boundaries are merged into office territories and projected (Mercator), with no d3/topojson/external fetch. The service-area map and the inflow/outflow map share one viewBox/coordinate system
- Fonts: Korean (Nanum Gothic) and Japanese (Noto Sans JP) subset to only the glyphs used → woff2 → base64-embedded (~248KB total). No font CDN, works offline. Latin/numerals use system fonts
- External dependency: one chart library (Chart.js)

## Notes

- "Staying foreigners" (~2.87M) and "registered foreigners" (~1.62M) are different populations; every figure is labeled with which one it uses.
- Figures such as the illegal-stay rate, which could fuel discrimination against undocumented residents, are not included.
- The dashboard is estimate-based and has no legal force. It is not official statistics and is an independent academic project unaffiliated with the MOJ.

## Run / deploy (GitHub Pages)

Place `index.html` at the repository root and set **Settings → Pages → Source** to the `main` branch root; it will be served at `https://<user>.github.io/<repo>/`.

```
immigration/
├── index.html        # dashboard (single file, fonts & map embedded)
├── README.md         # 한국어
├── README.en.md      # English
└── README.ja.md      # 日本語
```

On data refresh: re-aggregate jurisdiction from the new district CSV → re-verify zero unmapped / matching total → re-bake the map. Re-subset the fonts if the on-screen text changes.

## License

Data follows the Korea Open Government License Type 1 (attribution). Embedded fonts: Nanum Gothic (OFL), Noto Sans JP (OFL). Independent academic/research project.
