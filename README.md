# FloodSight Lagos Prototype

**FloodSight** is an open-data flood intelligence platform that combines geospatial risk modelling, population exposure analysis, and rainfall-triggered alerts to support climate adaptation and early warning in flood-prone urban communities.

This pilot focuses on selected Lagos State LGAs: **Eti-Osa, Lagos Island, and Kosofe**.

## Current project status

FloodSight is currently at **pilot stage**. The Lagos pilot has produced:

- Static flood-risk map at 30 m grid resolution
- High-risk population exposure map
- Demo rainfall alert scenario map
- Real CHIRPS rainfall no-alert map for selected dates
- Population by flood-risk class table
- Population by demo alert level table
- QGIS-based working geospatial model

## Why the methodology is documented

Yes, it is necessary to document how the maps were calculated. The documentation is important because it shows:

1. **Transparency** - judges, partners, and users can understand how FloodSight works.
2. **Reproducibility** - another analyst can rebuild the pilot from open data.
3. **Trust** - the risk map is not just a picture; it is based on defined data layers and scoring rules.
4. **Upgrade path** - the static model can later be improved with calibration, validation, and real-time rainfall.
5. **Operational readiness** - emergency teams need to know what each map means and how alerts are generated.

## How the map becomes dynamic after deployment

FloodSight has two parts:

### 1. Static flood susceptibility baseline

This is not recalculated every hour. It is updated periodically because the underlying factors change slowly:

- elevation
- slope
- flow accumulation
- distance to water
- land cover
- population exposure
- roads, buildings, and water bodies

The static baseline produces the fields:

- `flood_score`
- `risk_class`

### 2. Dynamic rainfall alert layer

This is what changes frequently after deployment. The web application will fetch rainfall data, calculate rainfall totals, and update alert levels.

Dynamic fields include:

- `rain_24h`
- `rain_72h`
- `alert_level`

The deployed application will update alerts through this loop:

1. Download or fetch latest rainfall data from GPM IMERG, CHIRPS, or forecast APIs.
2. Process rainfall into 24-hour and 72-hour totals.
3. Sample rainfall values at FloodSight grid-cell centroids.
4. Join rainfall values back to the 30 m grid.
5. Recalculate `alert_level` using the flood-risk class and rainfall thresholds.
6. Publish the updated alert layer to the web dashboard.

In simple terms:

```text
Static risk map + latest rainfall = live Watch/Warning alert map
```

## Open data sources used

FloodSight uses open data only.

| Theme | Dataset |
|---|---|
| Administrative boundaries | Nigeria ADM boundaries |
| Elevation | Copernicus DEM GLO-30 |
| Land cover | ESA WorldCover |
| Population | WorldPop |
| Roads, buildings, water bodies | OpenStreetMap |
| Rainfall | CHIRPS daily rainfall; future upgrade to GPM IMERG |

Large raw datasets are **not stored in this repository**. They should be downloaded from their original open-data providers.

## Model workflow

1. Organise raw datasets into the project folder.
2. Extract Lagos State and pilot LGA boundaries.
3. Reproject working layers to UTM Zone 31N.
4. Clip DEM, population, land cover, and other rasters to the pilot area.
5. Create a 30 m modelling grid.
6. Derive terrain features:
   - elevation
   - slope
   - filled DEM
   - flow accumulation
7. Extract OSM roads, buildings, and water bodies.
8. Create distance-to-water raster.
9. Add raster and vector values to each grid cell.
10. Calculate normalized risk factors.
11. Calculate `flood_score`.
12. Classify cells into Low, Moderate, High, and Very High risk.
13. Add rainfall fields and calculate alert levels.
14. Export maps, tables, and methodology notes.

## Flood score factors

| Factor | Risk interpretation |
|---|---|
| Low elevation | Higher flood susceptibility |
| Low slope | Higher ponding and surface-water accumulation risk |
| High flow accumulation | More likely water concentration path |
| Short distance to water | Higher exposure to water bodies and drainage overflow |
| Flood-sensitive land cover | Built-up, wetland, water-adjacent areas can increase susceptibility |
| Population exposure | Indicates people potentially exposed in risk zones |

## Current scoring formula

```text
flood_score =
  0.25 * risk_elev +
  0.15 * risk_slope +
  0.20 * risk_flow +
  0.15 * risk_waterdist +
  0.15 * risk_landcover +
  0.10 * risk_pop
```

Risk classes:

| Score range | Class |
|---|---|
| < 0.25 | Low |
| 0.25 - 0.50 | Moderate |
| 0.50 - 0.75 | High |
| >= 0.75 | Very High |

## Rainfall alert logic

Real rainfall fields:

- `rain_24h`
- `rain_72h`

Alert field:

- `alert_level`

Current rule:

```text
Warning:
High or Very High risk AND rain_24h >= 100 mm OR rain_72h >= 150 mm

Watch:
High or Very High risk AND rain_24h >= 50 mm OR rain_72h >= 100 mm

Watch:
Moderate risk AND rain_24h >= 100 mm OR rain_72h >= 150 mm

Otherwise:
No Alert
```

A stress-test rainfall scenario was also created to demonstrate the Watch/Warning alert engine under heavier rainfall conditions.

## Repository structure

```text
floodsight-lagos-pilot/
  README.md
  docs/
    floodsight_methodology_note.pdf
  outputs/
    maps/
      floodsight_lagos_pilot_risk_map.png
      floodsight_high_risk_population_exposure_map.png
      floodsight_demo_alert_scenario_map.png
      floodsight_real_chirps_no_alert_map.png
    tables/
      population_by_risk_class.csv
      population_by_demo_alert_level.csv
  qgis/
    FloodSight_Lagos_Pilot_Final.qgz
```

## Limitations

This is a first open-data prototype. It is not yet a calibrated hydrodynamic flood-depth model. The current version provides flood susceptibility, exposure intelligence, and rainfall-triggered alert priority.

## Next development phase

1. Integrate near-real-time GPM IMERG rainfall.
2. Automate daily or hourly rainfall ingestion.
3. Publish map tiles and alert layers to a web dashboard.
4. Add historical flood-event validation.
5. Add community reporting and field verification.
6. Expand to all Lagos LGAs and later other Nigerian coastal cities.

## Suggested application positioning

FloodSight is a pilot-stage climate adaptation and early-warning platform that turns open geospatial data into actionable flood-risk intelligence for vulnerable urban communities.
