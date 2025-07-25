# Requirements Analysis Template - ระบบขอเอกสารออนไลน์

## 1. Stakeholder Analysis

### 1.1 Primary Stakeholders
- **นักศึกษา** - ผู้ขอเอกสาร
- **อาจารย์ที่ปรึกษา** - ผู้อนุมัติลำดับแรก
- **เจ้าหน้าที่ทะเบียน** - ผู้ดำเนินการและอนุมัติสุดท้าย

### 1.2 Secondary Stakeholders
- **หัวหน้าภาควิชา** - กรณีเอกสารพิเศษ
- **คณบดี** - กรณีเอกสารระดับคณะ
- **ผู้ดูแลระบบ** - การจัดการและบำรุงรักษา

## 2. Business Requirements

### 2.1 Core Business Rules
- นักศึกษาต้องได้รับการอนุมัติจากอาจารย์ที่ปรึกษาก่อนเสมอ
- แต่ละสาขาวิชามีอาจารย์ที่ปรึกษาแตกต่างกัน
- เอกสารแต่ละประเภทมีข้อกำหนดและระยะเวลาต่างกัน
- ต้องมี audit trail สำหรับการติดตาม

### 2.2 Document Types Requirements

#### Template สำหรับเก็บข้อมูลประเภทเอกสาร:
```
เอกสารประเภท: ________________
- ข้อมูลที่จำเป็น: ________________
- เอกสารแนบที่ต้องการ: ________________
- ระยะเวลาดำเนินการ: ________________ วัน
- ค่าธรรมเนียม: ________________ บาท
- เงื่อนไขพิเศษ: ________________
- ผู้อนุมัติเพิ่มเติม (ถ้ามี): ________________
```

**ตัวอย่างเอกสารที่ควรสำรวจ:**
- ใบรับรองการเป็นนักศึกษา
- ใบแสดงผลการเรียน (Transcript)
- หนังสือรับรองการสำเร็จการศึกษา
- ใบรับรองคะแนนเฉลี่ย (GPA)
- หนังสือขอผ่อนผันค่าธรรมเนียม
- หนังสือรับรองความประพฤติ
- เอกสารสำหรับขอทุนการศึกษา

## 3. User Requirements

### 3.1 นักศึกษา (Student)
**Functional Requirements:**
- สามารถสมัครสมาชิกและเข้าสู่ระบบได้
- เลือกประเภทเอกสารที่ต้องการ
- กรอกข้อมูลและแนบเอกสารประกอบ
- ติดตามสถานะคำขอ
- รับการแจ้งเตือนผ่าน email/SMS
- ดาวน์โหลดเอกสารที่เสร็จแล้ว

**Non-Functional Requirements:**
- ระบบต้องใช้งานง่าย (User-friendly)
- รองรับการใช้งานบนมือถือ
- ข้อมูลต้องปลอดภัย

### 3.2 อาจารย์ที่ปรึกษา (Advisor)
**Functional Requirements:**
- เข้าสู่ระบบด้วย credentials ของอาจารย์
- ดูรายการคำขอของนักศึกษาที่อยู่ในความดูแล
- อนุมัติ/ปฏิเสธ พร้อมเหตุผล
- เพิ่มความเห็นหรือข้อแนะนำ
- ตั้งค่าการแจ้งเตือน

### 3.3 เจ้าหน้าที่ทะเบียน (Registry Staff)
**Functional Requirements:**
- เห็นคำขอที่ผ่านการอนุมัติจากอาจารย์แล้ว
- ตรวจสอบและดำเนินการออกเอกสาร
- อนุมัติสุดท้ายและส่งเอกสาร
- จัดการข้อมูลประเภทเอกสารและค่าธรรมเนียม
- สร้างรายงานสถิติ

## 4. System Requirements

### 4.1 Data Requirements

#### 4.1.1 Student Master Data
```
ข้อมูลนักศึกษา:
- รหัสนักศึกษา (Student ID)
- ชื่อ-นามสกุล (ไทย/อังกฤษ)
- สาขาวิชา/ภาควิชา
- คณะ
- ชั้นปี
- สถานะการเรียน
- อาจารย์ที่ปรึกษา ID
- ข้อมูลติดต่อ (Email, เบอร์โทร)
- ที่อยู่
```

#### 4.1.2 Advisor Master Data
```
ข้อมูลอาจารย์ที่ปรึกษา:
- รหัสอาจารย์
- ชื่อ-นามสกุล
- ตำแหน่ง
- ภาควิชา/คณะ
- กลุ่มนักศึกษาที่ดูแล
- ข้อมูลติดต่อ
- สิทธิ์การอนุมัติ
```

#### 4.1.3 Document Type Master Data
```
ประเภทเอกสาร:
- รหัสเอกสาร
- ชื่อเอกสาร (ไทย/อังกฤษ)
- คำอธิบาย
- ข้อมูลที่จำเป็น (JSON Schema)
- เอกสารแนบที่ต้องการ
- ระยะเวลาดำเนินการ
- ค่าธรรมเนียม
- เงื่อนไขพิเศษ
- สถานะใช้งาน
```

### 4.2 Workflow States
```
Draft → Submitted → Advisor_Review → Advisor_Approved → Registry_Review → Completed
                                  ↓
                              Advisor_Rejected → Draft (ถ้าแก้ไขได้)
                                               → Cancelled (ถ้าแก้ไขไม่ได้)
```

## 5. Integration Requirements

### 5.1 External Systems
- **Student Information System (SIS)** - ดึงข้อมูลนักศึกษา
- **HR System** - ข้อมูลอาจารย์
- **Payment Gateway** - ชำระค่าธรรมเนียม
- **Email/SMS Service** - การแจ้งเตือน
- **Digital Signature** - การลงนามอิเล็กทรอนิกส์

### 5.2 Authentication
- Single Sign-On (SSO) กับระบบมหาวิทยาลัย
- Multi-factor Authentication สำหรับเจ้าหน้าที่

## 6. Non-Functional Requirements

### 6.1 Performance
- Response time < 3 วินาที
- รองรับผู้ใช้พร้อมกัน 500 คน
- Uptime 99.5%

### 6.2 Security
- ข้อมูลส่วนบุคคลต้องเข้ารหัส
- Access logging และ audit trail
- Role-based access control

### 6.3 Usability
- รองรับภาษาไทยและอังกฤษ
- Responsive design
- Accessibility compliance

## 7. Questions for Further Requirements Gathering

### 7.1 Business Process Questions
1. มีเอกสารประเภทไหนบ้างที่ต้องการในระบบ?
2. แต่ละประเภทเอกสารใช้เวลาดำเนินการนานแค่ไหน?
3. มีค่าธรรมเนียมหรือไม่? คำนวณอย่างไร?
4. มีกรณีที่ต้องผ่านการอนุมัติเพิ่มเติมหรือไม่?
5. การแจ้งเตือนต้องการช่องทางไหนบ้าง?

### 7.2 Technical Questions
1. ระบบปัจจุบันที่มีอยู่แล้วคืออะไรบ้าง?
2. Database ที่ใช้คืออะไร?
3. Infrastructure requirements คืออะไร?

### 7.3 User Experience Questions
1. ผู้ใช้ส่วนใหญ่เข้าถึงระบบผ่านอะไร? (Desktop/Mobile)
2. ช่วงเวลาไหนที่มีการใช้งานเยอะที่สุด?
3. มีการฝึกอบรมผู้ใช้หรือไม่?

## 8. Requirements Validation Checklist

### 8.1 Completeness Check
- [ ] ครอบคลุมทุก stakeholder หรือไม่?
- [ ] ระบุประเภทเอกสารครบถ้วนหรือไม่?
- [ ] Workflow ชัดเจนหรือไม่?

### 8.2 Consistency Check
- [ ] Business rules ขัดแย้งกันหรือไม่?
- [ ] User roles และ permissions สอดคล้องกันหรือไม่?

### 8.3 Feasibility Check
- [ ] Technical requirements ทำได้จริงหรือไม่?
- [ ] Budget และ timeline เพียงพอหรือไม่?
- [ ] Resources เพียงพอหรือไม่?

---

## การดำเนินการขั้นตอนต่อไป:
1. **Workshop กับ Stakeholders** - ยืนยันและเพิ่มเติม requirements
2. **Prototype Creation** - สร้าง mockup/wireframe
3. **Technical Architecture Design** - ออกแบบระบบ
4. **Database Design** - ออกแบบฐานข้อมูล
5. **API Specification** - กำหนด interface
6. **Development Planning** - วางแผนการพัฒนา
