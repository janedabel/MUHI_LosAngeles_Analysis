# MUHI_LosAngeles_Analysis

1. NDVI Calculation
# NDVI = (NIR - Red) / (NIR + Red)
import numpy as np

def calculate_ndvi(nir_band, red_band):
    ndvi = (nir_band - red_band) / (nir_band + red_band + 1e-10)
    return ndvi



2. Proportion of Vegetation (PV) and Emissivity
def calculate_pv(ndvi, ndvi_min=0.2, ndvi_max=0.8):
    pv = ((ndvi - ndvi_min) / (ndvi_max - ndvi_min)) ** 2
    return np.clip(pv, 0, 1)

def calculate_emissivity(pv):
    return 0.004 * pv + 0.986


3. Brightness Temperature (BT)
def calculate_bt(radiance, K1=774.89, K2=1321.08):
    return K2 / np.log((K1 / (radiance + 1e-10)) + 1) - 273.15


4. Land Surface Temperature (LST)
def calculate_lst(bt, emissivity, wavelength=10.8e-6, rho=1.438e-2):
    return bt / (1 + (wavelength * bt / rho) * np.log(emissivity))


5. MUHI Thresholds
def get_canopy_threshold(lst, ndvi, threshold=0.7):
    return np.max(lst[ndvi >= threshold])

def get_top_2_percent_threshold(lst):
    return np.percentile(lst, 98)



6. Ground-Based Calibration
def calibrate_lst(lst_sat):
    # Derived from field regression: LST_true = 0.95 * LST_sat + 1.2
    return 0.95 * lst_sat + 1.2


7. Land Classification (Conceptual)
# Assuming preprocessed OSM land type raster, reclassify:
land_classes = {
    1: "Vegetated",
    2: "Unvegetated Soil",
    3: "Concrete",
    4: "Asphalt",
    5: "Buildings",
    6: "Water",
    7: "Other"
}

# Apply classification map to raster data array
def reclassify_land_cover(raster_array, reclass_map):
    return np.vectorize(reclass_map.get)(raster_array)
