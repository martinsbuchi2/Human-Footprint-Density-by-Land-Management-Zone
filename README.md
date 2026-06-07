# Human Footprint Density by Land Management Zone
### Project Documentation

**Project file:** `HFI_Density_by_LMZ.qgz`
**Date completed:** May 13, 2026
**CRS:** EPSG:3400 - NAD83 / Alberta 10-TM Forest
**Analyst tool:** QGIS (MCP-integrated workflow)

---

## 1. Project Overview

This project quantifies the spatial density of other-vegetated-surface human footprint (HFI) across Alberta's Green and White land management zones using 2022 Human Footprint Inventory data. The primary output is a choropleth map of footprint density expressed as a percentage of each zone's total area, supported by a per-zone summary table. The analysis reveals the relative distribution of vegetated-surface human pressure between the forested Green Area in the north and the agricultural White Area in the south.

---

## 2. Input Layers

| Layer | File | CRS | Features | Type |
|---|---|---|---|---|
| Land Management Zones | `Alberta_Green_and_White_Areas_` | EPSG:3857 (source) | 39 | Polygon |
| Provincial Boundary | `Provincial_Boundary_` | EPSG:3857 | 1 | Polygon |
| HFI 2022 - Other Vegetated Surface | `o11_OtherVegetatedSurface_HFI_2022_` | EPSG:3400 | 11,360 | Polygon |

The HFI layer represents the o11 - Other Vegetated Surface footprint class from Alberta's 2022 Human Footprint Inventory. The Green and White Areas layer encodes Alberta's two primary land management regimes: the Green Area (Crown forested land in the north and mountains, governed under the Public Lands Act and Forest Management Agreements) and the White Area (settled and agricultural land in the central and southern portions of the province).

---

## 3. Methodology

### 3.1 CRS Alignment
The Green and White Areas layer was reprojected from EPSG:3857 to EPSG:3400 to match the HFI layer. EPSG:3400 (NAD83 / Alberta 10-TM Forest) is the standard projected CRS for Alberta provincial spatial analysis and preserves area calculations accurately at the provincial scale.

### 3.2 Geometry Repair
Both the reprojected zones layer and the HFI layer contained invalid geometries. Fix Geometries (native:fixgeometries) was applied to both before any spatial operations. Skipping this step causes immediate failures in the join and field calculator operations.

### 3.3 Area Field Calculation
A field `area_ha` was added to the fixed HFI layer using the expression `$area / 10000`. This converts the geometry area from square metres (native unit for EPSG:3400) to hectares, which is the standard reporting unit for land area in Alberta resource management contexts.

### 3.4 Spatial Join
Join Attributes by Location (Summary) was run with the fixed Green and White zones as the base layer and the area-attributed HFI layer as the join. Predicate: intersects. Summary statistics requested on area_ha: count, sum, mean. This produced three new fields attached to each zone polygon: area_ha_count, area_ha_sum, and area_ha_mean.

### 3.5 Derived Fields
Three additional fields were calculated on the joined output:

| Field | Expression | Description |
|---|---|---|
| zone_area_ha | Shape_Area / 10000 | Total area of each zone polygon in hectares |
| footprint_ha | if(area_ha_sum IS NULL, 0, area_ha_sum) | HFI footprint area in hectares; NULL replaced with 0 for unmatched zones |
| density_pct | (footprint_ha / zone_area_ha) * 100 | Footprint as a percentage of zone area |

### 3.6 Symbology - Manual Graduated Classification
Quantile classification was explicitly avoided. With 26 of 39 zones at zero density and one outlier zone at 3.748%, quantile collapses most classes to zero and produces an uninterpretable map. A manual five-class scheme was applied on density_pct using the YlOrRd colour ramp:

| Class | Range (%) | Hex Colour | Interpretation |
|---|---|---|---|
| No Footprint | 0 to 0.0001 | #ffffcc | No HFI coverage intersects the zone |
| Very Low Density | 0.0001 to 0.01 | #fed976 | Negligible vegetated surface footprint |
| Low Density | 0.01 to 0.1 | #fd8d3c | Sparse, localized footprint |
| Moderate Density | 0.1 to 0.5 | #e31a1c | Noticeable footprint, likely fragmented patches |
| High Density | 0.5 to 4.0 | #800026 | Concentrated footprint, highest pressure zone |

Polygon stroke: white, 0.2 pt. Provincial boundary overlay: hollow fill, dark border at 0.8 pt.

---

## 4. Results

### 4.1 Zone-Level Summary

| Zone | Zone Area (ha) | HFI Footprint (ha) | Density (%) | HFI Patches |
|---|---|---|---|---|
| Green Area | 115,572,055.65 | 11,521.79 | 0.0100 | 1,846 |
| White Area | 70,168,593.59 | 61,542.76 | 0.0877 | 9,322 |
| Total | 185,740,649.24 | 73,064.55 | 0.0394 | 11,168 |

### 4.2 Class Distribution (39 zone polygons)

| Density Class | Zone Count |
|---|---|
| No Footprint | 26 |
| Very Low Density | 3 |
| Low Density | 6 |
| Moderate Density | 3 |
| High Density | 1 |

### 4.3 Key Findings

The White Area carries approximately 8.7 times the footprint density of the Green Area (0.0877% vs 0.0100%), despite being 39% smaller in total land area. The White Area accounts for 84.2% of all HFI footprint in absolute terms while covering only 37.8% of Alberta's land base represented in the dataset. The single High Density zone, with a density of 3.748%, accounts for a disproportionately large share of the White Area footprint total and warrants closer examination in any follow-on risk or reclamation screening.

---

## 5. Output Files

| File | Description | Size |
|---|---|---|
| HFI_Density_by_LMZ.qgz | Final QGIS project with all layers and symbology | 16.7 KB |
| hfi_density_final.gpkg | Primary output - zones with all derived density fields | 1,832 KB |
| hfi_with_area.gpkg | Fixed HFI layer with area_ha field | 15,680 KB |
| hfi_fixed.gpkg | Geometry-repaired HFI layer (intermediate) | 15,552 KB |
| green_white_fixed.gpkg | Geometry-repaired zones in EPSG:3400 (intermediate) | 1,832 KB |
| green_white_areas_3400.gpkg | Reprojected zones before repair (intermediate) | 1,832 KB |
| hfi_density_by_zone.gpkg | Raw spatial join output before field enrichment (intermediate) | 1,832 KB |

---

## 6. Key Attribute Reference - hfi_density_final.gpkg

| Field | Type | Description |
|---|---|---|
| GWA_NAME | String | Zone name (Green Area / White Area) |
| GWA_CODE | String | Zone code (GLC_G / GLC_W) |
| Shape_Area | Double | Zone polygon area in square metres |
| area_ha_count | Integer | Count of HFI patches intersecting the zone |
| area_ha_sum | Double | Total HFI footprint area in hectares |
| area_ha_mean | Double | Mean HFI patch size in hectares |
| zone_area_ha | Double | Zone total area in hectares (derived) |
| footprint_ha | Double | Null-safe HFI footprint in hectares (derived) |
| density_pct | Double | HFI footprint as % of zone area (derived) |

---

## 7. Reproduction Notes

- Both input layers require geometry repair before spatial operations. This is not optional.
- Do not use quantile classification on density_pct. The distribution is heavily zero-inflated and right-skewed; manual breaks are required for a meaningful choropleth.
- area_ha_sum returns NULL for zones with no intersecting HFI features. The footprint_ha field handles this with a null-safe expression. Use footprint_ha rather than area_ha_sum for any downstream calculations.
- The 39 zone polygons represent individual sub-polygons of the Green and White Areas, not 39 distinct management units. Aggregate by GWA_NAME or GWA_CODE for province-wide Green vs White comparisons.
- This analysis can serve as an environmental context overlay for abandoned well cluster outputs by intersecting cluster hull polygons with hfi_density_final.gpkg to flag clusters in high-density footprint zones.

---

## 8. Integration with Abandoned Well Workflow

This layer is directly compatible with the standard hotspot/density workflow. Intersect cluster_hulls.gpkg with hfi_density_final.gpkg on GWA_NAME or density_pct to flag abandoned well clusters that co-occur with existing vegetated-surface footprint. Clusters in the Green Area with even low-density footprint are of higher ecological concern given the forested land management context and stricter reclamation requirements under provincial policy.
