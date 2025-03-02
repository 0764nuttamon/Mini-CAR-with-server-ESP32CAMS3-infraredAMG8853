โปรเจกต์นี้เป็นการใช้ ESP32-CAM
-  เพื่อสตรีมภาพวิดีโอ (MJPEG) ผ่านเครือข่าย Wi-Fi
-  ควบคุมมอเตอร์ (รถหรือหุ่นยนต์) ผ่านหน้าเว็บ
-  อ่านข้อมูล เซ็นเซอร์ตรวจจับความร้อน (AMG88xx)
-  แล้วสตรีมข้อมูลด้วย Server-Sent Events (SSE)
-  ตกแต่งหน้าเว็บ สไตล์บาร์บี้ (สีชมพูหวาน) พร้อมภาพพื้นหลังจาก wallpapercave.com
-  แสดง “กล่องเตือน” เมื่อค่าเฉลี่ยอุณหภูมิ ≥ 37°C


**การแยกเซิร์ฟเวอร์**
Server A (พอร์ต 80): ให้บริการหน้าเว็บหลัก (HTML) + Handler /action สำหรับควบคุมมอเตอร์
Server B (พอร์ต 81): สตรีมภาพ MJPEG จากกล้อง ESP32-CAM
Server C (พอร์ต 82): สตรีมค่าเซ็นเซอร์ AMG88xx ด้วย SSE (text/event-stream)

**หน้าเว็บหลัก**
ตกแต่งธีมสีชมพู พร้อมภาพพื้นหลัง
ส่วนควบคุม: ปุ่ม Forward, Backward, Left, Right, Stop (เชื่อม /action?go=...)

**ส่วนแสดงผลเซ็นเซอร์:**
ตาราง 8x8 แสดงอุณหภูมิเป็น “ฟ้า/เขียว/แดง” ตามเงื่อนไข
ถ้าอุณหภูมิเฉลี่ย (ทั้ง 64 จุด) ≥ 37°C จะแสดงกล่องข้อความ
“อุณหภูมิสูงขึ้น อาจเจอสิ่งมีชีวิต!!” ข้าง ๆ หัวข้อ “Temperature Data”

**การตั้งค่ากล้อง**
ใช้โค้ด esp_camera_init เพื่อกำหนด GPIO และรูปแบบภาพ (JPEG)
เลือกขนาดเฟรม (เช่น FRAMESIZE_VGA) และคุณภาพ (jpeg_quality)

**การเชื่อมต่อ Wi-Fi**
ระบุ SSID, password ในตัวแปร ssid และ password
ถ้าเชื่อมต่อไม่สำเร็จใน 10 วินาที จะไม่เริ่มเซิร์ฟเวอร์

**การอ่านเซ็นเซอร์ AMG88xx**
ใช้ไลบรารี Adafruit_AMG88xx
กำหนดขา SDA, SCL, Address (0x68 หรือ 0x69)
อ่านค่า 8x8 = 64 จุด แล้วส่งเป็น JSON ผ่าน SSE

**เทคนิคที่ใช้**
Server-Sent Events (SSE) เพื่ออัปเดตตารางอุณหภูมิแบบเรียลไทม์
MJPEG Streaming (multipart/x-mixed-replace) สำหรับกล้อง
HTTP GET เพื่อสั่งมอเตอร์แบบ /action?go=forward, /stop, ฯลฯ
CSS Styling: ตกแต่งธีมบาร์บี้, ปุ่ม, ตาราง, ภาพพื้นหลัง

**วิธีใช้งาน (โดยย่อ)**
แก้ไข SSID และ Password ในตัวแปร
const char* ssid = "Pony";
const char* password = "knuttamon0002";
อัปโหลดโค้ดไปยังบอร์ด ESP32-CAM (เช่น AI-Thinker ESP32-CAM)
เปิด Serial Monitor เพื่อดู IP ที่ได้รับ เช่น 192.168.1.100
เปิดเว็บเบราว์เซอร์ ไปที่ http://192.168.1.100/
จะเห็นหน้าเว็บธีมสีชมพู มีพื้นหลังจาก URL ภายนอก
มีปุ่ม Forward, Backward, Left, Right, Stop ควบคุมมอเตอร์
ภาพ MJPEG สตรีมจาก http://192.168.1.100:81/stream
ตารางอุณหภูมิเรียลไทม์จาก http://192.168.1.100:82/sensorStream
ถ้าเซ็นเซอร์อ่านค่าเฉลี่ย ≥ 37°C จะแสดงกล่องเตือน “อุณหภูมิสูงขึ้น อาจเจอสิ่งมีชีวิต!!”

**ข้อควรระวัง / หมายเหตุ**
AMG88xx ต้องการไฟ 3.3V และเชื่อม I2C ถูกต้อง (SDA=47, SCL=14 ตามโค้ด)
ภาพพื้นหลังดึงจาก URL ภายนอก (wallpapercave.com) หากลิงก์เสียหรือเน็ตไม่เสถียร ภาพอาจไม่ขึ้น
ถ้าต้องการความเร็วสตรีมมากขึ้น หรือให้มอเตอร์ตอบสนองไว อาจปรับลด vTaskDelay(...) ในลูปสตรีม
หากเปิดใช้งาน HTTPS อาจเกิดปัญหา Mixed Content ถ้าภาพพื้นหลังหรือ SSE ไม่ได้อยู่บน HTTPS

