# How to Use Sentinel-1 RTC Products for Flood Mapping?

## Introduction

This document provides a detailed analysis of Sentinel-1 RTC product files and their application in flood mapping, using the example product `S1A_IW_20240526T095829_SVP_RTC10_G_gdufed_FC26`. The filename components indicate this is:

- `S1A`: Sentinel-1A satellite
- `IW`: Interferometric Wide swath mode
- `20240526T095829`: Acquisition date/time (May 26, 2024, 09:58:29 UTC)
- `SVP`: Single polarization VV, Precise orbit
- `RTC10`: Radiometric Terrain Correction with 10m pixel spacing
- `G`: GAMMA software used for processing
- `gdufed`: Processing parameters
  - `g`: gamma0 radiometry (accounts complex scattering geometry of urban structures)
  - `d`: decibel scale (better discrimination of low backscatter values)
  - `u`: unmasked (no layover/shadow mask, see `_ls_map.tif`)
  - `f`: speckle filtered (reduces noise while maintaining edges, 7x7 Lee filter)
  - `e`: entire area (not clipped)
  - `d`: dead reckoning (no DEM matching, consistent geolocation)
- `FC26`: Unique product ID

## Core Data Files

### 1. `_VV.tif`

- **Content**: Primary radiometric terrain corrected SAR data in VV polarization
- **Format**: 32-bit floating-point GeoTIFF
- **Processing Role**: Primary dataset for flood detection
- **Application Notes**:
  - Values are in decibel scale, making water detection more straightforward
  - Dark pixels (low backscatter values, typically < -20 dB) often indicate water
  - Compare with pre-flood image using ratio or difference analysis

### 2. `_ls_map.tif`

- **Content**: Layover/shadow mask
- **Format**: 8-bit unsigned integer GeoTIFF
- **Processing Role**: Quality control and masking
- **Application Notes**:
  - Essential for excluding unreliable pixels affected by geometric distortions
  - Pay special attention to urban areas where layover is common
  - Create a binary mask excluding values indicating layover (4) or shadow (16)

### 3. `_inc_map.tif`

- **Content**: Local incidence angle map
- **Format**: 32-bit floating-point GeoTIFF
- **Processing Role**: Backscatter normalization and interpretation
- **Application Notes**:
  - Use to normalize backscatter values in urban areas
  - Critical for reducing false positives in flood detection
  - Consider masking extreme angles (< 20° or > 65°)

### 4. `_dem.tif`

- **Content**: Digital Elevation Model used in processing
- **Format**: 16-bit signed integer GeoTIFF
- **Processing Role**: Topographic analysis and validation
- **Application Notes**:
  - Use for slope calculation to mask unlikely flood areas
  - Validate flood extent against topographic constraints
  - Consider in urban drainage pattern analysis

## Supporting Files

### 5. Visualization Files

- **`.png` and `.kmz`**: Quick-look images
  - Useful for rapid assessment and preliminary flood extent estimation
  - Enable quick sharing with emergency responders
  - `.png.aux.xml` contains georeference information

### 6. Vector Data

- **`_shape.*` files**: Data extent shapefile
  - Use to clip analysis to valid data areas
  - Helpful in mosaicking multiple scenes
  - Essential for defining processing boundaries

### 7. Metadata and Documentation

- **`.README.md.txt`**: Detailed product documentation
- **`.log`**: Processing history
- **`*tif.xml`**: ArcGIS metadata
  - Important for maintaining data lineage
  - Contains processing parameters
  - Useful for reproducibility

## Processing Workflow for Flood Mapping

1. **Pre-processing**

   ```plaintext
   1. Load VV.tif and create initial water mask:
      water_mask = VV.tif < -20  # Initial threshold in dB

   2. Apply quality masks:
      - Exclude layover/shadow areas using ls_map.tif
      - Apply incidence angle corrections using inc_map.tif
      - Consider topographic constraints using dem.tif

   3. Refine water mask:
      - Apply object-based analysis
      - Remove isolated pixels
      - Apply morphological operations
   ```

2. **Change Detection**

   ```plaintext
   1. Coregister pre- and post-flood images
   2. Calculate change ratio:
      change_ratio = 10 * log10(post_flood / pre_flood)
   3. Apply threshold to change_ratio for flood classification
   ```

## Quality Assessment Guidelines

1. **Geometric Accuracy**

   - Check corner coordinates against reference data
   - Verify alignment with pre-flood image
   - Validate against known ground control points

2. **Radiometric Quality**

   - Examine calibration using known stable targets
   - Check for artifacts or banding
   - Verify dynamic range in urban areas

3. **Flood Mapping Accuracy**
   - Validate against optical data (when available)
   - Check consistency with DEM and hydrological networks
   - Verify urban flood patterns against known topography

## Best Practices for Urban Flood Mapping

1. **Urban-Specific Considerations**

   - Use incidence angle map to account for building shadows
   - Apply stricter threshold in areas affected by layover
   - Consider DEM-derived slope in urban drainage patterns

2. **Validation Strategy**

   - Cross-reference with emergency response reports
   - Compare with optical data when available
   - Validate against ground observations

3. **Output Products**
   - Generate binary flood extent maps
   - Create flood probability maps
   - Produce change detection maps
   - Generate damage assessment reports
