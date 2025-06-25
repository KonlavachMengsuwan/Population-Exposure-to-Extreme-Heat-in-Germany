**Title: Population Exposure to Extreme Heat in Germany on 13 August 2024 using MODIS LST and WorldPop Data**

---

### 🎓 Introduction

Heatwaves are increasingly frequent across Europe due to climate change, posing serious risks to public health. Identifying where high population density coincides with extreme land surface temperatures (LST) can support planning for urban cooling and heat mitigation.

---

### ❓ Research Gap

While various studies highlight temperature extremes or population vulnerability separately, few combine high-resolution global datasets to quantify how many people are exposed to extreme LST at national scale, and where.

---

### 💡 Objective

To estimate and map the number of people in Germany exposed to land surface temperatures exceeding **35 °C** on the **hottest day of 2024**, identified as **13 August 2024**.

---

### ⚖️ Methods

* Use **MODIS Terra LST (MOD11A1)** daily 1 km product to extract surface temperatures on 13 August 2024.
* Use **WorldPop 2020** population density raster (\~100 m resolution) for Germany.
* Overlay LST and population datasets to compute how many people were located in areas with LST > 35 °C.
* Visualize and export results using **Google Earth Engine**.

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
* Analysis by: \[Your Name or GitHub Handle]
