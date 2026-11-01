# Tutorial 6 worksheet: serving patterns

**Name / ชื่อ:** ชรัยย์รัชฏ์ แสวงทรัพย์

**Student ID /** รหัสนิสิต: 6870049221

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง โดยใช้ forked repository ของตัวเอง*

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in Thai or English.

ตอนที่ตอบ ให้นิสิตคัดลอกผลจาก `results/runs.csv` (หรือจาก output ของ load test) ลงในตาราง ตอบสั้นๆ ข้อละ 1–3 ประโยค จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here / copy ผลลัพธ์ของ baseline จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|
| t0-large | 495 | 594 | 93.5% | 100% |
| t0-small | 11 | 67 | 72.5% | 0% |

**Q0.** What does the large model buy, and what does it cost?

*large model ให้อะไรเพิ่ม และต้องแลกด้วยอะไร*

> Large Model ให้ความแม่นยำสูงขึ้น แต่แลกมาด้วยความล่าช้าและการใช้ทรัพยากรมากขึ้น

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 1 จาก `results/runs.csv` มาวางที่นี่)

| Run | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
|---|---|---|---|---|---|---|
| t1-before | 30.3 | 243 | 410 | 21 | 0 | 92.7% |
| t1-after | 9.5 | 409 | 1009 | 0 | 21 | 93.3% |

**Q1.1** In `t1-before`, p50 can look *better* than the baseline. Why is p50 misleading here? Which column tells the real story?

*ใน `t1-before` ค่า p50 อาจดู "ดีกว่า" baseline ทำไม p50 จึงทำให้เข้าใจผิดในกรณีนี้ และ column ไหนบอกความจริง*

> p50 ดูดีกว่าเพราะวัดแค่งานที่เสร็จ แต่ไม่ได้วัดงานที่ล้มเหลว ส่วนคอลัมน์ "errors" และ "pending" บอกความจริงว่ารีวิวบางส่วนล้มเหลวหรือรออยู่

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.

*การค้างเกิดกับ calls ไปยัง large model แค่ 10% ทำไมจึงส่งผลต่อระบบแย่กว่าการที่รีวิว 10% ช้าลงเฉยๆ ลองคิดว่า request ที่รออยู่ถือทรัพยากรอะไรไว้ใน review service และ gateway*

> request ที่ค้างไปบล็อก connection และช่องรับงานของระบบ ทำให้ request อื่นต้องรอไปด้วย จึงไม่ใช่แค่ 10% ที่ช้าลง แต่ส่งผลกระทบต่อทั้งระบบ

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

*รีวิวที่ pending ยังไม่มี label ควรจัดการกับรีวิวเหล่านี้อย่างไร และ topology ใดจาก lecture ที่ทำแบบนั้น*

> รีวิวที่ pending ควรส่งเข้า queue เพื่อให้ worker ประมวลผลซ้ำภายหลัง เมื่อได้ label แล้วค่อยอัปเดตผล โดยใช้ Event-based ทำให้ request หลักไม่ต้องรอการประมวลผล

**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

*เราเลือก timeout 1 วินาที จะเกิดปัญหาอะไรถ้าตั้งเป็น 0.1 วินาที และถ้าตั้งเป็น 10 วินาที*

> 0.1 วินาที สั้นเกินไป ทำให้รีวิวปกติส่วนใหญ่ถูก timeout ทั้งที่โมเดลยังทำงานได้ ส่งผลให้ accuracy ลดลง และถ้าเป็น 10 วินาที นานเกินไป เพราะ request ที่ค้างยังบล็อกทรัพยากรของระบบ ทำให้ปัญหาเดิมกลับมา

## Task 2: Cascade (copy the Task 2 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 2 จาก `results/runs.csv` มาวางที่นี่)

| Run | threshold | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|---|
| t2-0.6 | 0.6 | 5 | 536 | 91.5% | 38% |
| t2-0.7 | 0.7 | 377 | 585 | 93.0% | 53% |
| t2-0.8 | 0.8 | 444 | 569 | 93.5% | 64% |
| t2-0.9 | 0.9 | 474 | 589 | 93.5% | 77% |

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

*จะเลือก threshold ใดสำหรับ deploy ใน CampusEats เพราะอะไร และต้องยอมเสีย quality ใดไป*

> เลือก threshold 0.8 เพราะให้ความแม่นยำสูงที่สุดเท่ากับ 0.9 แต่เรียก large model น้อยกว่า 0.9 และ p95 ดีกว่า 0.7 และ 0.9 โดยยอมเสีย large calls เพิ่มขึ้น และ p50 latency ที่มากกว่า 0.7

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?

*ดูค่า p50 และ p95 ของ threshold ที่เลือก ทำไม cascade จึงช่วย p50 ได้มากกว่า p95 มาก*

> p50 ดีขึ้นมากเพราะ cascade ช่วยให้รีวิวที่ confident ถูก small model จัดการได้เร็ว แต่ p95 ไม่ได้ดีขึ้นมากเพราะรีวิวที่ไม่ confident ยังคงต้องรอ large model ซึ่งช้า

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

*cascade เชื่อ confidence ของ small model ยกตัวอย่างหนึ่งสถานการณ์ที่ไม่ควรเชื่อ และผลที่เกิดกับผู้ใช้*

> ถ้า small model มั่นใจผิดพลาด เช่น รีวิวที่มีคำซับซ้อนหรือมีอารมณ์แฝง ผู้ใช้จะได้รับผลลัพธ์ที่ไม่ถูกต้องทันที เพราะ cascade จะไม่เรียก large model มาตรวจสอบอีกครั้ง

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 3 จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls | cache hits |
|---|---|---|---|---|---|
| t3-cache | 7 | 554 | 93.5% | 40% | 34% |

**Q3.1** Which reviews benefit from the cache, and which never will?

*รีวิวแบบไหนได้ประโยชน์จาก cache และแบบไหนไม่มีวันได้ประโยชน์*

> รีวิวที่ได้ประโยชน์จาก cache คือรีวิวที่เคยถูกประมวลผลแล้ว เพราะ cache จะเก็บผลลัพธ์ไว้ให้เรียกใช้ได้ทันที ส่วนรีวิวที่ไม่เคยเจอมาก่อนจะไม่ได้ประโยชน์จาก cache เลย

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?

*สัปดาห์หน้าจะ deploy large model ตัวใหม่ที่ดีกว่าเดิม ต้องทำอะไรกับ cache และจะออกแบบรองรับเรื่องนี้อย่างไร*

> ต้องเคลียร์แคชออกก่อน เพราะผลลัพธ์ที่เก็บในแคชเป็นของ large model เก่า โดยใส่ version ไว้ใน key ของ cache ให้ผลลัพธ์ที่ได้เป็นไปตาม version ของโมเดลที่เลือก

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.

*ยกตัวอย่างหนึ่งสิ่งที่ได้เรียนรู้จากการวัดผลครั้งนี้ ที่ไม่มีพูดถึงใน lecture*

> ได้เรียนรู้ว่า แคชช่วยให้รีวิวที่เคยประมวลผลแล้วตอบได้เร็วขึ้น แต่รีวิวใหม่ยังต้องรอโมเดลทำงานตามปกติ และเมื่อเปลี่ยนโมเดลก็ต้องจัดการแคชเดิม เพื่อไม่ให้ได้ผลลัพธ์จากโมเดลเก่าด้วย
