# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Phạm Xuân Duy (MSSV: 2A202602093)`
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

Bổ sung của nhóm (nếu có): `Không gán xe kéo đẩy tay, không gán các khối hàng/quầy sạp ven đường có hình hộp tương tự xe tải.`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | `Bảo toàn identity liên tục của cùng một xe khi bị che khuất tạm thời (short occlusion) theo tiêu chuẩn MOT.` |
| Xe bị che lâu hơn ngưỡng trên | `Mở track mới với ID mới (bấm outside khi bị che, gán ID mới khi xuất hiện lại).` | `Thời gian che khuất quá lâu (>2s) không đủ cơ sở thị giác chuyển động liên tục để chắc chắn cùng một xe nếu không có ReID.` |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | `Quy ước benchmark MOT: mỗi lần vật thể đi vào trường nhìn (FOV) tính là một quỹ đạo riêng biệt.` |
| Hai xe cắt nhau / chồng lên nhau | `Giữ nguyên ID của từng xe dựa trên vector chuyển động và hướng đi, không hoán đổi ID.` | `Tránh lỗi ID switch nghiêm trọng; xe ở trước vẽ bbox trọn vẹn, xe ở sau vẽ bbox phần nhìn thấy.` |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `khi xe lộ diện tối thiểu 20% hoặc kích thước >= 15x15 px và nhận diện được chi tiết xe 4 bánh` |
| Xe đang đỗ, không di chuyển | `Vẫn track và giữ nguyên ID trong suốt thời gian xuất hiện trong khung hình; đặt keyframe thưa (đầu, giữa, cuối).` |
| Keyframe đặt dày ở đâu | `Đặt dày (cách 3–5 frame) tại khúc xe đổi hướng, phanh/tăng tốc, bắt đầu/kết thúc occlusion; đặt thưa (15–20 frame) khi đi thẳng đều.` |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 51–54 / ID 4`
- Tình huống: `Xe 4 bắt đầu xuất hiện ở mép phải màn hình, frame 51 mới chỉ lộ một phần rất nhỏ của mũi xe (rộng ~5px).`
- Quyết định: `Bắt đầu gán track ID 4 từ frame 54 khi cấu trúc xe bốn bánh đã hiện rõ hơn 50%, loại bỏ các frame 51–53 gán quá sớm.`
- Lý do: `Tránh tạo ra ghost prediction (false positive) do xe chưa đạt ngưỡng kích thước và độ tin cậy nhận diện của benchmark.`

### Ca 2
- Clip / frame / ID: `clip_01 / frame 84–89 / ID 5`
- Tình huống: `Xe 5 bị xe 4 đi phía trước che khuất một phần thân xe; nếu vẽ bao cả phần bị che thì IoU tụt xuống ~0.54 (loose box).`
- Quyết định: `Chỉ vẽ bbox ôm sát phần thân và đuôi xe nhìn thấy được (visible part), bổ sung keyframe ở frame 86 và 88.`
- Lý do: `Tuân thủ triệt để nguyên tắc không đoán phần bị che khuất; nâng IoU lên trên 0.70 so với gold reference.`

### Ca 3
- Clip / frame / ID: `clip_01 / frame 86–100 / ID 6`
- Tình huống: `Xe 6 ở vị trí rất xa trên tuyến đường, hình ảnh mờ và bị che một phần bởi các xe phía trước, dễ nhầm với vật thể tĩnh.`
- Quyết định: `Không gán ở frame 86–100; bắt đầu track ID 6 từ frame 101 khi xe tiến gần và chuyển động vào làn rõ ràng.`
- Lý do: `Khắc phục lỗi ghost track 15 frame bị báo trong eval_vs_gold.json; đảm bảo nhất quán với mốc bắt đầu của teaching reference.`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- `Bổ sung quy tắc định lượng rõ ràng cho điểm bắt đầu track: Chỉ kích hoạt track khi xe lộ diện >= 20% thân xe hoặc kích thước tối thiểu đạt 15x15 pixel và nhận diện rõ kết cấu xe bốn bánh; không gán khi xe mới chỉ thò vài pixel ở rìa ảnh.`
- `Bổ sung quy chuẩn đặt keyframe khi bị che khuất (occlusion): Bắt buộc thêm keyframe tại frame bắt đầu bị che và frame thoát che, chỉ vẽ bbox phần nhìn thấy được (visible portion) thay vì ước đoán toàn bộ xe để triệt tiêu lỗi bbox trôi (loose box với IoU < 0.60).`
