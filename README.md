# LabProject3
# แนวทางการทำงาน ESP32 project cook book

## 1. ระบุตัวอย่างที่ใช้ ว่าเอามาจากตัวอย่างไหน
ตัวอย่างที่ใช้เป็นการควบคุม 7-segment display ด้วย ESP32 เพื่อแสดงตัวเลข โดยดัดแปลงเพื่อให้แสดงผลตัวเลขทีละคู่ในลำดับ "65030059" (เป็นรหัสนักศึกษา)
## 2. ระบุว่า จะแก้ไขตรงไหนบ้าง เพื่ออะไร
เพิ่มการจัดเก็บตัวเลข: ใช้ sequence ในรูปแบบของอาร์เรย์ 2 มิติ เพื่อจัดเก็บตัวเลขที่จะถูกแสดงผลเป็นคู่ตามลำดับที่ต้องการ
  ```cpp
int sequence[][2] = {
    {6, 5}, // "65"
    {0, 3}, // "03"
    {0, 0}, // "00"
    {5, 9}  // "59"
};
int sequenceLength = sizeof(sequence) / sizeof(sequence[0]);
 ```

ฟังก์ชันการแสดงผลทีละคู่: เพิ่มฟังก์ชัน displayPair() เพื่อสลับการแสดงผลแต่ละหลักของ 7-segment ให้แสดงคู่ตัวเลขสลับกัน เพื่อให้แสดงผล 2 หลักได้อย่างต่อเนื่อง
 ```cpp
void displayPair(int left, int right) {
    for (int i = 0; i < 200; i++) { // แสดงผลแต่ละคู่หลายครั้งเพื่อให้มองเห็นชัด
        displayDigit(1, left);
        displayDigit(2, right);
    }
}

void loop() {
    for (int i = 0; i < sequenceLength; i++) {
        displayPair(sequence[i][0], sequence[i][1]);
        delay(500); // หน่วงเวลาระหว่างแต่ละคู่ตัวเลข
    }
}
 ```

## 3. แสดงขั้นตอนการทำ project
**ขั้นตอนที่ 1 การต่อวงจร**
1. ต่อวงจรจาก ESP32 ไปยัง 7-segment display ตามขั้นตอนนี้

| 7-segment |ESP32 Pin | 
| -------- | -------- | 
| A | GPIO 12 | 
| B | GPIO 14 | 
| C	| GPIO 27 |
| D |	GPIO 26 |
| E	| GPIO 25 |
| F	| GPIO 33 |
| G	| GPIO 32 |

2. ต่อขา Control ของหลัก

| 7-segment |ESP32 Pin | 
| -------- | -------- | 
| Digit 1 | GPIO 4 | 
| Digit 2 | GPIO 5 | 

**ขั้นตอนที่ 2 การติดตั้งไลบรารีและการตั้งค่าบอร์ด**
1. ติดตั้ง Board ESP32
เปิด Arduino IDE แล้วไปที่ Tools → Board → Boards Manager... แล้วค้นหา "ESP32" แล้วคลิก Install
2. เลือกบอร์ด ESP32:
ไปที่ Tools → Board → เลือก ESP32 Dev Module
3. เลือกพอร์ตการเชื่อมต่อ:
ไปที่ Tools → Port แล้วเลือกพอร์ตที่เชื่อมต่อกับ ESP32

**ขั้นตอนที่ 3 เขียนโค้ด**
 ```cpp
int segA = 12, segB = 14, segC = 27, segD = 26, segE = 25, segF = 33, segG = 32;
int digit1 = 4;
int digit2 = 5;

int digits[10][7] = {
    {1, 1, 1, 1, 1, 1, 0}, // 0
    {0, 1, 1, 0, 0, 0, 0}, // 1
    {1, 1, 0, 1, 1, 0, 1}, // 2
    {1, 1, 1, 1, 0, 0, 1}, // 3
    {0, 1, 1, 0, 0, 1, 1}, // 4
    {1, 0, 1, 1, 0, 1, 1}, // 5
    {1, 0, 1, 1, 1, 1, 1}, // 6
    {1, 1, 1, 0, 0, 0, 0}, // 7
    {1, 1, 1, 1, 1, 1, 1}, // 8
    {1, 1, 1, 1, 0, 1, 1}  // 9
};

int sequence[][2] = {
    {6, 5}, // "65"
    {0, 3}, // "03"
    {0, 0}, // "00"
    {5, 9}  // "59"
};
int sequenceLength = sizeof(sequence) / sizeof(sequence[0]);

void setup() {
    pinMode(segA, OUTPUT);
    pinMode(segB, OUTPUT);
    pinMode(segC, OUTPUT);
    pinMode(segD, OUTPUT);
    pinMode(segE, OUTPUT);
    pinMode(segF, OUTPUT);
    pinMode(segG, OUTPUT);
    
    pinMode(digit1, OUTPUT);
    pinMode(digit2, OUTPUT);
}

void displayDigit(int digit, int number) {
    int* segments = digits[number];
    digitalWrite(segA, segments[0]);
    digitalWrite(segB, segments[1]);
    digitalWrite(segC, segments[2]);
    digitalWrite(segD, segments[3]);
    digitalWrite(segE, segments[4]);
    digitalWrite(segF, segments[5]);
    digitalWrite(segG, segments[6]);
    digitalWrite(digit == 1 ? digit1 : digit2, HIGH);
    delay(5);
    digitalWrite(digit == 1 ? digit1 : digit2, LOW);
}

void displayPair(int left, int right) {
    for (int i = 0; i < 200; i++) { 
        displayDigit(1, left);
        displayDigit(2, right);
    }
}

void loop() {
    for (int i = 0; i < sequenceLength; i++) {
        displayPair(sequence[i][0], sequence[i][1]);
        delay(500); 
    }
}
 ```

**ขั้นตอนที่ 4 การอัปโหลดโค้ดไปยัง ESP32**
1. เชื่อมต่อ ESP32 กับคอมพิวเตอร์โดยใช้สาย USB
2. คลิกที่ Upload ใน Arduino IDE เพื่ออัปโหลดโค้ดไปยัง ESP32
3. รอจนกระทั่งการอัปโหลดเสร็จสิ้น 

## 4. แสดงผลการทำ project
มื่อโปรแกรมทำงานเสร็จสมบูรณ์ ตัวเลข "65030059" จะถูกแสดงผลบน 7-segment display ทีละคู่ในลำดับดังนี้:

แสดงผล "65": ตัวเลข "6" และ "5" จะถูกแสดงบนหลักที่ 1 และหลักที่ 2
แสดงผล "03": ตัวเลข "0" และ "3" จะถูกแสดงบนหลักที่ 1 และหลักที่ 2
แสดงผล "00": ตัวเลข "0" และ "0" จะถูกแสดงบนหลักที่ 1 และหลักที่ 2
แสดงผล "59": ตัวเลข "5" และ "9" จะถูกแสดงบนหลักที่ 1 และหลักที่ 2


https://github.com/user-attachments/assets/4575a1b2-2c0f-4f24-966c-e2b6498193d1



## 5. สรุปผลการทำ project
โปรเจคนี้แสดงให้เห็นถึงการควบคุมการแสดงผลของ 7-segment display ที่สามารถแสดงตัวเลขหลายหลักได้ โดยการแบ่งการแสดงผลออกเป็น 2 หลักทีละคู่ ทำให้ดูเหมือนว่าแสดงตัวเลข "65030059" แบบต่อเนื่องและสลับกันในลำดับที่กำหนด
