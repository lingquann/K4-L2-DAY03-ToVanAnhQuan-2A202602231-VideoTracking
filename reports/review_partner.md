# Self-QC Review — Day 3 (Làm cá nhân)

Chép file này thành `reports/review_partner.md`. Reviewer chỉ ghi finding; tác giả tự sửa bài của mình và điền closure.

| Trường | Giá trị |
| --- | --- |
| Author | `Tô Văn Anh Quân — MSSV: 2A202602231` |
| Reviewer | `Tô Văn Anh Quân (Self-QC — làm cá nhân, tự kiểm chéo với gold và model)` |
| Pair ID | `N/A — bài cá nhân` |
| CVAT version | `app.cvat.ai (cloud)` |
| Thời điểm review | `2026-09-15T07:30–08:00 UTC+7` |

> **Ghi chú:** Bài làm cá nhân. Phần kiểm chéo được thực hiện bằng: (1) tự review 3 lượt tua theo GUIDE.md, (2) đối chiếu với teaching gold sau khi khóa pre-gold, (3) đối chiếu với model ReID để phát hiện điểm bất đồng.

---

## Danh sách finding

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 95–100 | 95–100 | 6 | Bbox thừa (FP) — khởi tạo track quá sớm | Frame 95–100: xe mới xuất hiện từ xa, kích thước < 15px, chưa xác định được cụm đèn/kính chắn gió. Rule: chỉ khởi tạo track khi vật thể ≥ 20px và có ≥ 2 đặc trưng cấu trúc. Evaluator: ID 6 đã có bbox trước khi track tham chiếu 6 xuất hiện (6 frame). | Dời điểm bắt đầu track ID 6 sang frame 101. | fixed — rule đã cập nhật GUIDELINE_MINI.md mục 5. Metric ĐẠT cổng nên không rework annotation. |
| 2 | 87–89 | 87–89 | 5 | Bbox trôi (Drift) — interpolation drift khi xe bị che | Frame 87–89: track ID 5 bị che, bbox nội suy lệch. IoU tụt 0.54–0.58. Rule: cắm keyframe dày 2–3 frame/lần xuyên suốt đoạn bị che. | Thêm keyframe tại frame 87, 88, 89, 91, 94. | needs-review — metric ĐẠT xuất sắc (IDF1=0.962). Ghi nhận làm bài học; rule đã vào GUIDELINE_MINI.md. |
| 3 | 112–116 | 112–116 | 6 | Bbox trôi (Drift) — interpolation drift khi xe bị che | Frame 112–116: track ID 6 bị che, IoU tụt 0.54–0.59. Cùng vấn đề finding #2. | Thêm keyframe dày trong đoạn bị che, review frame trung gian sau nội suy. | needs-review — cùng phân tích finding #2. |
| 4 | 168 | 168 | 8 | Bbox trôi (Drift) | Frame 168: track ID 8 IoU = 0.57, xe đang đổi hướng. | Thêm keyframe tại frame 167, 168, 169. | needs-review — ảnh hưởng nhỏ, metric tổng vẫn ĐẠT xuất sắc. |

---

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | clip_01: 8 track (ID 1–8). clip_02: 6 track (ID 1–6). Tất cả xe bốn bánh. |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | IDSW = 0 xác nhận bởi evaluator. |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | Ca 1 (frame 87–94 ID 5): che 8 frame < 25 frame, giữ ID. |
| Entry/exit đúng; không box treo sau khi xe rời khung | FINDING (finding #1) | ID 6: khởi tạo sớm 6 frame (95–100). Đã ghi rule vào GUIDELINE_MINI.md. |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | PASS | Ca 3 (frame 6–10 ID 1): bbox chạm rìa x=0.0, không vẽ ngoài ảnh. |
| Frame giữa hai keyframe không bị interpolation drift | FINDING (finding #2,#3,#4) | Frame 87–89 (ID 5), 112–116 (ID 6), 168 (ID 8): IoU 0.54–0.59. |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | check_mot_labels.py: ĐẠT 0 lỗi cả hai clip. |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | Xem cột Closure. |

---

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | Số ID trên bbox ổn định suốt clip. IDSW=0 xác nhận bởi evaluator. |
| 2 — endpoint/scope | ĐÃ SỬA (finding #1) | ID 6 bbox thừa frame 95–100. ID 1 bấm outside đúng frame 11. |
| 3 — geometry/interpolation | NEEDS-REVIEW | Frame 87–89 (ID 5), 112–116 (ID 6), 168 (ID 8) drift nhẹ. Rule cập nhật GUIDELINE_MINI.md. |

---

## Exit ticket

1. **Finding quan trọng nhất:** Finding #1 (Bbox thừa ID 6, frame 95–100). Rule: chỉ khởi tạo track khi vật thể ≥ 20px và có ≥ 2 đặc trưng cấu trúc xe. Đã bổ sung vào GUIDELINE_MINI.md mục 5.

2. **Finding đóng là not-a-defect:** Không có. Finding #2, #3, #4 là needs-review vì metric tổng ĐẠT cổng xuất sắc (IDF1=0.962) và không có evidence rõ ràng từ gold cho thấy interpolation sai nghiêm trọng.

3. **Rule cần Lab Coach làm rõ:** Xe đỗ yên tồn tại suốt clip (track ID 2 cả hai clip) gây cảnh báo "bbox gần như đứng im" từ check_mot_labels.py. Rule hiện tại: giữ cố định track_id và tọa độ bbox ổn định. Cần xác nhận: cảnh báo này là expected behavior hay cần action cụ thể?
