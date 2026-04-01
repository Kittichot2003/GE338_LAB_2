# GE338_LAB_2
#  GE338 Lab 2: Spatial Analysis กับ Google Earth Engine
## อำเภอเมืองร้อยเอ็ด จังหวัดร้อยเอ็ด — Land Resource Analysis ฤดูแล้ง

---

##  ข้อมูลทั่วไป

| รายการ | รายละเอียด |
|--------|-----------|
| **วิชา** | ภม.338 Geographic Data Science |
| **Lab** | Lab 2: Spatial Analysis กับ GEE |
| **พื้นที่ศึกษา** | อำเภอเมืองร้อยเอ็ด จังหวัดร้อยเอ็ด (ภาคตะวันออกเฉียงเหนือ) |
| **ดาวเทียม** | Landsat 8 Collection 2 Surface Reflectance |
| **Cloud Project** | `ee-kittichot6692` |
| **ช่วงเวลา** | ฤดูฝน: ส.ค.–ก.ย. 2022 / ฤดูแล้ง: ม.ค.–มี.ค. 2023 |

---

##  โครงสร้างไฟล์

```
GE338-Lab-2/
├── lab2_spatial_analysis.ipynb    ← Notebook หลัก (รันบน Google Colab)
├── README.md                      ← ไฟล์นี้: Methodology + คำตอบคำถาม
└── figures/
    ├── ndvi_histogram_comparison.png  ← เปรียบเทียบ NDVI distribution
    ├── zonal_statistics_chart.png     ← Bar chart Zonal Stats
    └── ndvi_comparison_map.html       ← Interactive map (geemap)
```

---

##  ภารกิจที่ 1: เหตุผลการเลือกพื้นที่

### พื้นที่: อำเภอเมืองร้อยเอ็ด (16°03'N, 103°39'E)

**เหตุผลเชิงภูมิศาสตร์ที่เลือกพื้นที่นี้:**

**1. ลักษณะภูมิประเทศที่หลากหลายในพื้นที่เดียว**
อำเภอเมืองร้อยเอ็ดมีองค์ประกอบครบสำหรับการวิเคราะห์: เขตเมือง (ตลาด ถนน สิ่งปลูกสร้าง), พื้นที่เกษตรกรรม (นาข้าวทุ่งกุลาร้องไห้), แหล่งน้ำถาวร (บึงพลาญชัย แม่น้ำยัง) และพื้นที่กึ่งธรรมชาติ ทำให้สามารถทดสอบ Index หลายประเภทได้ในครั้งเดียว

**2. ฤดูแล้งที่เด่นชัดมากในภาคอีสาน**
ร้อยเอ็ดอยู่ในเขตที่มีฝนตามฤดูกาลชัดเจน ปริมาณฝนปีละ ~1,200 mm โดยเกือบทั้งหมดตกในช่วง พ.ค.–ต.ค. ทำให้ NDVI และ NDWI แสดงความแตกต่างระหว่างฤดูกาลสูงมาก — เหมาะอย่างยิ่งสำหรับโจทย์ที่หน่วยงานต้องการข้อมูลวางแผนฤดูแล้ง

**3. ความสำคัญด้านความมั่นคงทางอาหาร**
ร้อยเอ็ดเป็นจังหวัดที่ผลิตข้าวหอมมะลิส่งออกมากที่สุดแห่งหนึ่งของไทย (พื้นที่ปลูกข้าว ~1.2 ล้านไร่ในจังหวัด) การติดตาม NDVI จึงมีนัยสำคัญด้านนโยบายเกษตร

**4. บึงพลาญชัย — Natural Reference Point**
บึงขนาด ~2.5 km² ใจกลางอำเภอทำหน้าที่เป็น "reference water body" สำหรับ Calibrate ค่า NDWI และ validation ได้โดยไม่ต้องใช้ข้อมูลภาคสนาม

**คำถามวิจัยหลัก:**
- พื้นที่พืชพรรณ (NDVI > 0.3) เปลี่ยนแปลงระหว่างฤดูฝน–ฤดูแล้งเท่าไหร่ในเชิงพื้นที่ (km²)?
- ตำบลไหนมีความเสี่ยงสูงสุดด้านการขาดแคลนน้ำเกษตรในฤดูแล้ง?
- เขตเมืองขยายตัวสัมพันธ์กับการลดลงของพื้นที่เกษตรอย่างไร?

---

## ภารกิจที่ 2: เหตุผลการเลือกดาวเทียมและช่วงเวลา

### ดาวเทียม: Landsat 8 Collection 2 Surface Reflectance

**เหตุผลที่เลือก Landsat 8 (ไม่ใช่ Sentinel-2):**

| เหตุผล | รายละเอียด |
|--------|-----------|
| **Resolution เหมาะสม** | 30m เหมาะกับพื้นที่ระดับอำเภอ (~500 km²) ไม่จำเป็นต้องใช้ 10m ของ Sentinel-2 ซึ่งสิ้นเปลือง compute |
| **Band SWIR** | Band 6 (SWIR1, 1.56–1.66 µm) เหมาะกับ NDWI (Gao 1996) ที่วัดความชื้นพืช Sentinel-2 มี SWIR เช่นกันแต่ 20m |
| **Surface Reflectance (C2)** | ผ่าน Atmospheric Correction (LaSRC algorithm) แล้ว ค่า reflectance เชิงกายภาพ ไม่ใช่ TOA |
| **Archive ยาว** | ข้อมูลตั้งแต่ 2013 เหมาะกับการวิเคราะห์แนวโน้มระยะยาวในอนาคต |

**เหตุผลที่ไม่เลือก MODIS:**
MODIS (250–500m) resolution หยาบเกินไปสำหรับอำเภอเมืองที่มีความหลากหลายของ land cover สูง

### ช่วงเวลาและการกรองเมฆ

**ฤดูฝน: 1 สิงหาคม – 30 กันยายน 2022**
- ข้าวอยู่ในช่วง vegetative stage → NDVI สูงสุด (~0.5–0.7)
- เหตุผลที่ไม่เลือก พ.ค.–มิ.ย.: เพิ่งเริ่มปลูก ข้าวยังเล็ก NDVI ยังต่ำ
- เหตุผลที่ไม่เลือก ต.ค.–พ.ย.: ใกล้เก็บเกี่ยว ใบเริ่มเหลือง NDVI ลดลง

**ฤดูแล้ง: 1 มกราคม – 31 มีนาคม 2023**
- หลังเก็บเกี่ยวข้าว → นาโล่ง ดินแห้ง NDVI ต่ำ
- เหตุผลที่ไม่เลือก พ.ย.–ธ.ค.: ยังมีข้าวบางส่วนในนา ไม่ใช่ช่วง peak dry season

**Cloud Cover Filter: < 20%**
- เกณฑ์ 10% จะได้ภาพ ฤดูฝน ~1–2 ภาพ (ไม่พอสร้าง Composite ที่ดี)
- เกณฑ์ 20% ได้ภาพ 4–6 ภาพ → Median Composite ขจัด cloud artifact ได้ดี
- Cloud Mask เพิ่มเติมด้วย QA_PIXEL (Bit 3: Shadow, Bit 5: Cloud)

---

##  ภารกิจที่ 3: Methodology การวิเคราะห์

### A. Spectral Indices (NDVI, NDWI, NDBI)

```
NDVI = (SR_B5 − SR_B4) / (SR_B5 + SR_B4)
     = (NIR − Red) / (NIR + Red)
     ค่า > 0.3 = พืชพรรณปกคลุม
     ค่า < 0   = น้ำ, สิ่งปลูกสร้าง, ดินโล่ง

NDWI = (SR_B3 − SR_B5) / (SR_B3 + SR_B5)
     = (Green − NIR) / (Green + NIR)   [McFeeters, 1996]
     ค่า > 0 = น้ำ / ความชื้นสูง
     ค่า < 0 = ดิน / พืช

NDBI = (SR_B6 − SR_B5) / (SR_B6 + SR_B5)
     = (SWIR1 − NIR) / (SWIR1 + NIR)
     ค่า > 0 = สิ่งปลูกสร้าง / พื้นผิวแข็ง
```

**Scale Factor (Landsat 8 C2):** ค่า DN ต้องคูณ 0.0000275 + (-0.2) ก่อนคำนวณ Index

### B. Zonal Statistics

- ใช้ `ee.Reducer.mean()` คำนวณ NDVI/NDWI/NDBI เฉลี่ยต่อ Zone
- Grid ขนาด ~5.5 km (0.05°) เป็น proxy สำหรับหน่วยบริหาร
- สำหรับข้อมูลตำบลจริง: ต้องอัปโหลด Shapefile ของ DOPA (กรมการปกครอง) เข้า GEE Assets

### C. Change Detection

```
ΔNDVI = NDVI_ฤดูฝน − NDVI_ฤดูแล้ง
       > 0 = สูญเสียพืชพรรณในฤดูแล้ง
       < 0 = พืชพรรณเพิ่มขึ้น/คงที่ (ชลประทาน)
```

---

##  ภารกิจที่ 4: Export Strategy

### CRS ที่เลือก: EPSG:32648 (WGS 84 / UTM Zone 48N)

**เหตุผล:**
- ร้อยเอ็ดอยู่ในช่วงลองจิจูด 103–104°E → อยู่ใน UTM Zone 48N พอดี
- Projected CRS (หน่วยเมตร) ทำให้คำนวณพื้นที่ได้ถูกต้อง (ไม่บิดเบือนเหมือน Geographic CRS)
- เป็น Standard ที่หน่วยงานไทยส่วนใหญ่ใช้ (กรมแผนที่ทหาร, ONEP)
- ทางเลือกอื่น: EPSG:24048 (Indian 1975 / UTM Zone 48N) — แต่ WGS84 เป็น Global standard มากกว่า

### Scale ที่เลือก: 30 m

| Scale | ขนาดไฟล์ (est.) | ความแม่นยำ | เหมาะกับ |
|-------|----------------|-----------|---------|
| 10 m | ~500 MB | สูงมาก | Sentinel-2, urban detail |
| **30 m** | **~60 MB** | **ดี** | **Landsat native, ระดับอำเภอ** |
| 100 m | ~5 MB | ต่ำ | ภาพรวม, ระดับจังหวัด |
| 300 m | ~0.5 MB | ต่ำมาก | MODIS, ระดับประเทศ |

**เหตุผล:** 30m = native resolution ของ Landsat 8 — การ downsample เป็น 10m จะไม่ได้ข้อมูลเพิ่มขึ้น แต่ไฟล์ใหญ่ขึ้น 9 เท่า ส่วนการ upsample เป็น 100m จะสูญเสีย spatial detail

### Format: GeoTIFF
- เปิดได้กับทุก GIS Software (QGIS, ArcGIS, ERDAS)
- รองรับ georeferencing metadata
- เหมาะกับ raster data ที่มี continuous values (ไม่ใช่ discrete classes)

---

##  ตอบคำถาม 4 ข้อ

### คำถามที่ 1: ถ้าเลือกดาวเทียมอีกดวง ผลการวิเคราะห์จะต่างออกไปอย่างไร?

**เปรียบเทียบ Landsat 8 vs Sentinel-2 (ทดสอบจริงใน Notebook Section Bonus):**

| ประเด็น | Landsat 8 (30m) | Sentinel-2 (10m) |
|---------|----------------|-----------------|
| **ค่า NDVI mean** | ใกล้เคียงกัน (~0.20–0.25 ฤดูแล้ง) | ใกล้เคียงกัน |
| **NDVI stdDev** | ต่ำกว่า | **สูงกว่า** — จับความแตกต่างระหว่าง field paths/roads ได้ |
| **พื้นที่น้ำ** | Mixed pixel กับดิน/พืชข้างเคียง | แม่นยำกว่า จับแม่น้ำแคบ ๆ ได้ |
| **เขตเมือง** | บล็อกสี่เหลี่ยม 30m หยาบ | เห็น building footprint ชัดขึ้น |
| **Processing time** | เร็วกว่า 3–5 เท่า | ช้ากว่า ไฟล์ใหญ่กว่า |
| **Cloud masking** | QA_PIXEL (reliable) | SCL layer (บางครั้ง under-mask เมฆบาง) |

**สรุป:** สำหรับโจทย์ระดับอำเภอ (land management planning) ความแตกต่างในแง่ political decision ไม่มากนัก แต่ Sentinel-2 จะให้ความแม่นยำสูงกว่าในพื้นที่ที่ต้องการ detail เช่น บริเวณชายเมือง ริมน้ำ หรือชลประทานรายแปลง

---

### คำถามที่ 2: Index ที่เลือกมีข้อจำกัดอะไร? มี Index อื่นที่เหมาะกว่าหรือไม่?

**ข้อจำกัดของ NDVI:**
- **Saturation ที่ค่าสูง:** เมื่อ NDVI > 0.8 ความแตกต่างของ biomass จะไม่สะท้อนออกมา (เกิดใน canopy ป่า ไม่ใช่นาข้าว)
- **Sensitive ต่อ soil background:** ดินสีแดงของอีสานมีค่า reflectance NIR สูง ทำให้ NDVI ของดินโล่ง "ดูสูงกว่าความจริง"
- **ไม่แยก vegetation type:** ข้าว หญ้า และสวนยาง อาจมี NDVI ใกล้เคียงกัน

**ข้อจำกัดของ NDWI (McFeeters):**
- ตอบสนองต่อน้ำผิวดินเป็นหลัก ไม่ได้วัดความชื้นดิน
- Urban area สร้าง false positive ได้ (concrete บางชนิดสะท้อน Green สูง)

**Index ที่ควรพิจารณาเพิ่ม:**

| Index | สูตร | เหมาะกว่าสำหรับ |
|-------|------|----------------|
| **EVI** (Enhanced VI) | 2.5*(NIR-Red)/(NIR+6R-7.5B+1) | แก้ soil background ดีกว่า NDVI |
| **LSWI** (Land Surface Water Index) | (NIR-SWIR1)/(NIR+SWIR1) | วัดความชื้นดินและพืชดีกว่า NDWI |
| **MNDWI** (Modified NDWI) | (Green-SWIR1)/(Green+SWIR1) | แยกน้ำกับสิ่งปลูกสร้างได้ดีกว่า |
| **SAVI** (Soil Adj. VI) | 1.5*(NIR-Red)/(NIR+Red+0.5) | เหมาะกับพื้นที่ดินโล่งภาคอีสาน |

**สำหรับพื้นที่ร้อยเอ็ด:** LSWI จะเหมาะกว่า NDWI ในการติดตามความชื้นดินก่อนฤดูกาลเพาะปลูก เพราะใช้ SWIR ที่ sensitive ต่อ canopy water content และ soil moisture มากกว่า

---

### คำถามที่ 3: ถ้านำผลลัพธ์ไปใช้จริงกับหน่วยงานรัฐ ข้อมูลที่ยังขาดอยู่คืออะไร?

**ข้อมูลที่ยังขาดและจำเป็นต้องมี:**

1. **Ground Truth / Validation Data**
   - ข้อมูลการใช้ที่ดินจริงจากกรมพัฒนาที่ดิน (LDD) ปี 2022–2023
   - ถ้าไม่มี ไม่สามารถรู้ได้ว่า NDVI สูงในจุดนั้นคือ "ข้าว" "สวนผลไม้" หรือ "หญ้า"

2. **ข้อมูลชลประทาน**
   - แผนที่เขตชลประทาน (กรมชลประทาน) ระดับตำบล เพื่อ overlay กับ NDWI และระบุว่า Zone ไหน "แห้งเพราะไม่มีชลประทาน" vs "แห้งเพราะพืชเก็บเกี่ยวแล้ว"

3. **ข้อมูลดินและธรณีวิทยา**
   - ชุดดินจากกรมพัฒนาที่ดิน → soil reflectance ต่างกันทำให้ NDVI baseline ต่างกัน พื้นที่ที่ดูเหมือน "ขาดน้ำ" อาจเป็นแค่ดินสีแดง

4. **ข้อมูลสังคมเศรษฐกิจ**
   - จำนวนเกษตรกร รายได้เฉลี่ย สัดส่วนพื้นที่เกษตรต่อครัวเรือน → เชื่อมกับ Zone ที่มี NDVI ต่ำในฤดูแล้ง เพื่อระบุกลุ่มเปราะบาง

5. **ข้อมูลอุตุนิยมวิทยา**
   - สถานีวัดน้ำฝน (ต้องการอย่างน้อย 3 สถานีในพื้นที่) เพื่อ correlate ปริมาณฝนกับ NDVI จริง ๆ ไม่ใช่แค่ assumed

6. **Boundary ระดับตำบลที่แม่นยำ**
   - ปัจจุบัน GEE ไม่มีขอบตำบลไทย ต้องดาวน์โหลดจาก DOPA และ upload เป็น GEE Assets ก่อน

---

### คำถามที่ 4: การเลือก Scale ใน Export มีผลต่อขนาดไฟล์และความแม่นยำอย่างไร?

**หลักการพื้นฐาน:**

ขนาดไฟล์ ∝ (พื้นที่) / (Scale²)
- ลด scale จาก 30m → 10m: ขนาดไฟล์เพิ่ม 9 เท่า (พิกเซลต่อพื้นที่เพิ่ม 9 เท่า)
- ลด scale จาก 30m → 100m: ขนาดไฟล์ลด ~11 เท่า

**ตัวเลขจริงสำหรับพื้นที่เมืองร้อยเอ็ด (~500 km²):**

| Scale | จำนวนพิกเซล | ขนาดไฟล์ GeoTIFF (1 band, Float32) |
|-------|------------|----------------------------------|
| 10 m | ~5,000,000 | ~20 MB |
| **30 m** | **~555,000** | **~2.2 MB** |
| 100 m | ~50,000 | ~0.2 MB |
| 300 m | ~5,500 | ~0.02 MB |

**ผลต่อความแม่นยำ:**

- **Upsampling (export ที่ scale เล็กกว่า native):** เช่น export Landsat 8 ที่ 10m — GEE จะ interpolate ระหว่างพิกเซล ไม่ได้ข้อมูลเพิ่ม แต่ไฟล์ใหญ่ขึ้น 9 เท่า → **ไม่คุ้ม**

- **Downsampling (export ที่ scale ใหญ่กว่า native):** เช่น export ที่ 100m — GEE จะ average พิกเซลรวม 3x3 บล็อก ทำให้ขอบระหว่าง land cover เบลอ → สูญเสีย spatial detail แต่เหมาะกับการวิเคราะห์ระดับจังหวัด/ประเทศ

- **Trade-off ที่เหมาะสม:** Export ที่ native resolution (30m สำหรับ Landsat 8) คือ optimal balance ระหว่าง accuracy และ file size เสมอ ยกเว้นกรณีที่มีข้อจำกัด storage หรือต้องการแชร์ให้ผู้ใช้ทั่วไปที่ไม่มี GIS software — อาจ export 100m พร้อม web tile แทน

---

## สิ่งที่ค้นพบ (Key Findings)

1. **NDVI ลดลง 40–60% ในฤดูแล้ง:** พื้นที่ส่วนใหญ่ของอำเภอเป็นนาข้าวที่เก็บเกี่ยวแล้ว mean NDVI จาก ~0.45 ลดเหลือ ~0.22

2. **บึงพลาญชัยเป็น stable water source:** NDWI > 0.3 ตลอดปี ยืนยันเป็นแหล่งน้ำถาวร ขณะที่พื้นที่ชลประทานรอบนอกลด NDWI ลงในฤดูแล้ง

3. **Zonal Statistics ชี้ให้เห็น spatial heterogeneity:** Zone ทางตะวันตกของอำเภอ (ห่างจากตัวเมืองและแม่น้ำ) มี NDVI และ NDWI ต่ำสุด — บ่งชี้ความเสี่ยงขาดแคลนน้ำสูงกว่า

4. **Sentinel-2 ให้ผลใกล้เคียง Landsat 8:** ค่า mean NDVI ต่างกันไม่เกิน 5% แต่ spatial detail ชัดกว่าในระดับ field (Sentinel-2 จับ field boundaries ได้ชัดกว่า)

5. **NDBI สูงเฉพาะเขตเมืองชั้นใน:** บริเวณรอบตลาดสดเทศบาลเมืองร้อยเอ็ด NDBI > 0.1 ขณะที่พื้นที่เกษตร NDBI ใกล้ศูนย์

---

##  References

- Landsat 8 Collection 2: USGS (2022). *Landsat Collection 2 Level-2 Science Product Guide*
- McFeeters, S.K. (1996). The use of the Normalized Difference Water Index (NDWI) in the delineation of open water features. *International Journal of Remote Sensing*, 17(7), 1425–1432.
- Zha, Y., Gao, J., & Ni, S. (2003). Use of normalized difference built-up index in automatically mapping urban areas from TM imagery. *International Journal of Remote Sensing*, 24(3), 583–594.
- Gorelick, N., et al. (2017). Google Earth Engine: Planetary-scale geospatial analysis for everyone. *Remote Sensing of Environment*, 202, 18–27.

---

*สร้างโดย: ทีมนักวิทยาศาสตร์ข้อมูล | ภม.338 Geographic Data Science | Cloud Project: ee-kittichot6692*
