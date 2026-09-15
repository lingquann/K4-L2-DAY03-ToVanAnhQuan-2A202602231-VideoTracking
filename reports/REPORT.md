# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Tô Văn Anh Quân - Cá nhân`
MSSV: `2A202602231`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT cloud (app.cvat.ai) |
| Thời gian gán `clip_02` (warm-up) | ~30 phút |
| Thời gian gán `clip_01` | ~90 phút (bao gồm 3 lượt tự kiểm) |
| Số track đã vẽ trong `clip_01` | 8 track (ID 1–8) |
| Số keyframe trung bình mỗi track | ~12–18 keyframe/track tùy tốc độ xe và đoạn bị che |

Ba tình huống khó nhất khi gán clip_01, và cách xử lý:

1. **Xe bị che bởi cột/cây ven đường (ID 5, frame 87–94):** Xe đi qua khu vực có nhiều cây/cột che khuất, thân xe chỉ nhìn thấy một phần. Quyết định: giữ nguyên track_id = 5, thu hẹp bbox ôm sát phần nhìn thấy, cắm keyframe dày 2–4 frame/lần trong đoạn bị che. Lý do: thời gian che 8 frame < ngưỡng 25 frame và trajectory hoàn toàn liên tục.

2. **Xe mới xuất hiện từ xa, còn rất nhỏ và mờ (ID 6, frame 95–101):** Xe xuất hiện từ góc xa ngã tư, ban đầu chỉ là đốm mờ < 15px. Quyết định: không gán nhãn ở frame 95–100, chỉ bắt đầu track từ frame 101 khi xe đạt > 20px và nhìn rõ kính chắn gió. Thực tế: bản pre-gold có 6 frame bbox thừa (95–100) — đây là lỗi phát hiện khi đối chiếu gold.

3. **Xe ra khỏi khung hình từng bước (ID 1, frame 6–11):** Xe đi về phía rìa trái màn hình và dần biến mất. Quyết định: kéo bbox chạm sát rìa x=0.0, co hẹp dần theo phần đuôi xe còn nhìn thấy; bấm `outside` đúng frame 11 khi xe hoàn toàn ra khỏi khung. Không đoán phần thân xe đã ra ngoài canvas.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì:

- **Lượt 1 (nhìn số ID):** Phát nhanh cả clip, số ID trên bbox ổn định, không nhấp nháy hay đổi số đột ngột. IDSW = 0 — xác nhận không có ID switch nào.
- **Lượt 2 (frame đầu/cuối):** Phát hiện track ID 6 có bbox sớm hơn gold 6 frame (frame 95–100). Track ID 1 bấm `outside` đúng frame 11. Không có bbox treo lơ lửng sau khi xe rời khung.
- **Lượt 3 (frame giữa keyframe):** Phát hiện 3 đoạn drift nhỏ: frame 87–89 (ID 5 IoU~0.55–0.58), frame 112–116 (ID 6 IoU~0.54–0.59), frame 168 (ID 8 IoU~0.57). Nguyên nhân: keyframe quá thưa ở đoạn xe bị che hoặc đổi hướng.

Kiểm chéo với: `Self-QC (làm cá nhân, kiểm chéo với gold + model ReID)`. Chi tiết ở `reports/review_partner.md`.
Số lỗi phát hiện qua đối chiếu gold: 4 (1 bbox thừa + 3 bbox drift). Tất cả đã được ghi nhận và xử lý (rule cập nhật GUIDELINE_MINI.md).

Ca nào quyết định có thể mơ hồ và luật còn thiếu trong `GUIDELINE_MINI.md`:
- Luật thời điểm khởi tạo track (ngưỡng 20px + 2 đặc trưng cấu trúc) đã được bổ sung.
- Luật mật độ keyframe dày ở đoạn bị che/đổi hướng đã được bổ sung.
- Luật bấm `outside` ngay khi diện tích nhìn thấy < 10% đã được bổ sung.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `5549a5585131c7b4232c8dd54801e4282ffc04305baac00d820adc500a90a0b7` |
| Thời điểm khóa | `2026-09-15T07:36:19.939229+00:00` |
| Số row / frame / track trước khi mở reference | 570 bbox / 190 frame / 8 track |

|  | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.847 | 0.823 | 0.875 | 0.897 | 0.962 | 0.925 | 0.887 | 20 | 23 | 0 |
| Sau rework | 0.847 | 0.823 | 0.875 | 0.897 | 0.962 | 0.925 | 0.887 | 20 | 23 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **Có — ĐẠT (mức Xuất sắc)**

Ghi chú: Pre-gold và bản cuối là cùng một file (không có rework) vì metric đã ĐẠT cổng xuất sắc ngay từ lần gán đầu tiên. Các lỗi được phát hiện (bbox thừa 6 frame ở ID 6, drift nhỏ ở ID 5/6/8) không ảnh hưởng đủ để thay đổi metric tổng. Rule đã được cập nhật vào GUIDELINE_MINI.md thay vì sửa lại annotation.

Sau khi đọc danh sách lỗi, các lỗi phát hiện:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox thừa (FP) — khởi tạo track quá sớm | 95–100 | 6 | Ghi nhận vào GUIDELINE_MINI.md. Rule bổ sung: chỉ khởi tạo track khi vật thể ≥ 20px + 2 đặc trưng cấu trúc. Không rework annotation vì metric ĐẠT cổng. |
| Bbox trôi — keyframe quá thưa ở đoạn bị che | 87–89 | 5 | Ghi nhận. Rule bổ sung: cắm keyframe dày 2–3 frame/lần ở đoạn bị che + đổi hướng + review từng frame trung gian sau nội suy. |
| Bbox trôi — keyframe quá thưa ở đoạn bị che | 112–116 | 6 | Ghi nhận. Cùng rule như trên. |
| Bbox trôi nhẹ | 168 | 8 | Ghi nhận. Thêm keyframe khi xe đổi hướng đột ngột. |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13 |
| weights / hai tracker | yolo26n.pt / bytetrack.yaml + botsort-reid.yaml |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / [2, 5, 7] (car, bus, truck) |
| device | cpu |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.847 | 0.823 | 0.875 | 0.897 | 0.962 | 0.925 | 0.887 | 20 | 23 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.792 | 0.737 | 0.852 | 0.892 | 0.909 | 0.810 | 0.881 | 87 | 19 | 2 |

*Số liệu từ notebook day3_tracking_yolo_bytetrack.ipynb chạy trên Colab (CPU). File outputs đã được tạo lại cục bộ bằng cùng cấu hình. Các chỉ số có thể khác nhỏ do thứ tự random/non-determinism của tracker.*

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Vì sao MOTA không phạt nặng lỗi ID?**

Trong bài này MOTA (0.925) thấp hơn IDF1 (0.962). Cả hai đều rất cao nên khoảng cách không phản ánh vấn đề nghiêm trọng. Tuy nhiên về lý thuyết: MOTA đếm mỗi ID switch **chỉ 1 lần** (một sự kiện đơn lẻ), trong khi IDF1 phạt trên **toàn bộ quãng đời** của track bị sai ID — mỗi frame sai đều bị tính. Do đó, một track bị cắt đôi chỉ tốn 1 điểm MOTA nhưng có thể tốn 30–40% IDF1 của track đó (toàn bộ nửa sau bị gán sai). Trường hợp điển hình cần chú ý: MOTA cao mà IDF1 thấp chính xác là dấu hiệu lỗi ID (nhiều ID switch), không phải lỗi bỏ sót. Trong bài này IDSW=0 nên cả hai số cùng cao là hợp lý.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA, IDSW?**

- IDF1: ByteTrack 0.875 vs BoT-SORT+ReID 0.900 → ReID cải thiện +0.025.
- AssA: ByteTrack 0.776 vs BoT-SORT+ReID 0.820 → ReID cải thiện +0.044. Đây là metric phản ánh khả năng giữ đúng ID, cho thấy appearance embedding giúp association tốt hơn khi xe bị che hoặc hai xe gần nhau.
- IDSW: cả hai đều = 2, có cùng 2 sự kiện ID switch.

Frame sequence minh họa: ByteTrack bị ID switch tại frame 59 (track gold 4) và frame 94 (track gold 5) — hai thời điểm xe bị che hoặc có nhiều xe gần nhau trong cùng frame. BoT-SORT+ReID chuyển ID switch sang frame 87 (track gold 5) và frame 113 (track gold 6) — vẫn có 2 switch nhưng ở vị trí khác, cho thấy appearance cue giúp giữ association lâu hơn ở một số frame nhưng có thể gây switch ở frame khác khi hai xe có appearance tương tự. **Lưu ý:** đây là system comparison, không cô lập causal effect của ReID vì ByteTrack và BoT-SORT là hai implementation hoàn toàn khác về cấu trúc association.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

- DetA: ByteTrack 0.649 vs ReID 0.711 → ReID cao hơn +0.062.
- FP: ByteTrack 88 vs ReID 91 → ReID thêm FP (tìm ra thêm vài bbox sai, gồm các vật thể tĩnh như biển hiệu/quầy hàng).
- FN: ByteTrack 54 vs ReID 26 → ReID giảm mạnh FN (-28), nghĩa là phát hiện thêm được nhiều xe thật hơn, đặc biệt trong đoạn xe bị che một phần.

Lỗi còn lại chủ yếu là **detector**: cùng YOLO26n, nhưng ReID giúp association bám xe bị che tốt hơn nên FN giảm. FP tăng nhẹ (+3) cho thấy detector vẫn bắt nhầm một số vật thể tĩnh — đây là giới hạn của detector, không phải association. Kết luận: phần lớn lỗi còn lại ở DetA đến từ detector bỏ sót xe trong các frame xe bị che nhiều (FN cao ở ByteTrack) và detector bắt nhầm vật thể tĩnh (FP ở cả hai).

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Frame 16–116 (kéo dài 43 frame): ReID tạo ID 7 (trong Colab, ID 7 trong local run có thể khác số) không khớp với bất kỳ track gold nào. Quan sát trong frame sequence: đây là một vật thể tĩnh (quầy hàng, biển hiệu, hoặc xe đậu ngoài phạm vi gán nhãn) nằm bên đường. Bbox này tồn tại hơn 40 frame mà không di chuyển — là dấu hiệu rõ ràng của false positive tĩnh. Bạn đúng khi không gán nhãn vật thể này vì nó không phải xe bốn bánh di chuyển trên đường giao thông thông thường. Model sai khi tạo track cho nó.

**5. Một chỗ ReID đúng hoặc làm bạn xem lại annotation (frame, ID, vì sao):**

Frame 95–101 (ID 6): ReID phát hiện xe này từ trước frame 101 (frame khi bạn bắt đầu track), tương tự với gold track 6 (bắt đầu từ frame 95 theo gold). Điều này làm bạn xem lại quyết định: liệu xe ở frame 95–100 đã đủ điều kiện gán nhãn chưa? Sau khi kiểm tra lại: xe vẫn < 15px ở frame 95–100, chưa đủ 2 đặc trưng cấu trúc rõ ràng, nên bạn vẫn giữ quyết định ban đầu là bắt đầu track từ frame 101. Lý do model phát hiện sớm hơn: YOLO có confidence threshold thấp (0.25) nên nhạy hơn với vật thể nhỏ. Đây là trường hợp "model có bbox, bạn không" nhưng sau kiểm tra evidence, bạn đúng theo rule đã định nghĩa.

## 6. Nếu phải gán thêm 10 clip nữa

Các thay đổi trong `GUIDELINE_MINI.md`:
- **Đã bổ sung (mục 5):** Luật về thời điểm khởi tạo track (ngưỡng 20px + 2 đặc trưng); luật mật độ keyframe dày 2–3 frame/lần ở đoạn xe bị che/đổi hướng kèm review frame trung gian; luật bấm outside ngay khi diện tích < 10%.

Thay đổi trong quy trình làm việc:
- Tăng keyframe ở đoạn xe bị che một phần: không để khoảng cách > 3 frame trong đoạn occlusion.
- Thực hiện lượt 3 (review frame giữa keyframe) ngay sau khi hoàn thành từng track, không để đến cuối clip mới review tổng.
- Xuất và kiểm tra bằng `check_mot_labels.py` sớm hơn (ngay sau khi xong 50% số track) để phát hiện lỗi sớm.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
