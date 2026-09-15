# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Lê Hùng Cường — MSSV 2A202602218 (làm cá nhân)`
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

Bổ sung của nhóm (nếu có): `Làm cá nhân — không bổ sung; áp dụng nguyên văn mặc định của lab (GUIDE.md).`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Occlusion ngắn, xe chưa khuất đủ lâu → vẫn là cùng một xe |
| Xe bị che lâu hơn ngưỡng trên | mở **track mới** (ID mới) khi xe hiện lại | Mất dấu quá 2 giây, không còn đủ bằng chứng motion để khẳng định cùng xe |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Ra khỏi khung = kết thúc track; quay lại là lần xuất hiện độc lập |
| Hai xe cắt nhau / chồng lên nhau | **giữ nguyên ID** của từng xe theo quỹ đạo liên tục, không đổi ID | Motion/IoU là tín hiệu mạnh nhất; chồng lấn không được phép gây swap ID |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `bbox đủ nét để phân biệt xe bốn bánh với xe máy — mốc chọn: chiều cao bbox ≥ ~10 px` |
| Xe đang đỗ, không di chuyển | `vẫn là vehicle — giữ track suốt thời gian ở trong khung, kể cả khi không di chuyển` |
| Keyframe đặt dày ở đâu | `ở chỗ xe đổi hướng, bị che, vào/ra khung và đoạn hai xe cắt nhau (để interpolation không trôi)` |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1

- Clip / frame / ID: `clip_01` / frame `148–151` / ID `4`
- Tình huống: `Xe 4 nhỏ dần rồi khuất ở mép khung hình — khó xác định chính xác frame xe thực sự rời khung.`
- Quyết định: `Bấm outside ở frame 151, tức muộn hơn reference (gold kết thúc track 4 ở frame 148) → để lại ghost bbox ở frame 149/151.`
- Lý do: `Xe còn lộ một mẩu nhỏ ở rìa nên chưa chắc đã hoàn toàn rời khung; đây là chỗ ranh giới “còn trong khung” và “đã ra khỏi khung” rất mơ hồ.`

### Ca 2

- Clip / frame / ID: `clip_01` / frame `111–114` / ID `7`
- Tình huống: `Xe 7 bị che một phần, phần nhìn thấy không rõ ranh giới nên bbox dễ ôm rộng/nhỏ hơn thực tế.`
- Quyết định: `Bbox ôm phần nhìn thấy được, không đoán phần bị che.`
- Lý do: `Đúng theo luật “bbox ôm phần nhìn thấy”, nhưng ranh giới vùng che khó xác định → lệch so với gold (IoU 0.516–0.599).`

### Ca 3

- Clip / frame / ID: `clip_01` / frame `~137–167` / ID `7` và `8`
- Tình huống: `Hai xe 7 và 8 tiến sát nhau ở vùng rìa cuối clip, có đoạn gần như chồng vùng — dễ nhầm ID nếu chỉ dựa vào vị trí.`
- Quyết định: `Giữ nguyên ID của từng xe, bám theo quỹ đạo chuyển động liên tục, không đổi ID.`
- Lý do: `Ưu tiên tín hiệu motion/IoU để tránh swap ID; kết quả đối chiếu gold cho IDSW = 0.`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- `Luật outside khi xe bị cắt rìa (frame 148–151, track 4): khi phần xe còn nhìn thấy < ~10% diện tích bbox thì kết thúc track bằng outside.`
- `Luật bbox khi xe bị che một phần (frame 111–114, track 7): ôm đúng mép ngoài của phần nhìn thấy, không cộng thêm ước đoán phần bị che.`
- `Luật hai xe cắt/gần nhau ở vùng rìa (frame ~137–167, track 7/8): ưu tiên quỹ đạo liên tục, không đổi ID chỉ vì box tạm thời chồng lấn.`
