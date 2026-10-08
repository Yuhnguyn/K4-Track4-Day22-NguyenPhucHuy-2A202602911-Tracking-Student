# Báo cáo lab: chọn tracker cho 5 video

**Nhóm:** K4-Track4-Day22 **Thành viên:** Nguyễn Phúc Huy (2A202602911)

Detector cố định: `yolo26n.pt`, ảnh 640 px, Re-ID `osnet_x0_25_msmt17`. Không đổi các mục này trong bài nộp chính.

## 1. Cấu hình đã chọn

Mỗi video: tracker bạn nộp, `conf`, `iou`, điều bạn **nhìn thấy** trên video, và một cấu hình đã thử rồi loại.

| Video | Tracker | conf | iou | Quan sát khi xem video | Đã thử nhưng loại |
|---|---|---|---|---|---|
| video_1 (quảng trường, tĩnh, ban ngày) | botsort | 0.3 | 0.7 | Cảnh tĩnh, mật độ vừa. BoT-SORT (có Re-ID) giữ ID ổn khi hai người đi cắt nhau, ít đổi màu ID. NMS `iou 0.7` giữ được các hộp người đứng gần nhau. Recall còn thấp vì detector `conf 0.3` bỏ sót người nhỏ/xa. | bytetrack (HOTA 26.9), deepocsort (HOTA 27.4) thấp hơn; `conf 0.5` làm HOTA tụt còn 27.2 |
| video_2 (phố đêm, tĩnh, rất đông) | bytetrack | 0.3 | 0.5 | Camera tĩnh nên mô hình chuyển động đáng tin; ban đêm ngoại hình kém nên Re-ID không lợi. ByteTrack cho ít ID hơn và gần như không có track ngắn (16 ID, 0 mảnh vụn) so với botsort (23 ID, 3 mảnh vụn). | botsort: nhiều ID/mảnh vụn hơn ở cảnh tối, đông |
| video_3 (camera di động, ảnh nhỏ) | botsort | 0.3 | 0.5 | Camera tự di chuyển làm "vận tốc" của tracker chuyển động sai; Re-ID nối lại người dù nền đổi. | bytetrack: dựa vị trí, mất ID khi cả khung hình dịch chuyển |
| video_4 (trong nhà, camera di chuyển) | botsort | 0.3 | 0.5 | Camera tiến tới + kính phản chiếu → vị trí/vận tốc không đáng tin, cần ngoại hình để giữ ID và loại "người ma" trong gương. | bytetrack: dựa vị trí, dễ đổi ID khi camera tiến |
| video_5 (trên xe bus, giao lộ đông) | botsort | 0.3 | 0.5 | Rung lắc làm vận tốc nhiễu; Re-ID ổn định hơn khi cả nền chuyển động và người cắt nhau ở giao lộ đông. | bytetrack: rung + đông → khớp vị trí kém, dễ đổi ID |

## 2. Số liệu video_1

Dán bảng HOTA / MOTA / IDF1 do `scripts/evaluate_practice.py` in ra.

Cấu hình nộp `video_1`: **botsort**, `--conf 0.3`, `--iou 0.7` (chạy đủ 600 frame).

| Metric | HOTA | DetA | AssA | MOTA | IDF1 | IDSW | FP | FN |
|---|---|---|---|---|---|---|---|---|
| **video_1** | **29.969** | 18.408 | 49.061 | 19.025 | 29.703 | 33 | 611 | 14402 |

**So sánh tracker trên `video_1`** (conf 0.3, iou 0.5, full):

| Tracker | HOTA | MOTA | IDF1 |
|---|---|---|---|
| bytetrack | 26.912 | 17.292 | 25.713 |
| ocsort | 27.455 | 19.811 | 28.733 |
| **botsort** | **29.460** | **19.811** | 29.354 |
| strongsort | 28.658 | 19.698 | 29.854 |
| deepocsort | 27.380 | 19.762 | 27.798 |

**Quét tham số trên BoT-SORT** (full 600 frame):

| conf | iou | HOTA | DetA | AssA | MOTA | IDF1 |
|---|---|---|---|---|---|---|
| 0.15 | 0.5 | 29.343 | 19.236 | 45.113 | 20.731 | 29.561 |
| 0.15 | 0.7 | 29.664 | 19.434 | 45.625 | 20.327 | 29.878 |
| 0.30 | 0.5 | 29.460 | 18.095 | 48.223 | 19.811 | 29.354 |
| 0.30 | 0.4 | 29.316 | 17.361 | 49.632 | 19.466 | 29.823 |
| **0.30** | **0.7** | **29.969** | 18.408 | 49.061 | 19.025 | 29.703 |
| 0.50 | 0.5 | 27.167 | 14.303 | 51.649 | 15.252 | 24.558 |

Kết luận: HOTA cao nhất ở `conf 0.3, iou 0.7`. Hạ `conf` làm tăng DetA/recall nhưng tăng FP; tăng `conf` giữ ID tốt hơn nhưng bỏ sót nhiều.

`video_2` đến `video_5` không có nhãn trong gói lab. Không điền số cho các video đó.

## 3. Phân tích

Với **ít nhất hai video** (nên gồm một video bạn chỉ đánh giá bằng mắt), viết 3–5 câu:

- Tracker đã chọn giữ ID tốt hơn, hay ít hộp giả hơn, ở điểm nào bạn nhìn thấy?
- Cảnh đó (đứng yên / chuyển động, đông / thưa, sáng / tối, trong nhà / ngoài trời) khiến tracker này hợp hơn tracker kia như thế nào?

### video_1 (quảng trường, camera tĩnh, ban ngày)

Chọn BoT-SORT vì đây là tracker Re-ID thắng HOTA (29.46) so với bytetrack (26.91) và deepocsort (27.38) ở cùng ngưỡng, và có AssA cao (48.22) — nghĩa là **giữ danh tính tốt hơn** khi người đi cắt nhau. Ở cảnh camera **đứng yên, ban ngày, mật độ vừa**, các tracker chuyển động đã đủ tốt về vị trí, nên phần tăng điểm đến từ khả năng **nhận lại đúng người sau khi bị che khuất** — đúng thế mạnh của Re-ID.

Nút thắt của cảnh này **không nằm ở tracker mà ở detector**: mọi tracker đều có `DetA ≈ 15–18` và bỏ sót ~78% nhãn (recall thấp). Vì vậy tôi quét `conf`: hạ xuống 0.15 tăng recall (DetA 19.24) nhưng đưa vào nhiều hộp giả hơn (FP 505); nâng lên 0.5 thì sạch (FP 229) nhưng bỏ sót nặng (MOTA tụt còn 15.25). Quét `iou` cho thấy NMS lỏng (0.7) giữ được người đứng gần nhau tốt hơn. Cân bằng tốt nhất là **conf 0.3, iou 0.7** (HOTA 29.97).

Một hạn chế nữa của báo cáo này: đánh giá chất lượng track "bằng mắt" cho video_1 dựa trên số liệu và preview, chưa đối chiếu từng khung hình gây lỗi.

### video_2 (phố đêm, camera tĩnh trên cao, rất đông)

Chọn ByteTrack vì camera **đứng yên** nên mô hình chuyển động (Kalman filter + khớp IoU) đáng tin, còn **ban đêm** làm vector ngoại hình kém chất lượng — đúng chỗ Re-ID mất lợi thế. Trong lượt chạy 150 frame, ByteTrack cho **16 ID và 0 track ngắn**, còn BoT-SORT cho 23 ID với 3 mảnh vụn → ByteTrack ít bị vỡ danh tính hơn ở cảnh tối. Đám đông dày cũng là nơi ByteTrack phát huy bước gán thứ hai với các hộp conf thấp để bám người mờ.

### video_3 (camera di động, ảnh nhỏ, ít khung/giây)

Chọn BoT-SORT vì camera **tự di chuyển** làm vị trí của cùng một người dịch chuyển giữa hai frame, phá giả định "camera đứng yên" của tracker chuyển động — vận tốc suy ra bị sai. Phần **ngoại hình (Re-ID)** giúp nối lại danh tính dù khung nền đã đổi. Ảnh nhỏ và ít khung/giây càng làm dự đoán vận tốc kém chính xác, nên nghiêng hẳn về Re-ID. Đây là ví dụ điển hình cho quy tắc: **camera càng di chuyển thì càng cần Re-ID**.

## 4. Nếu có thêm thời gian

Một hoặc hai câu: bạn sẽ thử tiếp điều gì (Re-ID khác, quét `conf` mịn hơn, xem frame gây lỗi…).

Quét `conf` mịn hơn (0.20 / 0.25) quanh giá trị tốt nhất của `video_1` để tìm điểm cân bằng HOTA/MOTA, và tách vài frame bị đổi ID để xác định lỗi đến từ **detector bỏ sót** hay từ **tracker gán nhầm**. Ngoài bài nộp chính (được phép như phần mở rộng), có thể thử một mô hình Re-ID mạnh hơn cho cảnh đêm/rung.
