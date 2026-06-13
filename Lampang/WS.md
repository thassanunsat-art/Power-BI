# Get Data
1. File Excell : Home --> New Sources --> Excell --> เลือกไฟล์ที่ต้องนำเข้า --> OK
<img width="447" height="347" alt="image" src="https://github.com/user-attachments/assets/5b137eaa-3b9d-4aab-882d-7cd47287ef82" />

### การแก้ไขเดือน

1. Add Column -> Custom Column 
2. เติมข้อมูลดังนี้ 
      New column name 
3.    Custom column fomula
<img width="341" height="215" alt="image" src="https://github.com/user-attachments/assets/0b9bc44b-ca40-4331-b1ac-713982c045db" />

```
{"มกราคม","กุมภาพันธ์","มีนาคม","เมษายน","พฤษภาคม","มิถุนายน",
"กรกฎาคม","สิงหาคม","กันยายน","ตุลาคม","พฤศจิกายน","ธันวาคม"}
{Number.From(Text.Remove([Attribute], {"r", "t"})) - 1}
```
4. OK
