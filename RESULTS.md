# K6 lab results

## Part 0

- Step 0.1, `http_req_duration`: 286.82 ms (one request; avg, min, med, max, p(90), and p(95) are equal). The status 200 check passed (1/1), and `http_req_failed` was 0%.
- Step 0.2, 5 VUs for 20 seconds: 80 HTTP requests, 3.787753 requests/second, and `http_req_duration` p(95) of 331.69 ms. All 80 checks passed, with 0% failed requests. The request rate stayed below 5/second because each user waits for the HTTP response and then sleeps for 1 second before starting the next iteration; one user cannot make one request every second.
- Step 0.3, 5 VUs sharing 20 iterations: exactly 20 iterations and 20 HTTP requests completed in 5.7 seconds. All 20 checks passed, with 0% failed requests. Unlike Step 0.2, this run stopped when the fixed amount of work finished.
- Step 0.4, vegetarian pizza API: all three checks passed (status 200, a pizza name, and only vegetarian ingredients). `http_req_duration` was 304.61 ms, and `http_req_failed` was 0%.
- Step 0.5, request without a token: QuickPizza returned the expected 401 status. The check passed (1/1), and `http_req_failed` stayed at 0% because the script marks 401 as an expected response.
- Step 0.6, default `P95=1500`: all three thresholds passed; measured p(95) was 388.58 ms, checks were 100%, failed requests were 0%, and the exit code was 0. With `P95=300`, the response-time threshold failed because p(95) was 408.31 ms; checks were still 100% and failed requests 0%, but the exit code was 99.
- Step 0.7, requirement run with 3 VUs for 30 seconds: `p(95)<1000` passed at 394.36 ms, `p(99)<1500` passed at 423.61 ms, and `http_req_failed rate<0.01` passed at 0%; exit code 0. In a second run, I changed only the p(95) limit from 1000 ms to 100 ms using `-e P95=100`. The measured p(95) was 405.59 ms, so that rule failed while the p(99) and failure-rate rules passed; exit code 99. The script's default thresholds remain the original requirement.
- Step 0.8, initial run with 5 VUs for 20 seconds: 75 pizza responses had status 200, but 13 `no pepperoni` checks and 34 `no knife needed` checks failed (47 of 225 checks failed overall). The API exploration showed the exact names `Pepperoni` and `Knife`; the request had used lowercase `pepperoni` and `knife`, so the exclusions did not match. After fixing the capitalization, all 225 checks passed across 75 responses, with 0% failed HTTP requests. Repeated requests under load sampled different recommendations, exposing unwanted pizzas that one random manual response might miss.

## Part A results

| Step | Test type | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | --------- | ----- | ----- | ----- | --- | ------ | ------ |
| A1 | Smoke | 1 | 0.545477 | 359.51 ms | 368.73 ms | 0% | 100% (24/24) |
| A2 | Average load | 3 | 1.308686 | 376.95 ms | 420.84 ms | 0.41% (1/243) | 99.38% (322/324) |
| A3 | Stress | 5 | 2.17773 | 369.3 ms | 442.94 ms | 0% (0/408) | 100% (544/544) |
| A4 | Spike | 5 | 1.039496 | 369.7 ms | 447.54 ms | 0% (0/150) | 100% (200/200) |
| A5 | Soak | 2 | 0.96244 | 384.9 ms | 458.32 ms | 0% (0/636) | 100% (848/848) |

Step A2: All thresholds passed, including home p(95) 313.19 ms and recommendation p(95) 394.9 ms. One recommendation request did not return 200 and had no pizza name, causing two failed checks. The HTML dashboard is in `reports/a2-average.html`.

Step A3: All thresholds passed, including home p(95) 314.05 ms and recommendation p(95) 403.76 ms. Overall p(95) was 369.3 ms, 7.65 ms lower than A2's 376.95 ms, despite the higher request rate. The small difference in these separate runs does not show a clear slowdown at 5 VUs. The HTML dashboard is in `reports/a3-stress.html`.

Step A4: All thresholds passed, and no requests or checks failed. The spike began at 30 seconds, reached 5 VUs at 35 seconds, and returned to 1 VU at 70 seconds. In the dashboard's 10-second windows, overall p(95) was about 305-355 ms before the spike, around 355-370 ms while at 5 VUs, and about 405 ms in the 70-80 second window. It was 304 ms in the 80-90 second window, so the response time returned to its earlier range within about 20 seconds after the drop to 1 VU. These short-window values vary, so this is an approximate recovery time. The HTML dashboard is in `reports/a4-spike.html`.

Step A5: All thresholds passed. The run completed 212 journeys, 636 requests, and 848 checks with no failed requests or checks. For a like-for-like comparison at 2 VUs, the mean of the dashboard's 10-second p(95) values was 368.28 ms in the early 30-150 second interval and 372.14 ms in the late 510-630 second interval. This 3.86 ms difference is small, with no clear degradation over the 10-minute hold and no increase in the error rate. The HTML dashboard is in `reports/a5-soak.html`.

## Part B

1. ผล A1-A5 อยู่ในตารางข้างบนเลย มีทั้งจำนวน users, Req/s, p(95), Max, อัตราคำขอล้มเหลว และ checks จากการรันจริง
2. Step 0.2 ใช้ 5 users แต่ได้แค่ 3.787753 requests/s เพราะแต่ละคนต้องรอเว็บตอบก่อน แล้ว `sleep(1)` อีก 1 วินาที ถึงจะเริ่มรอบใหม่ได้ เลยส่งได้ไม่ถึงคนละ 1 request/s
3. รอบ stress ไม่ได้ช้าลงชัดเจน p(95) ของ A2 คือ 376.95 ms ส่วน A3 คือ 369.3 ms (ต่ำกว่า 7.65 ms) โหลดระดับ 5 users น่าจะยังอยู่ในช่วงที่ระบบรับไหว แต่เป็นเว็บสาธารณะและรันคนละเวลา เลยเอาความต่างเล็กน้อยนี้ไปฟันธงเรื่องขีดจำกัดจริงของระบบไม่ได้
4. หลัง spike ลดกลับมา 1 user ที่ประมาณวินาที 70 ค่า p(95) ช่วงวินาที 70-80 ยังสูงราว 405 ms แต่ช่วง 80-90 ลดเหลือราว 304 ms ใกล้ก่อน spike แล้ว เลยประเมินว่ากลับมาปกติภายในประมาณ 20 วินาที
5. รอบ soak ไม่เห็นว่าแย่ลงชัดเจน ค่าเฉลี่ย p(95) จากช่วงต้นกับท้ายที่โหลด 2 users คือ 368.28 กับ 372.14 ms ต่างกัน 3.86 ms และคำขอล้มเหลว 0/636 ถ้ารันหลายชั่วโมงอาจเจอ memory leak ที่ค่อย ๆ กินแรมจนตอบช้าหรือล่มได้
6. ใช้ p(95) เพราะมันบอกว่าคำขอ 95% เสร็จภายในกี่ ms ทำให้เห็นกลุ่มที่ช้ากว่าคนส่วนใหญ่ ค่าเฉลี่ยอย่างเดียวอาจกลบประสบการณ์ของคนที่เจอคำขอช้า
7. ตัวเลขในแล็บอาจต่างจากการยิงระบบในห้องแล็บ เพราะ (1) ระยะทางและความหน่วงของอินเทอร์เน็ตต่างกัน และ (2) QuickPizza เป็นเว็บที่คนอื่นใช้ร่วมกัน โหลดจากคนอื่นเปลี่ยนได้ตลอด
8. การยิง stress, spike หรือ breakpoint หลายร้อย users ใส่เว็บที่เราไม่ได้เป็นเจ้าของอาจทำให้เว็บช้าหรือล่ม กระทบผู้ใช้จริง และเราไม่ได้รับอนุญาตให้สร้างโหลดระดับนั้น
9. ตัวอย่างแอปสั่งอาหารที่ใช้ประจำ: ข้อกำหนดกว้าง ๆ คือ "หน้าแรกต้องโหลดเร็ว" ถ้าจะวัดจริง ขอเป็น "เมื่อมี 3 users เปิดหน้าแรกพร้อมกันนาน 30 วินาที 95% ของคำขอต้องเสร็จภายใน 1 วินาที 99% ภายใน 1.5 วินาที และคำขอล้มเหลวต่ำกว่า 1%" โดยตั้ง `vus: 3`, `duration: '30s'` และ thresholds เป็น `http_req_duration: ['p(95)<1000', 'p(99)<1500']`, `http_req_failed: ['rate<0.01']`
