# i.sen2bathy.opencl

GRASS GIS addon for Satellite-Derived Bathymetry (SDB) from Sentinel-2 imagery using the methods described in Saeed et al. (2026) "Satellite-derived bathymetry of Arabian Gulf ports under leakage-free validation: a multi-site benchmark of empirical, machine-learning and deep-learning models against 285,000 multibeam soundings, with an outlook for UAE waters" (arXiv:2610.33609v1).

## Description

This addon implements empirical bathymetry retrieval methods (Stumpf log-ratio and Lyzenga linear transform) as described in the paper, with support for:

- Data acquisition via `r.in.sentinel` (Sentinel-2 L2A from Microsoft Planetary Computer)
- Stumpf ratio method: $p_{ij} = \ln(n\rho_i)/\ln(n\rho_j)$ with linear regression to depth
- Lyzenga transform: $X_i = \ln(\rho_i - \rho_{\infty,i})$ with multi-band linear regression
- Hybrid approach combining both methods
- Optional calibration with known depth points
- Cloud masking support

## Requirements

- GRASS GIS 8.x
- Python 3.x
- `r.in.sentinel` addon (for data download)
- Access to Microsoft Planetary Computer STAC API or Google Earth Engine

## Installation

Copy the addon to your GRASS GIS addons directory:

```bash
cp i.sen2bathy ~/.grass8/addons/scripts/
chmod +x ~/.grass8/addons/scripts/i.sen2bathy
```

Or symlink it to your GRASS installation's addon scripts directory.

## Usage

```bash
i.sen2bathy start=2022-01-01 end=2022-03-31 method=stumpf \
  calibration=depth_points column=depth output=bathymetry \
  clouds=20 -c
```

### Parameters

- `start` (required): Start date (YYYY-MM-DD)
- `end` (required): End date (YYYY-MM-DD)
- `clouds`: Maximum cloud cover percentage (default: 20)
- `method`: Bathymetry method - `stumpf`, `lyzenga`, or `hybrid` (default: `stumpf`)
- `calibration`: Vector points map with known depths for calibration
- `column`: Column with depth values in calibration points
- `output` (required): Output bathymetry raster name
- `prefix`: Prefix for temporary/imported raster maps (default: `s2bathy`)
- `resolution`: Output resolution in meters (default: 10)

### Flags

- `-c`: Use cloud masking
- `-f`: Force re-download/import even if data exists

## Methods

### Stumpf Ratio (default)
Based on Stumpf et al. (2003), uses the ratio of log-transformed reflectances:
$$
p_{23} = \frac{\ln(n\rho_{B02})}{\ln(n\rho_{B03})}, \quad n=1000
$$
Depth is estimated as $d = m_0 + m_1 p_{23}$ (linear regression with calibration).

### Lyzenga Transform
Based on Lyzenga (1978, 1985), uses log-transformed deep-water corrected reflectances:
$$
X_i = \ln(\rho_i - \rho_{\infty,i})
$$
Multiple bands are combined linearly to estimate depth.

### Hybrid
Weighted combination of Stumpf and Lyzenga results.

## Notes

This implementation follows the methodology from Saeed et al. (2026). For optimal results:
- Use images from clear water seasons (winter in Arabian Gulf)
- Provide calibration points from recent bathymetric surveys
- Apply tidal corrections if needed
- Consider water turbidity and optical depth limits (~11m in turbid waters as per paper)

## References

Saeed, M. et al. (2026). Satellite-derived bathymetry of Arabian Gulf ports under leakage-free validation: a multi-site benchmark of empirical, machine-learning and deep-learning models against 285,000 multibeam soundings, with an outlook for UAE waters. arXiv:2610.33609v1.
