# Get Data
## File 
## API - HDC
### 📑 ดึงข้อมูลจาก HDC 

Home -> Tranform data -> Home -> New Source  -> Blank Qury
```powerquery
let
    // 1. สร้างรายการปีที่ต้องการดึงข้อมูล (สามารถเพิ่มปีได้)
    YearsList = {"2566", "2567", "2568", "2569"}, 
    
    // 2. แปลงรายการปีให้เป็นตาราง
    YearsTable = Table.FromList(YearsList, Splitter.SplitByNothing(), {"Year"}, null, ExtraValues.Error),
    
    // 3. ฟังก์ชันดึงข้อมูล พร้อมระบบตรวจสอบความผิดพลาด
    GetData = (TargetYear as text) =>
        let
            url = "[https://opendata.moph.go.th/api/report_data](https://opendata.moph.go.th/api/report_data)",
            body = "{
                ""tableName"": ""s_epi1_w"",
                ""year"": """ & TargetYear & """,
                ""province"": ""11"",
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
            
    // 4. เรียกใช้งานฟังก์ชันเพื่อสร้างคอลัมน์ข้อมูล
    InvokedFunction = Table.AddColumn(YearsTable, "Data", each GetData([Year])),
    
    // 5. กรองเอาแถวที่เป็น null (ปีที่ไม่มีข้อมูลหรือดึงไม่สำเร็จ) ออกไปทันที
    FilteredRows = Table.SelectRows(InvokedFunction, each ([Data] <> null))
in
    FilteredRows

```
### 📑 ดึงข้อมูลจาก HDC - มากกว่า 1 

Home -> Tranform data -> Home -> New Source  -> Blank Qury

```
let
    // 1. สร้างรายการปี
    YearsList = {"2566", "2567", "2568", "2569"},
    YearsTable = Table.FromList(YearsList, Splitter.SplitByNothing(), {"Year"}, null, ExtraValues.Error),

    // 2. สร้างรายการรหัสจังหวัด (77 จังหวัดของประเทศไทย)
    ProvincesList = {
        "10","11","12","13","14","15","16","17","18","19",
        "20","21","22","23","24","25","26","27",
        "30","31","32","33","34","35","36","37","38","39",
        "40","41","42","43","44","45","46","47","48","49",
        "50","51","52","53","54","55","56","57","58",
        "60","61","62","63","64","65","66","67",
        "70","71","72","73","74","75","76","77",
        "80","81","82","83","84","85","86",
        "90","91","92","93","94","95","96"
    },
    ProvincesTable = Table.FromList(ProvincesList, Splitter.SplitByNothing(), {"Province"}, null, ExtraValues.Error),

    // 3. จับคู่ทุกปี เข้ากับ ทุกจังหวัด (Cross Join) จะได้ทั้งหมด 308 แถว
    AddProvinces = Table.AddColumn(YearsTable, "Custom", each ProvincesTable),
    ExpandedProvinces = Table.ExpandTableColumn(AddProvinces, "Custom", {"Province"}, {"Province"}),

    // 4. ฟังก์ชันดึงข้อมูลตาม ปี และ จังหวัด ที่จับคู่ไว้
    GetData = (TargetYear as text, TargetProvince as text) =>
        let
            url = "https://opendata.moph.go.th/api/report_data",
            body = "{
                ""tableName"": ""s_epi1_w"", 
                ""year"": """ & TargetYear & """,
                ""province"": """ & TargetProvince & """,
                ""type"": ""json""
            }",
            
            // ดักจับ Error เผื่อบางจังหวัดในบางปีไม่มีข้อมูลหรือเซิร์ฟเวอร์ตอบกลับช้า
            Response = try Json.Document(Web.Contents(url, [
                Headers = [#"Content-Type"="application/json"],
                Content = Text.ToBinary(body),
                ManualStatusHandling = {400, 404, 500}
            ])),
            
            Result = if Response[HasError] then null 
                     else if Response[Value] is list and List.IsEmpty(Response[Value]) then null
                     else if Response[Value] is record and Record.FieldCount(Response[Value]) = 0 then null
                     else Response[Value]
        in
            Result,

    // 5. เรียกใช้ฟังก์ชันเพื่อดึงข้อมูลของแต่ละแถว
    InvokedFunction = Table.AddColumn(ExpandedProvinces, "Data", each GetData([Year], [Province])),

    // 6. กรองเอาแถวที่เป็น null ทิ้ง (แถวที่ไม่มีข้อมูลจะได้ไม่รกตาราง)
    FilteredRows = Table.SelectRows(InvokedFunction, each ([Data] <> null))
in
    FilteredRows

```
