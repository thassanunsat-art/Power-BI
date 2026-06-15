'''
let
    // 1. สร้างรายการปี 2560 - 2569 อัตโนมัติ และแปลงเป็นข้อความ (Text)
    YearsList = List.Transform({2560..2569}, Text.From),
    
    // 2. แปลงรายการปีให้เป็นตาราง
    YearsTable = Table.FromList(YearsList, Splitter.SplitByNothing(), {"Year"}, null, ExtraValues.Error),
    
    // 3. ฟังก์ชันดึงข้อมูลตาราง s_pop_sex_age ของจังหวัด 52
    GetData = (TargetYear as text) =>
        let
            url = "https://opendata.moph.go.th/api/report_data",
            body = "{
                ""tableName"": ""s_pop_sex_age"",
                ""year"": """ & TargetYear & """,
                ""province"": ""52"",
                ""type"": ""json""
            }",
            
            // ดักจับ Error กรณีเซิร์ฟเวอร์มีปัญหาหรือปีนั้นไม่มีข้อมูล
            Response = try Json.Document(Web.Contents(url, [
                Headers = [#"Content-Type"="application/json"],
                Content = Text.ToBinary(body),
                ManualStatusHandling = {400, 403, 404, 500}
            ])),
            
            Result = if Response[HasError] then null 
                     else if Response[Value] is list and List.IsEmpty(Response[Value]) then null
                     else if Response[Value] is record and Record.FieldCount(Response[Value]) = 0 then null
                     else Response[Value]
        in
            Result,
            
    // 4. เรียกใช้งานฟังก์ชันเพื่อวนลูปดึงข้อมูลทั้ง 10 ปี
    InvokedFunction = Table.AddColumn(YearsTable, "Data", each GetData([Year])),
    
    // 5. กรองเอาปีที่เป็น null (ไม่มีข้อมูล) ทิ้งไป
    FilteredRows = Table.SelectRows(InvokedFunction, each ([Data] <> null)),
    #"Expanded Data" = Table.ExpandListColumn(FilteredRows, "Data")
in
    #"Expanded Data"

    '''
  
