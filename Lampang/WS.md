### การแก้ไขเดือน

1. Add Column -> Custom Column 
2. เติมข้อมูลดังนี้ 
      New column name 
3.    Custom column fomula
```
{"มกราคม","กุมภาพันธ์","มีนาคม","เมษายน","พฤษภาคม","มิถุนายน",
"กรกฎาคม","สิงหาคม","กันยายน","ตุลาคม","พฤศจิกายน","ธันวาคม"}
{Number.From(Text.Remove([Attribute], {"r", "t"})) - 1}
```
4. OK
