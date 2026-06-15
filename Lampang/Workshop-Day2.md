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
####*
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

### 2. Fact_EPI
#### คำนวน จำนวนการฉีดวัคซีนในเด็ก ดังนี้
 1. นำเม้าไปวางที่ตาราง Fact_EPI_target
 2. ที่เมนู Table tool เลือก New measure
 3. วาง Code ดังนี้

        A_Vaccinated = SUM(FACT_EPI[Value])

#### คำนวน ความครอบคลุมการฉีดวัคซีนในเด็ก ดังนี้   
 1. นำเม้าไปวางที่ตาราง Fact_EPI
 2. ที่เมนู Table tool เลือก New measure
 3. วาง Code ดังนี้

        % Coverage = 
         DIVIDE(FACT_EPI[A_Vaccinated], Fact_EPI_target[Measure_B_Target_Children], 0)

#### ดึงค่าเข็มที่การ ฉีดวัคซีน
1. นำเม้าไปวางที่ตาราง Fact_EPI
2. ที่เมนู Table tool เลือก New Column
3. วาง Code ดังนี้

          Dose_Number = 
         VAR _LastChar = RIGHT('ชื่อตารางของคุณ'[Vaccine], 1)
         VAR _CheckNumber = IFERROR(VALUE(_LastChar), BLANK())
         RETURN
         IF(
             ISBLANK(_CheckNumber),
             "เข็มที่ 1",   // ถ้าวัคซีนไม่มีเลขกำกับ (เช่น bcg) จะให้แสดงผลเป็น เข็มที่ 1 หรือจะปล่อยว่างก็ได้โดยเปลี่ยนเป็น BLANK()
             "เข็มที่ " & _LastChar
         )

#### ดึงชื่อเข็มที่การ ฉีดวัคซีน
1. นำเม้าไปวางที่ตาราง Fact_EPI
2. ที่เมนู Table tool เลือก New Column
3. วาง Code ดังนี้
   
         Vaccine_NameOnly = 
         VAR _FullText = 'ชื่อตารางของคุณ'[Vaccine]
         VAR _LastChar = RIGHT(_FullText, 1)
         VAR _IsNumber = IFERROR(VALUE(_LastChar), BLANK())
         RETURN
         IF(
             ISBLANK(_IsNumber),
             _FullText,                               // กรณีตัวสุดท้ายไม่ใช่ตัวเลข (เช่น bcg) ให้แสดงชื่อเต็มได้เลย
             LEFT(_FullText, LEN(_FullText) - 1)      // กรณีตัวสุดท้ายเป็นตัวเลข ให้ตัดตัวอักษรขวาสุดออก 1 ตัว
         )
-------

# Visualization
#####1. วิธีการสร้างและตั้งค่า Slicer

### 1. สร้างตัวกรอง ปี {B_YEAR} จากตาราง DIM_YEAR
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
  
### 2. ขั้นตอนการสร้างตัวกรอง (Slicer) อำเภอ/เขต

การสร้างตัวกรองพื้นที่เพื่อให้ผู้ใช้งานสามารถเลือกดูข้อมูลตามระดับอำเภอหรือเขตได้ มีขั้นตอนดังนี้:

1. **เพิ่ม Visual แบบ Slicer:**
   * ในหน้า Report view ไปที่แพลนเนล **Visualizations** (ด้านขวามือ)
   * คลิกที่ไอคอน **Slicer** เพื่อสร้างกล่องตัวกรองใหม่บนแคนวาส

2. **นำข้อมูล อำเภอ/เขต ใส่ใน Slicer:**
   * ไปที่แพลนเนล **Data** ขยายตาราง `DIM_AREA`
   * ลากฟิลด์ `อำเภอ/เขต` มาวางในช่อง **Field** ของ Slicer ที่สร้างไว้ 
   * *(หมายเหตุ: หากในตารางจริงใช้ชื่อคอลัมน์อื่น เช่น ตำบล/แขวง หรือ เขตบริการ ให้เลือกฟิลด์ที่ตรงตามโครงสร้างข้อมูลของคุณ)*

3. **ตั้งค่า Slicer Header เป็น "อำเภอ/เขต":**
   * ตรวจสอบว่าคลิกเลือก Slicer นี้อยู่
   * ไปที่แพลนเนล Visualizations คลิกที่ไอคอน **Format your visual** (รูปแปรงทาสี)
   * ไปที่แท็บ **Visual** > ขยายหัวข้อ **Slicer header**
   * ในช่อง **Title text** ให้พิมพ์คำว่า `อำเภอ/เขต`

---

### 3. ขั้นตอนการสร้างตัวกรอง (Slicer) วัคซีน

การสร้างตัวกรองเพื่อให้ผู้ใช้งานเลือกดูข้อมูลตามชนิดของวัคซีนที่ต้องการ:

1. **เพิ่ม Visual แบบ Slicer:**
   * คลิกพื้นที่ว่างบนแคนวาสเพื่อไม่ให้ทับ Visual เดิม
   * ไปที่แพลนเนล **Visualizations** แล้วคลิกที่ไอคอน **Slicer** เพื่อสร้างตัวกรองใหม่อีก 1 อัน

2. **นำข้อมูล Vaccine ใส่ใน Slicer:**
   * ไปที่แพลนเนล **Data** ขยายตาราง `DIM_EPI` 
   * *(หมายเหตุ: อ้างอิงจากแผนภาพก่อนหน้า ข้อมูลวัคซีนอาจอยู่ในตาราง `FACT_EPI` ให้ใช้ตารางที่มีฟิลด์ข้อมูลวัคซีนของคุณ)*
   * ลากฟิลด์ `Vaccine_NameOnly` มาวางในช่อง **Field** ของ Slicer ใหม่นี้

3. **ตั้งค่า Slicer Header เป็น "วัคซีน":**
   * ในขณะที่เลือก Slicer นี้อยู่ ให้ไปที่ไอคอน **Format your visual** (รูปแปรงทาสี)
   * ไปที่แท็บ **Visual** > ขยายหัวข้อ **Slicer header**
   * ในช่อง **Title text** ให้พิมพ์คำว่า `วัคซีน`

💡 **Tip เพิ่มเติมสำหรับการตั้งค่า Slicer ทั้ง 2 อัน:** หากข้อมูลมีรายการให้เลือกเยอะ แนะนำให้ไปที่ **Format your visual** > **Visual** > **Slicer settings** > **Style** แล้วเลือกเป็น **Dropdown** เพื่อประหยัดพื้นที่บนหน้ารายงาน หรือเปิดใช้งานฟังก์ชัน **Search** ให้ผู้ใช้พิมพ์ค้นหาชื่ออำเภอและวัคซีนได้ง่ายขึ้น
    
