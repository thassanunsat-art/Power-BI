# 1. Get & Transform
## นำข้อมูลเข้า ดังนี้
1. DIM_HealthCode_lampang : Home --> New Sources --> Excell --> เลือกไฟล์ที่ต้องนำเข้า --> OK
2. DIM_EPI_Name : Home --> Query --> Advanced Editor --> วาง Code ที่แนบ
3. Fact_EPI : Home --> --> Query --> Advanced Editor --> วาง Code ที่แนบ
4. B_YEAR : Home --> New Query -->Advanced Editor --> วาง Code ที่แนบ

### 1. DIM_HealthCode_lampang 
: Home --> New Sources --> Excell --> เลือกไฟล์ที่ต้องนำเข้า --> OK

### 2. DIM_EPI_Name : Home --> Query -->Advanced Editor --> วาง Code ที่แนบ
 ```
   let
    Source = Table.FromRows({
        {"010", "บีซีจี (BCG)"},
        {"031", "ดีทีพี1"},
        {"032", "ดีทีพี2"},
        {"033", "ดีทีพี3"},
        {"041", "ตับอักเสบบี1"},
        {"042", "ตับอักเสบบี2"},
        {"043", "ตับอักเสบบี3"},
        {"061", "หัด คางทูม หัดเยอรมัน1"},
        {"081", "โอพีวี1"},
        {"082", "โอพีวี2"},
        {"083", "โอพีวี3"},
        {"091", "ดีทีพีตับอักเสบบี1"},
        {"092", "ดีทีพีตับอักเสบบี2"},
        {"093", "ดีทีพีตับอักเสบบี3"},
        {"401", "ไอพีวี-ฉีด"},
        {"D11", "ดีทีพีฮิบ1"},
        {"D12", "ดีทีพีฮิบ2"},
        {"D13", "ดีทีพีฮิบ3"},
        {"D21", "ดีทีพีตับฮิบ1"},
        {"D22", "ดีทีพีตับฮิบ2"},
        {"D23", "ดีทีพีตับฮิบ3"},
        {"D31", "ดีทีพีไอพีวี1"},
        {"D32", "ดีทีพีไอพีวี2"},
        {"D33", "ดีทีพีไอพีวี3"},
        {"D41", "ดีทีพีไอพีวีตับ1"},
        {"D42", "ดีทีพีไอพีวีตับ2"},
        {"D43", "ดีทีพีไอพีวีตับ3"},
        {"D51", "ดีทีพีไอพีวีตับฮิบ1"},
        {"D52", "ดีทีพีไอพีวีตับฮิบ2"},
        {"D53", "ดีทีพีไอพีวีตับฮิบ3"},
        {"H11", "ฮิบ1"},
        {"H12", "ฮิบ2"},
        {"H13", "ฮิบ3"},
        {"H21", "ตับอักเสบบี (สัมผัสเชื้อ)"},
        {"H22", "ตับอักเสบบี (เด็กน้ำหนักน้อย)"},
        {"I11", "ไอพีวี1"},
        {"I12", "ไอพีวี2"},
        {"I13", "ไอพีวี3"},
        {"M11", "เอ็มเอ็มอาร์2"},
        {"P11", "พีซีวีฮิบ1"},
        {"P12", "พีซีวีฮิบ2"},
        {"P13", "พีซีวีฮิบ3"},
        {"P21", "พีซีวี1"},
        {"P22", "พีซีวี2"},
        {"P23", "พีซีวี3"},
        {"R11", "โรต้า2 สายพันธุ์-1"},
        {"R12", "โรต้า2 สายพันธุ์-2"},
        {"R21", "โรต้า5 สายพันธุ์-1"},
        {"R22", "โรต้า5 สายพันธุ์-2"},
        {"R23", "โรต้า5 สายพันธุ์-3"}
    }, type table [Code = text, VaccineName = text]),
    #"Changed Type" = Table.TransformColumnTypes(Source,{{"Code", type text}, {"VaccineName", type text}})
in
    #"Changed Type"
   ```

### 3. Fact_EPIAPI- HDC ความครอบคลุมของเด็กอายุครบ 1 ปีที่ได้รับวัคซีน BCG ,HBV1,DTP1,DTP3,HBV3,Hib3,โปลิโอ3, MMR1,IPV,Rota รายไตรมาส https://opendata.moph.go.th/
Home --> Query -->Advanced Editor --> วาง Code ที่แนบ --> เปลี่ยนชื่อ Table : Fact_EPI
    
```
  let
    // 1. สร้างรายการปีที่ต้องการดึงข้อมูล
    YearsList = {"2566", "2567", "2568", "2569"},
    
    // 2. แปลงรายการปีให้เป็นตาราง
    YearsTable = Table.FromList(YearsList, Splitter.SplitByNothing(), {"Year"}, null, ExtraValues.Error),
    
    // 3. ฟังก์ชันดึงข้อมูล พร้อมระบบตรวจสอบความผิดพลาด (อัปเดต tableName เป็น s_op_instype_all ตามคอมเมนต์แล้ว)
    GetData = (TargetYear as text) =>
        let
            url = "https://opendata.moph.go.th/api/report_data",
            body = "{
                ""tableName"": ""s_epi1"",
                ""year"": """ & TargetYear & """,
                ""province"": ""52"",
                ""type"": ""json""
            }",
            
            // ใช้ try เพื่อดักจับกรณีที่ Web Response เกิด Error
            Response = try Json.Document(Web.Contents(url, [
                Headers = [#"Content-Type"="application/json"],
                Content = Text.ToBinary(body),
                ManualStatusHandling = {400, 404, 500}
            ])),
            
            // เช็คผลลัพธ์: ถ้าเกิด Error หรือได้ผลลัพธ์เป็นตาราง/List ว่างเปล่า ให้คืนค่าเป็น null
            Result = if Response[HasError] then null 
                     else if Response[Value] is list and List.IsEmpty(Response[Value]) then null
                     else if Response[Value] is record and Record.FieldCount(Response[Value]) = 0 then null
                     else Response[Value]
        in
            Result,

    // 4. เรียกใช้ฟังก์ชันดึงข้อมูลตามปี (วนลูปตามรายการปีที่มีในตาราง)
    InvokeGetData = Table.AddColumn(YearsTable, "Data", each GetData([Year])),

    // 5. กรองเอาแถวที่ไม่มีข้อมูลออก (กรณีกดดึงแล้วเป็น null หรือ Error)
    FilterNulls = Table.SelectRows(InvokeGetData, each ([Data] <> null)),

    // 6. แตกข้อมูล (Expand) ออกมาเป็นตารางเดียวกัน
    // หมายเหตุ: ตรงขั้นตอนนี้ Power Query จะทำการ Expand คอลัมน์ที่ได้จาก API 
    // หากโครงสร้าง API ของคุณส่งค่าออกมาเป็น List ของ Record ระบบจะแตกออกมาให้โดยอัตโนมัติครับ
    ExpandedData = Table.ExpandListColumn(FilterNulls, "Data"),
    
    // 7. ดึงฟิลด์ต่างๆ ภายใน Record ออกมา (สมมติฟิลด์มาตรฐานทั่วไป หาก API คืนค่าฟิลด์อื่น สามารถกดเลือกเพิ่มที่หน้าต่าง Power Query ได้ครับ)
    CustomExpanded = Table.ExpandRecordColumn(ExpandedData, "Data", Record.FieldNames(Table.Column(ExpandedData, "Data"){0}? ?? [Dummy=null])),
    #"Unpivoted Columns" = Table.UnpivotOtherColumns(CustomExpanded, {"Year", "id", "hospcode", "areacode", "date_com", "b_year"}, "Attribute", "Value"),
    #"Added Custom" = Table.AddColumn(#"Unpivoted Columns", "Month", each {"มกราคม","กุมภาพันธ์","มีนาคม","เมษายน","พฤษภาคม","มิถุนายน","กรกฎาคม","สิงหาคม","กันยายน","ตุลาคม","พฤศจิกายน","ธันวาคม"}{Number.From(Text.End([Attribute], 2)) - 1}),
    #"Added Custom Column" = Table.AddColumn(#"Added Custom", "Custom", each let splitAttribute = Splitter.SplitTextByDelimiter("_", QuoteStyle.None)([Attribute]) in Text.Combine({Text.Start([Attribute], 6), Text.Middle(splitAttribute{1}?, 2)}), type text),
    #"Reordered Columns" = Table.ReorderColumns(#"Added Custom Column",{"Year", "id", "hospcode", "areacode", "date_com", "b_year", "Custom", "Value", "Month"}),
    #"Filtered Rows1" = Table.SelectRows(#"Reordered Columns", each ([Custom] <> "target")),
    #"Filtered Rows" = Table.SelectRows(#"Filtered Rows1", each ([Custom] <> "target")),
    #"Renamed Columns" = Table.RenameColumns(#"Filtered Rows",{{"Custom", "Vaccine"}}),
    #"Split Column by Delimiter" = Table.SplitColumn(#"Renamed Columns", "Vaccine", Splitter.SplitTextByDelimiter("_", QuoteStyle.Csv), {"Vaccine.1", "Vaccine.2"}),
    #"Changed Type" = Table.TransformColumnTypes(#"Split Column by Delimiter",{{"Vaccine.1", type text}, {"Vaccine.2", type text}}),
    #"Removed Columns1" = Table.RemoveColumns(#"Changed Type",{"Vaccine.2"}),
    #"Renamed Columns1" = Table.RenameColumns(#"Removed Columns1",{{"Vaccine.1", "Vaccine"}}),
    #"Changed Type1" = Table.TransformColumnTypes(#"Renamed Columns1",{{"Value", Int64.Type}})
in
    #"Changed Type1"
```

### Fact_EPI
1. เลือก target10 - pcv3_09 
2. เลือก Transform --> Unpivot data
   <img width="945" height="483" alt="image" src="https://github.com/user-attachments/assets/c1cc95a1-e7d3-41c8-83d1-50da8fb06931" />
3. สร้าง Column เดือน
   - เลือก  Add column name 
   - เลือก  Custom column fomula
   - New column name : Month
   - นำ Code ไปวางที่ Custom column fomula --> OK
```
{"มกราคม","กุมภาพันธ์","มีนาคม","เมษายน","พฤษภาคม","มิถุนายน","กรกฎาคม","สิงหาคม","กันยายน","ตุลาคม","พฤศจิกายน","ธันวาคม"}{Number.From(Text.End([Attribute], 2)) - 1}
```

4. ลบ  Rowe : target
5. เปลี่ยนชื่อ Column : Vaccine
6. duplicate Fact_EPI เป็น Fact_EPI_target

### 4. สร้างตาราง B_YEAR : 
Home --> Query -->Advanced Editor --> วาง Code ที่แนบ

```
let
    // กำหนดช่วงปี
    StartYear = 2550,
    EndYear = 2570,

    // สร้าง list ของปี
    Years = {StartYear..EndYear},

    // สร้าง list ของเดือน 1-12
    Months = {1..12},

    // สร้างทุก combination ของ ปี + เดือน
    YearMonth = List.Combine(
        List.Transform(Years, each 
            List.Transform(Months, (m) => [B_YEAR = _, Month = m])
        )
    ),

    // แปลงเป็น Table
    tbl = Table.FromRecords(YearMonth),

    // สร้างวันที่ (เอาวันที่ 1 ของเดือน)
    AddDate = Table.AddColumn(tbl, "MM", each #date([B_YEAR], [Month], 1), type date),

    // สร้าง Year (ปีปฏิทิน)
    AddYear = Table.AddColumn(AddDate, "Year", each Date.Year([MM]), Int64.Type),

    // สร้าง B_YEAR (ปีงบประมาณ: ถ้าเดือน >= 10 ให้ +1)
    AddFiscalYear = Table.AddColumn(AddYear, "B_YEAR_NEW", each 
        if Date.Month([MM]) >= 10 then [Year] + 1 else [Year], Int64.Type),

    // เลือกคอลัมน์ตามต้องการ
    Result = Table.SelectColumns(AddFiscalYear, {"MM", "B_YEAR_NEW", "Year"}),

    // เปลี่ยนชื่อให้ตรง format ที่คุณใช้
    RenameColumn = Table.RenameColumns(Result, {{"B_YEAR_NEW", "B_YEAR"}}),
    #"Inserted Month" = Table.AddColumn(RenameColumn, "Month", each Date.Month([MM]), Int64.Type),
    #"Inserted Month Name" = Table.AddColumn(#"Inserted Month", "Month Name", each Date.MonthName([MM]), type text),
    #"Removed Columns" = Table.RemoveColumns(#"Inserted Month Name",{"Month"}),
    #"Renamed Columns" = Table.RenameColumns(#"Removed Columns",{{"Month Name", "Month_E"}}),
    #"Added Custom" = Table.AddColumn(#"Renamed Columns", "Month_TH", each Date.MonthName([MM], "th-TH"))
in
    #"Added Custom"
```


   



