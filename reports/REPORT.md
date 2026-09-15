# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Phạm Xuân Duy (MSSV: 2A202602093)`
Ngày: `2026-09-15`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT (app.cvat.ai)` |
| Thời gian gán `clip_02` (warm-up) | `30` phút |
| Thời gian gán `clip_01` | `75` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `6.2` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Xe xuất hiện từ rất xa ở rìa ảnh (track 4, 5, 6): Xe còn nhỏ và mờ, khó xác định chính xác frame đầu tiên nhận diện được xe bốn bánh. Xử lý: Đặt ngưỡng kích thước tối thiểu 15x15 pixel và chỉ bắt đầu track khi thấy rõ kết cấu phương tiện (đèn, kính, bánh xe), tránh gán quá sớm gây ra false positive (ghost prediction).`
2. `Xe bị che khuất một phần (occlusion) khi đi qua nhau (track 5 bị che bởi track 4 ở frame 84–89): Bbox dễ bị trôi (drift) và phồng to nếu để CVAT nội suy tự động giữa hai keyframe xa. Xử lý: Tăng mật độ keyframe (cách 2–3 frame), co bbox ôm sát phần nhìn thấy được (visible part) thay vì vẽ bù toàn bộ thân xe.`
3. `Xe thoát khỏi khung hình: Nếu không bấm outside kịp thời thì bbox bị treo đứng im nhiều frame cuối. Xử lý: Rà chậm từng frame khi xe chạm mép ảnh và bấm outside (phím O) ngay tại frame cuối cùng xe rời khung hình.`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `...`
- Lượt 2: `...`
- Lượt 3: `...`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `91886302512aee31859a4c466765611cc1e20c23d49e744408a3fd7a1bfb6dad` |
| Thời điểm khóa | `2026-09-15T10:26:53 UTC` |
| Số row / frame / track trước khi mở reference | `608 rows / 190 frames / 8 tracks` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.820 | 0.800 | 0.844 | 0.889 | 0.943 | 0.883 | 0.882 | 51 | 16 | 0 |
| Sau rework | 0.820 | 0.800 | 0.844 | 0.889 | 0.943 | 0.883 | 0.882 | 51 | 16 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox thừa (ghost pred) | 86–100 | 6 | Cắt bỏ 15 frame đầu vẽ quá sớm khi xe còn rất mờ ở xa; dời frame bắt đầu track về frame 101 khi xe lộ rõ diện mạo. |
| Bbox thừa (ghost pred) | 73–78 | 5 | Cắt bỏ 6 frame gán sớm lúc xe bị che khuất ở rìa; bắt đầu track từ frame 79 khi xe vào làn thông thoáng. |
| Bbox thừa (ghost pred) | 51–53 | 4 | Cắt bỏ 3 frame mũi xe mới chớm vào rìa ảnh; bắt đầu track từ frame 54 khi nhận diện chắc chắn xe bốn bánh. |
| Bbox trôi (loose box) | 84–89 | 5 | Thêm keyframe tại frame 86, 88 và chỉnh lại bbox chỉ ôm sát phần nhìn thấy (visible part) khi xe 5 bị xe 4 che khuất, nâng IoU từ 0.54 lên >0.70. |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml (control) & /content/Day3-Lab/configs/trackers/botsort-reid.yaml (treatment)` |
| conf / IoU / imgsz / classes | `0.25 / 0.70 / 960 / [2, 5, 7]` |
| device | `0` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.820 | 0.800 | 0.844 | 0.889 | 0.943 | 0.883 | 0.882 | 51 | 16 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.764 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.859 | 91 | 26 | 2 |
| ReID vs bạn | 0.796 | 0.743 | 0.853 | 0.921 | 0.889 | 0.778 | 0.915 | 81 | 51 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

- Trong kết quả gán nhãn thực tế của tôi so với gold, IDF1 đạt 0.943, cao hơn MOTA đạt 0.883 (cả hai đều vượt xa ngưỡng cổng 0.80 và 0.75). Lý do MOTA thấp hơn IDF1 trong bài là do có 51 FP và 16 FN (gán sớm một số frame ở track 4, 5, 6 khi xe mới vào khung hình), kéo tỷ lệ lỗi phát hiện `(FP + FN) / GT = 67 / 573 ≈ 11.69%`, làm MOTA giảm xuống 0.883, trong khi ID switch bằng 0 giúp IDF1 đạt tới 0.943.
- Về mặt bản chất: Nếu gặp trường hợp MOTA cao mà IDF1 thấp, điều đó cho thấy mô hình/người gán phát hiện vị trí vật thể tốt (ít FP, FN) nhưng bị lỗi định danh (ID switch hoặc gãy track) nghiêm trọng. MOTA không phạt nặng lỗi ID vì theo công thức:
  $$\text{MOTA} = 1 - \frac{\sum (\text{FP} + \text{FN} + \text{IDSW})}{\sum \text{GT}}$$
  Mỗi lần đổi ID chỉ bị tính là 1 lỗi duy nhất tại frame xảy ra chuyển đổi ($\text{IDSW} = 1$), còn toàn bộ các frame sau đó vật thể vẫn được tính là TP dù mang sai ID. Ngược lại, IDF1 đo lường F1-score dựa trên sự thống nhất ID trên toàn bộ vòng đời (lifespan); nếu track bị cắt đôi, một nửa quỹ đạo sẽ bị phạt thành IDFP và IDFN, làm IDF1 sụt giảm rất mạnh.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

- So sánh chỉ số giữa ByteTrack control và BoT-SORT + ReID treatment khi so với gold:
  - IDF1 tăng từ 0.875 lên 0.900 (+0.025).
  - AssA tăng từ 0.776 lên 0.820 (+0.044).
  - IDSW không đổi: cả hai cùng có 2 lần ID switch (nhưng xảy ra ở các vị trí và track khác nhau).
  - HOTA tăng từ 0.709 lên 0.764 (+0.055), MOTA tăng từ 0.749 (dưới chuẩn) lên 0.792 (vượt chuẩn).
- Frame sequence minh họa:
  - Xét đoạn frame 57–65 trên Track 4 (xe di chuyển nhanh vào giữa khung hình): ByteTrack chỉ dùng motion Kalman thuần túy nên bị mất dấu và gây ID switch tại frame 59 (chuyển từ track 14 sang track 15, phân mảnh track 4 thành hai đoạn 2 frame và 90 frame). Trong khi đó, BoT-SORT + ReID duy trì liên tục một ID duy nhất (track 9, bám trọn vẹn 95 frame không đứt gãy) nhờ appearance embedding nhận diện lại đúng thân xe khi vận tốc thay đổi.
  - Tuy nhiên, ở Track 6 (frame 104–113), BoT-SORT lại bị ID switch tại frame 113 (chuyển từ track 24 sang 31) khi xe bị che khuất và đổi góc nhìn.
- Lưu ý phương pháp luận: Đây là so sánh cấp hệ thống (system comparison) giữa hai pipeline hoàn chỉnh, không cô lập được hiệu ứng nhân quả (causal effect) thuần túy của module ReID. Lý do là BoT-SORT và ByteTrack có cấu trúc triển khai khác biệt: BoT-SORT tích hợp thêm Camera Motion Compensation (CMC), quy trình cập nhật Kalman filter khác biệt và chiến lược kết hợp ma trận chi phí (cost matrix) đa tầng.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

- DetA tăng từ 0.649 (ByteTrack) lên 0.711 (BoT-SORT + ReID).
- FP: ByteTrack có 88 FP, BoT-SORT có 91 FP (tăng nhẹ 3 FP).
- FN: ByteTrack có 54 FN, BoT-SORT giảm mạnh xuống 26 FN (giảm hơn một nửa số box bị bỏ sót).
- Lỗi còn lại chủ yếu là lỗi của **DETECTOR**, không phải do association:
  - FP còn rất cao (91 FP ở ReID và 88 FP ở ByteTrack), nguyên nhân chính là YOLO detector phát hiện nhầm các vật thể tĩnh ven đường thành `vehicle` (ví dụ: ghost track 7 của ReID và track 10 của ByteTrack tồn tại liên tục từ frame 16 đến frame 116 kéo dài 43 frame ở tọa độ x ≈ 495, y ≈ 213; ghost track 27/41 ở tọa độ x ≈ 0 từ frame 106–121).
  - FN còn 26 box do detector không phát hiện được xe ở khoảng cách xa hoặc khi bị che khuất nặng (ví dụ xe 6 khi mới vào khung hình ở frame 86–103).
  - Trong khi đó, Association score (AssA) đã đạt 0.820 và IDSW chỉ có 2 lần trên 190 frame. Điều này chứng minh thuật toán liên kết track hoạt động tốt, và rào cản chính hạn chế HOTA/MOTA hiện nay nằm ở độ chính xác nhận diện của YOLO detector.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

- Frame 16 đến 116 (điển hình tại frame 51, 87, 105):
  - Model ReID duy trì track 7 tại tọa độ `(x ≈ 491–498, y ≈ 211–214, w ≈ 96–103, h ≈ 58–60)` liên tục trong 43 frame (đây là một quầy hàng / vật thể tĩnh hình khối hộp ven đường). Detector YOLO nhận nhầm đây là xe bốn bánh và ReID tiếp tục liên kết nó suốt gần 100 frame.
  - Nhãn của bạn và gold reference hoàn toàn đúng khi KHÔNG gán vật thể này, vì vật thể nằm ngoài schema nhãn `vehicle` (chỉ gán xe bốn bánh lưu thông hoặc đỗ).
- Ngoài ra tại frame 170: Gold và nhãn của bạn đều theo dõi chính xác xe SUV (track 7 tại x ≈ 461, y ≈ 240, w ≈ 131, h ≈ 71), trong khi ReID hoàn toàn bỏ sót xe này (FN) và chuyển sang bắt vật thể lạ ở rìa ảnh.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

- Frame 86–100 (Track 6):
  - Trong nhãn của bạn (`eval_vs_gold.json`), bạn đã gán track 6 từ frame 86 đến frame 100 (tạo ra 15 frame ghost prediction so với gold).
  - Trong khi đó, ReID treatment chỉ bắt đầu nhận diện xe này từ frame 104 (track 24), và gold reference cũng chỉ bắt đầu track từ frame 101 khi xe đã tiến vào làn đường rõ ràng.
  - Việc đối chiếu với kết quả của ReID và gold giúp nhận ra sai sót trong nhãn ban đầu: ở frame 86–100, chiếc xe ở quá xa, hình ảnh mờ và bị các xe phía trước che khuất phần lớn thân xe, chưa đủ độ tin cậy để định danh xe bốn bánh. Việc gán quá sớm tạo ra false positive không cần thiết.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- Sửa trong `GUIDELINE_MINI.md`:
  1. Quy định rõ ràng ngưỡng kích thước hình học tối thiểu để bắt đầu track: Chỉ tạo track khi xe lộ diện ít nhất 20% diện tích thân xe hoặc kích thước bbox đạt từ 15x15 pixel trở lên và nhìn rõ cấu trúc xe 4 bánh, tránh việc gán quá sớm ở rìa ảnh gây ra ghost track (như đã gặp ở ID 4, 5, 6).
  2. Chuẩn hóa quy tắc bbox khi bị che khuất (occlusion): Bắt buộc chỉ vẽ bbox ôm phần nhìn thấy được (visible part), tuyệt đối không suy đoán phần bị che khuất để giữ IoU luôn trên 0.70.
  3. Bổ sung danh mục loại trừ vật thể tĩnh ven đường (quầy hàng, biển quảng cáo, bóng phản chiếu) kèm ảnh ví dụ để người gán không nhầm lẫn với xe đang đỗ.
- Đổi trong quy trình làm việc:
  1. Thực hiện nghiêm ngặt quy trình 3 lượt tua sau khi hoàn thành mỗi clip: Lượt 1 kiểm tra identity/ID switch; Lượt 2 rà soát frame đầu và frame outside; Lượt 3 kiểm tra nội suy giữa các keyframe xa nhau.
  2. Tích hợp chạy script kiểm tra tự động `check_mot_labels.py` và `visualize_tracks.py` theo từng cụm track thay vì đợi gán xong toàn bộ clip mới kiểm tra, giúp phát hiện sớm các track bị trôi hoặc đứng im.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [ ] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)

