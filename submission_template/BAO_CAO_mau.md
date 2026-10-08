# Báo cáo lab: chọn tracker cho 5 video

**Nhóm:** ………………………… **Thành viên:** …………………………

Detector cố định: `yolo26n.pt`, ảnh 640 px, Re-ID `osnet_x0_25_msmt17`. Không đổi các mục này trong bài nộp chính.

## 1. Cấu hình đã chọn

Mỗi video: tracker bạn nộp, `conf`, `iou`, điều bạn **nhìn thấy** trên video, và một cấu hình đã thử rồi loại.

| Video | Tracker | conf | iou | Quan sát khi xem video | Đã thử nhưng loại |
|---|---|---|---|---|---|
| video_1 (quảng trường, tĩnh, ban ngày) | botsort | 0.3 | 0.7 | Cảnh tĩnh, mật độ vừa. BoT-SORT (có Re-ID) giữ ID ổn khi hai người đi cắt nhau, ít đổi màu ID. NMS `iou 0.7` giữ được các hộp người đứng gần nhau. Recall còn thấp vì detector `conf 0.3` bỏ sót người nhỏ/xa. | bytetrack (HOTA 26.9), deepocsort (HOTA 27.4) thấp hơn; `conf 0.5` làm HOTA tụt còn 27.2 |
| video_2 (phố đêm, tĩnh, rất đông) | | | | | |
| video_3 (camera di động, ảnh nhỏ) | | | | | |
| video_4 (trong nhà, camera di chuyển) | | | | | |
| video_5 (trên xe bus, giao lộ đông) | | | | | |

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

## 4. Nếu có thêm thời gian

Một hoặc hai câu: bạn sẽ thử tiếp điều gì (Re-ID khác, quét `conf` mịn hơn, xem frame gây lỗi…).
