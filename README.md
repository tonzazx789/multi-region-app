โปรเจกต์นี้คือการสร้าง เว็บแอปแบบ Multi-Region บน AWS
โดยเอาแอปเดียวกันไป deploy ไว้ หลาย AWS Region (เช่น us-east-1 กับอีก region หนึ่ง) เพื่อให้:
   - ถ้า Region หลักล่ม → ยังมีอีก Region สำรอง
   - รองรับ Disaster Recovery (DR)
   - เพิ่ม High Availability
   - ทดสอบว่า region ไหนตอบสนองเร็วกว่า (Latency ต่ำกว่า)
   - ใช้สำหรับฝึกแนวคิด Failover Architecture แบบของจริงบน AWS

      เป้าหมายหลักของโปรเจกต

     1. Deploy แอปเดียวกันไปหลาย Region
     2. ให้แต่ละ Region ทำงานแยกกันได้
     3. แสดงผลว่า request มาจาก Region ไหน
     4. วัด latency ระหว่างหลาย endpoint เพื่อเตรียมทำ failover

      โครงสร้างโปรเจกต์
     1) index.js → Backend (Express Server) หน้าที่ของไฟล์นี้คือ:

        1.สร้างเว็บเซิร์ฟเวอร์ด้วย Express
        2.เปิดพอร์ต PORT (ค่า default = 8080)
        3.อ่านค่า AWS_REGION จาก environment variable
        4.ใส่ CORS headers
        5.เสิร์ฟไฟล์ static เช่น index.html
    2) index.html → Frontend สำหรับวัด Latency ไฟล์นี้เอาไว้:

        1.กำหนด list ของ endpoint หลาย region
        2.ยิง fetch() ไปแต่ละ URL
        3.วัดเวลาตอบสนองด้วย performance.now()
        4.แสดงผลว่าแต่ละ region ใช้กี่ ms
    3) package.json → การรันโปรเจกต์ ตรงนี้มี dependency หลักคือ:
        -express และกำหนดว่าใช้ Node.js >=18

        โปรเจกต์นี้เกี่ยวกับ Disaster Recovery ยังไง?
            แนวคิด DR ของโปรเจกต์นี้
            Disaster Recovery = ถ้า region หนึ่งมีปัญหา ระบบยังไม่ตายทั้งระบบ
            เช่น:
              Region A = us-east-1
              Region B = ap-southeast-1
            ถ้า us-east-1 ล่ม:
              -ผู้ใช้ยังเข้าผ่าน Region B ได้
              -latency อาจสูงขึ้นหรือลดลงตามตำแหน่งผู้ใช้
              -ระบบยังออนไลน์อยู่
            นี่คือพื้นฐานของ Multi-Region Resilience ที่ AWS ใช้ในงานจริง
            และมักใช้คู่กับ Route 53 health checks / failover routing ในระบบ production



