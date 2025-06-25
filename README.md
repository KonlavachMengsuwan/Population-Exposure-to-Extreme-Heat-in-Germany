# Title: Population Exposure to Extreme Heat in Germany on 13 August 2024 using MODIS LST and WorldPop Data**

---

### 🎓 Introduction

Heatwaves have become more frequent, intense, and prolonged across Europe, posing significant threats to human health, urban infrastructure, and ecosystem stability. Germany, like many other countries, has experienced record-breaking temperatures in recent years, prompting increased interest in spatially explicit assessments of heat exposure. Land Surface Temperature (LST) retrieved from satellite sensors such as MODIS offers a valuable proxy for understanding spatial heat patterns at regional and local scales. When combined with gridded population datasets like WorldPop, it becomes possible to assess not only where extreme temperatures occur but also how many people are affected. Such integrated analyses are essential for informing public health responses, urban resilience planning, and climate adaptation strategies.

---

### ❓ Research Gap

Despite the availability of high-resolution satellite-derived temperature data and global population grids, relatively few studies have systematically combined these datasets to quantify population exposure to extreme surface temperatures at a national scale. Existing assessments often rely on coarse climate models or aggregate census data, which may obscure local-level variations in heat exposure, especially in urban environments. Moreover, there is a lack of real-time or near-real-time analytical workflows that enable timely assessments during or immediately after extreme heat events. Addressing this gap can enhance preparedness and response strategies during future heatwaves.

---

### 💡 Objective

This study aims to:

1. Identify and map areas in Germany experiencing Land Surface Temperatures (LST) greater than 35 °C on the hottest day of 2024 (13 August 2024) using MODIS satellite data.
2. Quantify the number of people exposed to these high temperatures by overlaying the thermal map with high-resolution population density data from WorldPop.
3. Develop a reproducible workflow using Google Earth Engine to support timely population exposure analysis for extreme heat events.

---

### ✍️ GEE Code

```javascript
// Germany boundary
var germany = ee.FeatureCollection("FAO/GAUL/2015/level0")
                .filter(ee.Filter.eq('ADM0_NAME', 'Germany'));

// MODIS LST on 13 August 2024
var lst = ee.ImageCollection('MODIS/061/MOD11A1')
            .filterDate('2024-08-13', '2024-08-14')
            .select('LST_Day_1km')
            .first()
            .multiply(0.02).subtract(273.15).rename('LST_C');

// WorldPop Germany 2020 (can be uploaded as asset or used from public source)
var pop = ee.Image("WorldPop/GP/100m/pop/DEU_2020").rename('POP');

// Identify areas with LST > 35 °C
var hotArea = lst.gt(35).selfMask();

// Mask population with hot areas
var exposedPop = pop.updateMask(hotArea);

// Compute total exposed population
var exposedTotal = exposedPop.reduceRegion({
  reducer: ee.Reducer.sum(),
  geometry: germany.geometry(),
  scale: 100,
  maxPixels: 1e13
});

print('People exposed to LST > 35 °C:', exposedTotal);

// Export GeoTIFF of exposed population
Export.image.toDrive({
  image: exposedPop,
  description: 'Exposed_Population_GT35C_Germany_20240813',
  folder: 'GEE_exports',
  fileNamePrefix: 'ExposedPop_GT35C_2024',
  region: germany.geometry(),
  scale: 100,
  maxPixels: 1e13,
  crs: 'EPSG:4326'
});
```

---

### 🔻 Plot: Population Density (WorldPop 2020)

* Source: \[WorldPop/GP/100m/pop/DEU\_2020]
* Resolution: 100 m
* Units: People per pixel
* Visualized using a purple gradient where higher density = darker

---

### 🔻 Plot: MODIS LST (13 August 2024)

* Source: MODIS MOD11A1 (daily)
* Resolution: 1 km
* Units: Degrees Celsius (converted from Kelvin)
* Temperature range: approx. 20 – 50 °C
* Visualized using blue-green-yellow-red palette

---

### 🔻 Plot: Exposed Population (LST > 35 °C)

* Computed by masking WorldPop where LST > 35 °C
* Units: People per pixel
* Visualized in yellow-red gradient
* Exported as GeoTIFF

---

### 🔹 Clarification of Temperature Data

* **LST Source:** MODIS Terra (MOD11A1)
* **Date:** 13 August 2024
* **Unit Conversion:** MODIS LST in Kelvin, scaled (multiply by 0.02, subtract 273.15)
* **Threshold used:** 35 °C (daytime land surface temperature)

---

### 🔹 Clarification of Population Dataset

* **Source:** WorldPop 2020 (Germany)
* **Layer:** `WorldPop/GP/100m/pop/DEU_2020`
* **Resolution:** 100 meters
* **Value:** Estimated people per pixel

---

### 🔹 Clarification of LST Dataset

* **Source:** MODIS MOD11A1 v6.1
* **Layer:** `LST_Day_1km`
* **Resolution:** 1 km
* **Temporal Coverage:** Daily
* **Acquisition Date:** 13 August 2024

---

### 🌍 License and Credits

* MODIS: NASA LP DAAC
* WorldPop: WorldPop project, University of Southampton
* Admin Boundaries: FAO GAUL
* Analysis by: Konlavach Mengsuwan
