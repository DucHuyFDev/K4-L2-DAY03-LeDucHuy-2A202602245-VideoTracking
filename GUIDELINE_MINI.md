# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu ghi lại quy tắc tôi dùng khi làm bài cá nhân. Mục tiêu là để
> việc tự kiểm và rework sau này có thể tái hiện nhất quán.

Hình thức: cá nhân
Họ tên: Lê Đức Huy
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán                            | Không gán                                                   |
| ------------------------------- | ------------------------------------------------------------- |
| xe con, SUV, taxi, xe bán tải | người đi bộ                                               |
| van, minivan                    | xe đạp                                                      |
| xe buýt, minibus               | **xe máy / mô tô**                                   |
| xe tải, xe đầu kéo          | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung: không có; bài được thực hiện cá nhân.

## 2. Luật ID — phần quan trọng nhất

| Tình huống                          | Quy tắc cá nhân                                                                                                    | Vì sao                                                                                  |
| ------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che dưới 25 frame (xấp xỉ 2 giây ở 12.5 fps)                                           | Tránh tách một xe thành nhiều identity khi vẫn có thể nhận ra xe sau vùng che. |
| Xe bị che lâu hơn ngưỡng trên   | mở track mới khi xe xuất hiện lại                                                                                | Sau khoảng che dài, không đủ bằng chứng chắc chắn để nối identity cũ.       |
| Xe rời khung hình rồi quay lại    | mở track mới                                                                                                        | Lần xuất hiện sau được xem là một đoạn track mới, theo quy ước của lab.    |
| Hai xe cắt nhau / chồng lên nhau   | giữ ID riêng; theo dõi đặc điểm, vị trí trước/sau vùng cắt và không đổi ID chỉ vì bbox chồng nhau | Ưu tiên continuity của từng xe, tránh ID switch khi hai xe đi qua nhau.            |

## 3. Luật bbox

| Tình huống                                   | Quy tắc cá nhân                                                                                                                                                      |
| ---------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Xe bị cắt bởi rìa ảnh                     | bbox chạm đúng rìa, không đoán phần ngoài ảnh                                                                                                                 |
| Xe bị xe khác che một phần                 | bbox ôm phần**nhìn thấy được**                                                                                                                             |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên có thể xác định đó là xe bốn bánh; nếu chưa phân biệt được với vật thể khác thì chờ thêm frame rõ hơn. |
| Xe đang đỗ, không di chuyển               | vẫn gán và giữ cùng ID trong toàn bộ thời gian xe còn trong khung.                                                                                             |
| Keyframe đặt dày ở đâu                   | đặt dày khi xe rẽ, phanh, bị che, gần rìa ảnh hoặc bbox thay đổi nhanh; đoạn đi thẳng đều có thể dùng khoảng cách thưa hơn.                     |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1

- Clip / frame / ID: `clip_01 / 55–67 / ID4`
- Tình huống: ID4 xuất hiện ở vùng gần rìa ảnh và pre-gold bị thiếu các row trong đoạn này.
- Quyết định: bổ sung bbox cho ID4 từ frame 55 đến 67, không tạo ID mới.
- Lý do: đây vẫn là cùng track liên tục; sau rework ID4 kéo dài đến frame 151 và số row tăng thêm 13.

### Ca 2

- Clip / frame / ID: `clip_01 / 149–151 / ID4`
- Tình huống: ID4 bị cắt mạnh bởi cạnh trái ảnh khi rời khung.
- Quyết định: giữ ID4, đặt bbox chạm cạnh trái và kết thúc track đúng khi xe rời khung; không đoán phần ngoài ảnh.
- Lý do: bbox phải chỉ bao phần nhìn thấy; diagnostics sau rework không còn ghost track 4 ở các frame này.

### Ca 3

- Clip / frame / ID: `clip_01 / 167 / ID8`
- Tình huống: ID8 ở sát rìa phải, bbox ban đầu bị loose và phần xe nhìn thấy thay đổi nhanh.
- Quyết định: chỉnh bbox theo phần xe thực sự nhìn thấy, không mở ID mới.
- Lý do: diagnostics pre-gold ghi loose box tại frame 167; sau rework lỗi này không còn.

## 5. Sửa gì sau khi chấm với gold và sau khi tự kiểm

- Khi xe sát rìa ảnh, bbox phải chạm đúng rìa ảnh và không được kéo dài vào phần ngoài khung; cần kiểm tra riêng frame cuối của track.
- Khi một đoạn bbox bị thiếu ở giữa một track liên tục, phải bổ sung vào ID đang có thay vì tạo track mới; chỉ mở ID mới khi xe thực sự rời khung hoặc bị che quá ngưỡng.
- Sau mỗi lần sửa phải chạy lại validator/evaluation và xem các frame biên, không chỉ kiểm tra số lượng track.
