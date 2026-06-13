### การแก้ไขเดือน

 Add Column -> Custom Column 
 เติมข้อมูลดังนี้ 
      New column name 
Custom column fomula
```
{"มกราคม","กุมภาพันธ์","มีนาคม","เมษายน","พฤษภาคม","มิถุนายน",
"กรกฎาคม","สิงหาคม","กันยายน","ตุลาคม","พฤศจิกายน","ธันวาคม"}
{Number.From(Text.Remove([Attribute], {"r", "t"})) - 1}
```
