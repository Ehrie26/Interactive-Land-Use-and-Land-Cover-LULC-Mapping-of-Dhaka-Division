# Interactive-Land-Use-and-Land-Cover-LULC-Mapping-of-Dhaka-Division

This project presents an interactive Land Use and Land Cover (LULC) mapping and change detection application for Dhaka Division, Bangladesh, built using Google Earth Engine (GEE). The application leverages Sentinel-2 surface reflectance imagery and a Random Forest classifier to visualize spatial patterns and land cover dynamics between 2017 and 2023.

The tool is designed to support environmental monitoring, urban growth analysis, and land management by enabling side-by-side comparison of LULC maps and highlighting areas of change over time.

## 🚀 Key Features

- Interactive split-map (slider) comparison of LULC maps  
- Year-by-year visualization for 2017, 2020, and 2023  
- Supervised Random Forest land cover classification  
- Change detection between selected years  
- Intuitive UI controls and dynamic legends  

## 🗺️ Study Area

- **Location:** Dhaka Division, Bangladesh  
- **Boundary Source:** FAO GAUL Level-1 Administrative Boundaries  


## 🛰️ Data Sources

- **Satellite Imagery:** Sentinel-2 Surface Reflectance (COPERNICUS/S2_SR_HARMONIZED)  
- **Spatial Resolution:** 10 m  
- **Temporal Coverage:** 2017, 2020, 2023  

## 🧠 Methodology

1. Annual median composites were generated from Sentinel-2 imagery after filtering by cloud cover.
2. Training samples were prepared using predefined land cover classes.
3. A Random Forest classifier (50 trees) was trained using 2023 data.
4. The trained model was applied to classify LULC for 2017, 2020, and 2023.
5. Change detection was performed by comparing classified maps between selected years, with gap-filling applied to improve visualization consistency.

## 🌱 Land Use / Land Cover Classes

| Class | Description   |
|------|---------------|
| 0    | Settlements   |
| 1    | Vegetation    |
| 2    | Bareland      |
| 3    | Water         |

## 🧭 How the Application Works

- **Left Map:** Displays LULC for an earlier selected year  
- **Right Map:** Displays LULC for a later selected year  
- **Year Selectors:** Choose years independently for each map  
- **Change Detection Toggle:** Highlights areas where land cover has changed  
- **Legends:** Dynamically explain LULC classes and change detection results  


## ▶️ How to Run the Code

1. Open [Google Earth Engine Code Editor](https://code.earthengine.google.com/)
2. Copy and paste the script into a new GEE script
3. Ensure training datasets (Settlements, Vegetation, Bareland, Water) are available in your Assets
4. Click **Run**
5. Use the UI controls to explore LULC patterns and changes


## ⚠️ Notes & Limitations

- Classification accuracy depends on the quality and representativeness of training data.
- Cloud contamination may still affect some areas despite filtering.
- The model is trained on 2023 data and applied to earlier years, which may introduce temporal bias.

## 👩🏽‍💻 Author

**Anuoluwapo Kuye**  
Geospatial Analyst | GIS & Environmental Mapping  
Built using Google Earth Engine and open satellite data.

## 🛠️ Tools & Technologies

- Google Earth Engine (JavaScript API)
- Sentinel-2 Satellite Imagery
- Random Forest Classification
- FAO GAUL Administrative Boundaries

