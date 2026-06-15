# GET DATA
### DIM_AREA
1. DIM_AREA
3. DIM_YEAR
4. FACT_EPI
5. Fact_EPI_target



# DAX : Calculated Columns ( add Column)
### 1. DIM_AREA : 
1. นำเม้าไปวางที่ตาราง DIM_AREA
2. ที่เมนู Table tool เลือก New Column
3. วาง Code ดังนี้
```
   hospcode_H = "H" & [รหัส 5 หลัก]
```
### 2. FACT_EPI : 
1. นำเม้าไปวางที่ตาราง FACT_EPI
2. ที่เมนู Table tool เลือก New Column
3. วาง Code ดังนี้
```
   hospcode_H = "H" & [รหัส 5 หลัก]
```
 ### 3. Fact_EPI_target : 
1. นำเม้าไปวางที่ตาราง Fact_EPI_target
2. ที่เมนู Table tool เลือก New Column
3. วาง Code ดังนี้
```
   hospcode_H = "H" & [รหัส 5 หลัก]
```

# Data Modeling (การสร้างแบบจำลองข้อมูล)
เอกสารนี้อธิบายโครงสร้างความสัมพันธ์ของตารางข้อมูล (Data Model) สำหรับระบบข้อมูลการให้บริการวัคซีน (EPI) และเป้าหมาย ซึ่งประกอบไปด้วย Dimension Tables และ Fact Tables

## 🏗️ Entity Relationship Diagram (ERD)
1. เปิด Power BI Desktop และมองหาแถบเมนูด้านซ้ายมือ
2. คลิกที่ไอคอน **Model view** (มุมมองโมเดล ที่เป็นรูปตารางเชื่อมโยงกัน)
3. ในหน้าต่างแคนวาส คุณจะเห็นตารางข้อมูลทั้งหมดที่คุณโหลดเข้ามา
4. ให้คลิกซ้ายค้างไว้ที่ **คอลัมน์ที่เป็นคีย์หลัก (Primary Key)** จากตาราง Dimension (เช่น คอลัมน์ `hospcode_H` ในตาราง `DIM_AREA`)
5. ลากเมาส์ไปวางทับ **คอลัมน์ที่เป็นคีย์นอก (Foreign Key)** ในตาราง Fact ที่ต้องการเชื่อมโยง (เช่น คอลัมน์ `hospcode_H` ในตาราง `FACT_EPI`)
6. ปล่อยเมาส์ Power BI จะประมวลผลและสร้างเส้นความสัมพันธ์ให้โดยอัตโนมัติ 

### 1. ความสัมพันธ์ของพื้นที่ (Area Relationships)
แสดงความสัมพันธ์ระหว่างตารางมิติพื้นที่ (`DIM_AREA`) กับตารางข้อมูลผลงานและเป้าหมาย
<img width="559" height="375" alt="image" src="https://github.com/user-attachments/assets/61a6440c-e182-4a81-945b-0233cde53f80" />

2. ความสัมพันธ์ของเวลา (Year Relationships)
แสดงความสัมพันธ์ระหว่างตารางมิติเวลา (DIM_YEAR) กับตารางข้อมูลผลงานและเป้าหมาย (จากภาพเป็นความสัมพันธ์แบบ Many-to-Many)
<img width="584" height="356" alt="image" src="https://github.com/user-attachments/assets/58cdb81d-3b30-43ce-b021-2d1c01ca6a54" />


# DAX : Measures
### 1. Fact_EPI_target 
 1. นำเม้าไปวางที่ตาราง Fact_EPI_target
 2. ที่เมนู Table tool เลือก New measure
 3. วาง Code ดังนี้

```
Measure_B_Target_Children = 
SUM('Fact_EPI_target'[จำนวนประชากร])
```

### 2. Fact_EPI_target 
 1. นำเม้าไปวางที่ตาราง Fact_EPI_target
 2. ที่เมนู Table tool เลือก New measure
 3. วาง Code ดังนี้

        A_Vaccinated = SUM(FACT_EPI[Value])
    
 1. นำเม้าไปวางที่ตาราง Fact_EPI_target
 2. ที่เมนู Table tool เลือก New measure
 3. วาง Code ดังนี้

        % Coverage = 
         DIVIDE(FACT_EPI[A_Vaccinated], Fact_EPI_target[Measure_B_Target_Children], 0)

# Visualization
## 1. สร้างตัวกรอง ปี {B_YEAR} จากตาราง DIM_YEAR
### 🎛️ ขั้นตอนการสร้างตัวกรอง (Slicer) ปีงบประมาณ
การสร้างตัวกรองสำหรับเลือกปีงบประมาณจากตาราง `DIM_YEAR` และการปรับแต่งชื่อหัวข้อ (Slicer Header) มีขั้นตอนดังนี้

##### วิธีการสร้างและตั้งค่า Slicer
1. **เปิดมุมมองรายงาน (Report view):**
   * ที่แถบเมนูด้านซ้ายมือ ให้คลิกเลือกไอคอน **Report view** (รูปกราฟแท่ง) เพื่อเข้าสู่หน้าแคนวาสสำหรับสร้างรายงาน

2. **เพิ่ม Visual แบบ Slicer:**
   * ไปที่แพลนเนล **Visualizations** (ด้านขวามือ)
   * คลิกที่ไอคอน **Slicer** (รูปกรวยกรองข้อมูลที่มีตาราง) 
   * กล่อง Visual ของ Slicer จะปรากฏขึ้นบนแคนวาส

3. **นำข้อมูล B_YEAR ใส่ใน Slicer:**
   * ไปที่แพลนเนล **Data** (ขวาสุด)
   * ขยายตาราง `DIM_YEAR`
   * คลิกค้างที่ฟิลด์ `B_YEAR` แล้วลากมาปล่อยในช่อง **Field** ของ Slicer (หรือลากไปวางบนกล่อง Slicer บนแคนวาสโดยตรง)

4. **ตั้งค่า Slicer Header เป็น "ปีงบประมาณ":**
   * คลิกเลือกที่กล่อง Slicer บนแคนวาส
   * ในแพลนเนล Visualizations ให้คลิกที่ไอคอน **Format your visual** (รูปแปรงทาสี)
   * เลือกแท็บ **Visual** 
   * ไปที่หัวข้อ **Slicer header** (ตรวจสอบให้แน่ใจว่าเปิดใช้งาน/Turn On อยู่)
   * ในช่อง **Title text** ให้ลบชื่อเดิมออก แล้วพิมพ์คำว่า `ปีงบประมาณ` ลงไป

5. **ปรับรูปแบบการแสดงผล (ทางเลือกเพิ่มเติม):**
   * เพื่อให้การเลือกปีดูง่ายขึ้น คุณสามารถไปที่ **Format your visual** > **Visual** > **Slicer settings**
   * ในส่วนของ **Style** สามารถเลือกเป็น **Dropdown** (เมนูแบบเลื่อนลง) หรือ **List** (รายการ) ตามความเหมาะสมของพื้นที่จัดแสดง
    
