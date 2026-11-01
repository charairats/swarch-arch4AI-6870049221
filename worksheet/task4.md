# Task 4: Deployment topologies and trade-offs

**Name / ชื่อ:** ชรัยย์รัชฏ์ แสวงทรัพย์

**Student ID /** รหัสนิสิต: 6870049221

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง ใน fork ของตัวเอง*

**Scenario (A, B or C) / Scenario ที่เลือก:**

Choose **one** scenario below. If the last digit of your student ID is odd, take A or B; if it is even, take B or C. Use your measurements from Tutorial 6 as evidence: cite run names from `results/runs.csv`. Answer in Thai or English.

เลือก scenario **หนึ่ง** ข้อด้านล่าง ถ้าเลขตัวสุดท้ายของรหัสนิสิตเป็นเลขคี่ ให้เลือก A หรือ B ถ้าเป็นเลขคู่ ให้เลือก B หรือ C ใช้ผลการวัดจาก tutorial ที่ทำไปเป็นข้อมูลอ้างอิง โดยอ้างชื่อ run จาก `results/runs.csv` จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

Facts that hold in every scenario / ข้อเท็จจริงที่ใช้กับทุก scenario:

- The large model serves at most **20 calls per second** (4 calls at once × about 200 ms each), and every call costs money.

  *large model รับได้สูงสุด **20 calls ต่อวินาที** (พร้อมกัน 4 calls × ประมาณ 200 ms ต่อ call) และทุก call มีค่าใช้จ่าย*
- The small model is fast and cheap enough to run anywhere, including inside the review service or on a kiosk, but it is much less accurate on its own.

  *small model เร็วและถูกพอที่จะรันที่ไหนก็ได้ รวมถึงภายใน review service หรือบน kiosk แต่ถ้าใช้ตัวเดียวจะแม่นน้อยกว่ามาก*

## Scenarios

**A. Owner alerts.** When a customer posts a negative review, the restaurant owner gets a push notification within **2 seconds**. At the lunch peak, CampusEats receives **40 reviews per second**. A missed alert is acceptable, but a wrong alert (for example, "your customer is unhappy" when they are happy) makes owners stop trusting alerts and, in the end, ignore them.

*เมื่อลูกค้าโพสต์รีวิวเชิงลบ เจ้าของร้านจะต้องได้รับ push notification ภายใน **2 วินาที** เวลาช่วงพีคตอนเที่ยง CampusEats ได้รับรีวิวจำนวน **40 รายการต่อวินาที** การพลาด alert ไปบ้างนั้นยอมรับได้ แต่ alert ที่ผิด (เช่น "ลูกค้าไม่พอใจ" ทั้งที่จริงๆ แล้วพอใจ) จะทำให้เจ้าของร้านไม่เชื่อถือและเลิกสนใจ alert ในที่สุด*

**B. Canteen kiosk.** A tablet in each canteen lets students leave a review on the spot. However, the canteen Wi-Fi is poor: the tablets lose their connection several times a day, for minutes at a time. The system must accept every review, even while a tablet is offline. Restaurant owners need the labels **by the end of the day**.

*มี tablet ในแต่ละโรงอาหารให้นิสิตรีวิวได้ทันที อย่างไรก็ดี การเชื่อมต่อ Wi-Fi ในโรงอาหารไม่ดี ทำให้ tablet หลุดการเชื่อมต่อวันละหลายครั้ง ครั้งละหลายนาที ระบบจะต้องรับรีวิวได้ทุกรายการแม้ขณะที่กำลัง offline อยู่ และเจ้าของร้านต้องการ label **ภายในสิ้นวัน***

**C. Weekly report.** Restaurant owners want a weekly sentiment report. Nothing in producing this report has to be real-time. **Accuracy matters most** and **cost must be minimal**.

*เจ้าของร้านต้องการรายงาน sentiment รายสัปดาห์ ไม่มีอะไรที่จะต้องทำแบบ real-time ในการออกรายงานนี้ ความแม่นยำ **(accuracy) สำคัญที่สุด** และ **ค่าใช้จ่ายต้องน้อยที่สุด***

## Topologies and patterns to choose from / Topology และ pattern ที่เลือกใช้ได้

| | Topology | Where the model runs / โมเดลรันที่ไหน |
|---|---|---|
| T1 | Model as a service, synchronous | Inference service, called for each review (today's baseline)<br>*inference service ถูกเรียกทุกรีวิว (baseline ของวันนี้)* |
| T2 | Model as a library | Inside the review service, or on the device<br>*ภายใน review service หรือบนอุปกรณ์* |
| T3 | Hybrid | Small model locally, large model as a remote service<br>*small model รันในเครื่อง large model เป็น remote service* |
| T4 | Asynchronous / batch | Reviews are queued (broker, W10) or stored, and scored later in bulk<br>*รีวิวถูกเข้าคิว (broker, W10) หรือเก็บไว้ แล้วติด label ทีหลังเป็นชุด* |

Combine one topology with any of the patterns from Tutorial 6: **fallback**, **cascade** and **cache**.

*เลือก topology หนึ่งแบบ แล้วผสมกับ pattern จาก Tutorial 6 ได้ตามต้องการ: **fallback**, **cascade** และ **cache***

## 1. Quality requirements

The three most important quality requirements for your scenario, each with a number where possible.

*quality requirements ที่สำคัญที่สุด 3 ข้อของ scenario ที่เลือก ระบุตัวเลขเมื่อทำได้*

| # | Quality | Requirement |
|---|---|---|
| 1 | Availability | The system shall receive all reviews even when the client is offline |
| 2 | Reliability | The system shall ensure that all reviews are recorded |
| 3 | Timeliness | The system shall label all reviews by the end of the day |


## 2. Design / การออกแบบ

Your topology, the patterns you add, and a container-level diagram (C4 level 2). The C4 diagram must be written in Mermaid.

*topology ที่เลือก pattern ที่เพิ่ม และ diagram ระดับ container (C4 level 2) เขียนด้วย Mermaid*

> เลือก T4 Asynchronous / batch กับ cascade และ fallback

```mermaid
C4Container
    title Canteen kiosk: T4 Asynchronous/batch + Cascade + Fallback

    Person(student, "Student", "สร้างรีวิวบน tablet")
    Person(owner, "Restaurant owner", "ดูรีวิวและ label ของร้านตัวเอง")

    System_Boundary(frontend, "Frontend - Tablet") {
        Container(tablet, "Tablet app", "On tablet", "สร้างรีวิวและดูรีวิว")
        ContainerDb(outbox, "Outbox Sync", "Local storage", "เก็บรีวิวเมื่อ Offline และส่งเมื่อ Wi-Fi กลับมา")
    }

    System_Boundary(backend, "Backend - Server") {
        Container(gateway, "API gateway", "Python, FastAPI", "รับรีวิวและ request จาก Tablet")
        Container(review, "Review service", "Python, FastAPI", "บันทึกและอ่านรีวิวกับ label")
        ContainerDb(reviewdb, "Review DB", "Database", "เก็บรีวิว สถานะ และ label")
        Container(batch, "Batch job", "Scheduled job", "ทุก 1 ชม. หรือครบ 1,000 รายการ รอบสุดท้าย 23:00 น.")
        ContainerQueue(queue, "Message queue", "Message broker", "พักรีวิวที่รอติด label")
        Container(inference, "Inference service", "Python, FastAPI", "Cascade threshold 0.7; timeout 10s หรือ error ให้คง pending เพื่อ retry")
    }

    Rel(student, tablet, "สร้างรีวิว")
    Rel(owner, tablet, "ดูรีวิวและ label")
    Rel(tablet, outbox, "บันทึกรีวิวในเครื่อง")
    Rel(tablet, gateway, "ขอดูรีวิวและ label", "HTTP/JSON")
    Rel(outbox, gateway, "Sync รีวิวเมื่อ Online", "HTTP/JSON")
    Rel(gateway, review, "ส่งต่อ request", "HTTP/JSON")
    Rel(review, reviewdb, "บันทึกและอ่านรีวิว")
    Rel(batch, reviewdb, "กวาดรีวิว pending และบันทึก label")
    Rel(batch, queue, "ส่งรีวิวเข้าคิว")
    Rel(queue, inference, "ส่งรีวิวเพื่อติด label")
    Rel(inference, batch, "คืนผล label หรือ timeout/error")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="2")
```

## 3. Evidence / หลักฐาน

Which runs support your choice? Show the numbers and, for scenario A, the capacity arithmetic: how many large-model calls per second does your design need at the peak?

*run ไหนที่สนับสนุนการตัดสินใจของนิสิต ให้นิสิตแสดงตัวเลขประกอบ และสำหรับ scenario A ให้นิสิตแสดงการคำนวณ capacity ด้วย เพื่อแสดงว่าในช่วงพีคนั้น design ของนิสิตต้องใช้ large model กี่ calls ต่อวินาที*

| Run | What it shows / แสดงอะไร |
|---|---|
| t2-0.7 | Cascade threshold 0.7 ให้ Accuracy 93.0% และเรียก Large Model 53% ช่วยลดจำนวน Calls ลง 47% เมื่อเทียบกับ Baseline โดย Accuracy ลดลงเพียง 0.5 percentage points |
| baseline | ถ้าไม่ใช้ cascade จะต้องเรียก large model ที่ 100% |
| t0-small | การใช้แต่ small model มีความแม่นยำต่ำ แค่ 72.5% จึงไม่สามารถใช้แต่ small model ได้ |
| t1-after | เรามีการแยกการบันทึกออกจากการติด label อยู่แล้ว ข้อมูลจะไม่หายไป กรณีล้มเหลวเป็น 0 แต่แลกกับระยะเวลานานขึ้น เพื่อตอบเรื่อง reliability ตาม requirement ที่ review ต้องไม่หายไป |

## 4. ADR

Use the ADR format from W5.

*ใช้รูปแบบ ADR จาก W5*

**Title:** Asynchronous batch labeling (T4) with a 0.7 cascade for the canteen kiosk

**Status:** Proposed

**Context:** Tablet ในโรงอาหารมี Wi-Fi ไม่เสถียร ระบบต้องรับรีวิวได้แม้ Offline และติด Label ภายในสิ้นวัน (24:00) โดย Large Model รองรับสูงสุด 20 calls/s การทดลอง t2-0.7 ให้ Accuracy 93.0% และใช้ Large Model เพียง 53% เทียบกับ Baseline 100%

**Decision:** เลือก T4 Asynchronous/Batch + Cascade (threshold 0.7) + Fallback เมื่อ Timeout/Error โดยเก็บงานเป็น Pending เพื่อ Retry


**Consequences:**
- Positive: รีวิวไม่หายเมื่อ Wi-Fi หลุด เพราะเก็บใน Outbox และ Sync เมื่อกลับมา Online อีกทั้งลด Large Model Calls ได้ 47% จาก Baseline
- Negative: Label ไม่ออกทันที ต้องรอ Batch สูงสุดประมาณ 1 ชั่วโมงตามรอบปกติ หาก Wi-Fi ไม่กลับมาก่อนสิ้นวัน รีวิวจะค้างใน Outbox และต้องยอมประมวลผลวันถัดไป นอกจากนี้ Accuracy ลดลงจาก 93.5% เหลือ 93.0%

**Alternatives considered:** at least one topology you rejected, and why.

*อย่างน้อยหนึ่ง topology ที่ไม่เลือก และเหตุผล*

> T3 (Hybrid): Small Model บน Tablet ให้ Label ได้ทันทีแม้ Offline แต่ไม่จำเป็น เพราะ Requirement ยอมให้ประมวลผลถึงสิ้นวัน อีกทั้งรีวิวประมาณ 53% (t2-0.7) ยังต้องส่งไป Large Model ผ่านเครือข่าย จึงยังต้องมี Outbox และรอ Wi-Fi เหมือนเดิม นอกจากนี้ยังเพิ่มภาระ Maintenance เพราะต้อง Deploy และอัปเดต Model บน Tablet ทุกเครื่อง แทนที่จะดูแลจาก Server ส่วนกลาง
