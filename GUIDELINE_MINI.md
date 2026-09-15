# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Tô Văn Anh Quân - MSSV: 2A202602231 (Thực hiện cá nhân)`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của sinh viên: Không gán container/rơ-moóc đứng rời không gắn với đầu kéo; không gán xe công trình chuyên dụng không di chuyển trên đường giao thông thông thường (máy xúc, xe lu); không gán hình ảnh xe cộ in trên áp phích/thân xe buýt.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của sinh viên | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Trong giao thông đô thị, các vật thể tĩnh che khuất như cây cối, cột đèn, biển báo chỉ che xe trong khoảng 0.5–1.5 giây (< 20 frame). Giữ nguyên ID bảo toàn quỹ đạo tracking liên tục, tránh sinh ra lỗi ID switch (IDSW) không cần thiết. |
| Xe bị che lâu hơn ngưỡng trên | Mở track mới (cấp `track_id` mới khi xe xuất hiện trở lại) | Khi xe bị che khuất quá 2 giây (> 25 frame), bất định về hành vi (xe có thể đã rẽ, dừng lại, hoặc bị tráo đổi với xe khác có cùng màu sắc/chủng loại) là rất lớn; gán chung ID sẽ có rủi ro cao tạo ra False Positive hoặc hoán đổi sai danh tính vật thể. |
| Xe rời khung hình rồi quay lại | Mặc định: **track mới** (kết thúc track cũ khi chạm rìa/mất dấu, mở ID mới khi quay lại) | Khi xe đã hoàn toàn đi ra ngoài Field of View (FoV) của camera, track cũ coi như đã kết thúc (`outside` trong CVAT). Một tracker camera đơn thông thường không có đủ đặc trưng 3D/GPS để khẳng định chắc chắn 100% cùng một cá thể sau một khoảng thời gian ngoài khung. |
| Hai xe cắt nhau / chồng lên nhau | Mỗi xe giữ nguyên một `track_id` riêng biệt. Xe ở lớp trước (foreground) có bbox ôm toàn bộ thân nhìn thấy; xe ở lớp sau (background) có bbox ôm sát phần thân xe **còn nhìn thấy được** (visible part), không vẽ bbox bao trùm phần bị che. Không đổi ID giữa hai xe. | Giữ đúng danh tính xuyên suốt tương tác giao cắt (occlusion handling), tránh hiện tượng đổi chéo ID khi hai bbox chồng lấn lớn. Tuân thủ chuẩn MOT benchmark về việc bbox chỉ bám phần nhìn thấy được. |

## 3. Luật bbox

| Tình huống | Luật của sinh viên |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa ảnh (x=0, y=0, x=width hoặc y=height), tuyệt đối không suy đoán hoặc vẽ phần thân xe nằm ngoài ảnh. |
| Xe bị xe khác che một phần | Bbox ôm sát phần **nhìn thấy được** (visible portion), không cố gắng ước lượng phần thân xe bị khuất phía sau. |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định được chắc chắn là xe bốn bánh; ngưỡng sinh viên chọn: **chiều rộng hoặc chiều cao tối thiểu >= 20 pixel** và nhận diện được tối thiểu 2 đặc trưng cấu trúc (như cụm đèn, kính chắn gió hoặc bánh xe). |
| Xe đang đỗ, không di chuyển | Vẫn gán nhãn nhãn `vehicle`, duy trì cố định một `track_id` duy nhất và giữ tọa độ bbox ổn định không co giật/rung lắc trong suốt thời gian xe nằm trong khung hình. |
| Keyframe đặt dày ở đâu | Đặt keyframe dày (cách nhau 2–4 frame) tại các frame xe bắt đầu/kết thúc bị che khuất (bởi cây/cột/xe khác), đoạn xe đổi hướng/rẽ cua, xe tăng/giảm tốc độ đột ngột, hoặc xe ở gần camera có phối cảnh biến dạng nhanh. Các đoạn xe chạy thẳng đều chỉ cần đặt keyframe cách nhau 10–15 frame. |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01` / Frame: `87–94` / ID: `5`
- Tình huống: Chiếc xe ID 5 di chuyển trên làn đường và bị một thân cây / cột ven đường che khuất một phần thân xe liên tục từ frame 87 đến frame 94 (khoảng 8 frame, tương đương ~0.64 giây).
- Quyết định: Giữ nguyên `track_id = 5`, không ngắt track thành ID mới. Thu hẹp bbox để chỉ ôm sát phần thân xe nhìn thấy được ngoài thân cây, đặt các keyframe tại frame 87, 89, 91, 94.
- Lý do: Thời gian che khuất là 8 frame, nhỏ hơn nhiều so với ngưỡng 25 frame (2 giây). Vị trí, hướng di chuyển và vận tốc của xe trước và sau khi đi qua cột cây là hoàn toàn khớp nhau, đảm bảo tính liên tục của trajectory.

### Ca 2
- Clip / frame / ID: `clip_01` / Frame: `95–101` / ID: `6`
- Tình huống: Chiếc xe ID 6 mới xuất hiện từ phía xa ở góc ngã tư; ở các frame 95–100, xe chỉ là một vệt mờ có kích thước nhỏ dưới 15 pixel, chưa nhìn rõ cụm đèn hay đường nét thân xe bốn bánh.
- Quyết định: Không vội gán nhãn từ frame 95 (tránh bbox thừa / False Positive). Bắt đầu khởi tạo track và gán `track_id = 6` từ frame 101 trở đi, khi xe đã đạt kích thước trên 20 pixel và phân biệt rõ được kính chắn gió cùng hình khối xe con.
- Lý do: Tuân thủ đúng ngưỡng xe mới xuất hiện: chỉ track khi vật thể đạt kích thước >= 20px và đủ căn cứ xác định là xe 4 bánh, giúp tránh nhiễu với xe máy/người đi bộ ở xa và khớp với ground truth chuẩn.

### Ca 3
- Clip / frame / ID: `clip_01` / Frame: `6–10` / ID: `1`
- Tình huống: Chiếc xe ID 1 đang chạy về phía rìa trái màn hình và dần đi ra khỏi khung hình; từ frame 6 đến frame 10 thân xe bị cạnh trái cắt từ 30% đến 80%, chỉ còn nhìn thấy phần đuôi xe nhỏ dần sát mép x=0.0 trước khi biến mất ở frame 11.
- Quyết định: Bbox được kéo chạm sát đúng mép trái màn hình (`x = 0.0`), co hẹp dần theo phần đuôi xe còn nhìn thấy trong ảnh. Ngay tại frame 11 khi xe hoàn toàn ra khỏi khung hình, lập tức kích hoạt thuộc tính `outside` để chấm dứt track ID 1.
- Lý do: Không vẽ bbox tràn ra ngoài tọa độ ảnh (ngoài canvas) và không đoán phần đầu xe đã ra ngoài; bấm `outside` đúng frame xe biến mất để ngăn chặn lỗi bbox treo (hanging/orphan bbox).

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Luật về thời điểm bắt đầu khởi tạo track cho xe ở xa (khắc phục lỗi Bbox thừa - FP):**
  - *Vấn đề trước đó:* Quy định ban đầu chỉ ghi "bắt đầu khi nhìn thấy xe bốn bánh", dẫn đến việc đặt bbox quá sớm ở các frame xe còn là đốm mờ (như ID 6 từ frame 95-100), gây ra 6 frame bbox thừa so với gold.
  - *Viết lại bổ sung:* "Chỉ khởi tạo track khi vật thể đạt kích thước tối thiểu >= 20 pixel theo cả hai chiều và quan sát rõ ít nhất 2 đặc trưng cấu trúc xe (kính chắn gió, bánh xe, đèn). Nếu xe chưa đạt ngưỡng trên thì bỏ qua, không gán nhãn cho đến khi đủ điều kiện."
- **Luật về mật độ keyframe khi xe bị che hoặc đổi hướng (khắc phục lỗi Bbox trôi - Drift):**
  - *Vấn đề trước đó:* Đặt keyframe quá thưa (cách 10–15 frame) khiến thuật toán nội suy tuyến tính (interpolation) của CVAT tạo ra bbox bị lệch, không bám sát vật thể trong các frame trung gian (cụ thể frame 87–89 ở track 5 có IoU giảm xuống 0.54–0.58; frame 112–116 ở track 6 có IoU chỉ đạt 0.54–0.59).
  - *Viết lại bổ sung:* "Bắt buộc cắm keyframe dày với mật độ 2–3 frame/lần xuyên suốt khoảng thời gian xe bị che khuất một phần, xe đổi hướng di chuyển hoặc thay đổi vận tốc. Sau khi nội suy tự động, người gán nhãn phải tua lại từng frame (frame-by-frame review) tại các đoạn này để tinh chỉnh bbox ôm sát phần nhìn thấy được."
- **Luật dứt điểm track khi xe rời khung hình (khắc phục lỗi Bbox treo):**
  - *Vấn đề trước đó:* Chưa quy định rõ frame dứt điểm khi xe rời khung hình, dẫn đến việc bbox vẫn tồn tại thêm 1–2 frame sau khi xe đã hoàn toàn đi khỏi tầm nhìn.
  - *Viết lại bổ sung:* "Khi diện tích nhìn thấy của xe ở rìa ảnh nhỏ hơn 10% hoặc bề rộng thân xe còn lại dưới 10 pixel, đánh dấu `outside` kết thúc track ngay tại frame đó; không duy trì bbox trống."
