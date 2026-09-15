# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: Lê Đức Huy
Ngày: 15/9/2026

---

## 1. Quá trình gán nhãn

| Mục                                 | Giá trị          |
| ------------------------------------ | ------------------ |
| Công cụ                            | CVAT / khác: CVAT |
| Thời gian gán`clip_02` (warm-up) | 7 phút           |
| Thời gian gán`clip_01`           | 15 phút           |
| Số track đã vẽ trong`clip_01`  | 8                  |
| Số keyframe trung bình mỗi track  | 8                  |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Vật thể bị chồng lấn: theo dõi đặc điểm, vị trí và hướng di chuyển của từng xe trước/sau vùng chồng lấn, giữ nguyên ID riêng và chỉ gán phần bbox thực sự nhìn thấy.
2. Vật thể xuất hiện lần đầu còn mờ hoặc khó nhìn: tua chậm đến frame đầu tiên có thể xác định đó là xe bốn bánh, sau đó đặt bbox sát phần nhìn thấy được và không đoán phần bị khuất.
3. CVAT nội suy sai giữa hai keyframe: tôi kiểm tra frame ở giữa đoạn nội suy, đặt thêm keyframe tại các điểm xe đổi hướng, bị che hoặc thay đổi hình dạng bbox, rồi kiểm tra lại trước khi lưu.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: ID không bị nhảy và có bám sát vào vật thể, không có 2 vật chung 1 ID
- Lượt 2: Khi vật thể rời khỏi khung hình, ngay lập tức tắt frame
- Lượt 3: Các frame giữa có 1 số frame do nội suy sai nên dễ rời khỏi vật thể

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence                                               | Giá trị                                                            |
| ------------------------------------------------------ | -------------------------------------------------------------------- |
| SHA-256 từ`evidence/pre-gold/clip_01/manifest.json` | `81a8a52f5e750b125c55fff58aeb4fc5da9093edd0f553c766051e4c4bacbcf5` |
| Thời điểm khóa                                     | `2026-09-15T04:08:19.744658+00:00`                                 |
| Số row / frame / track trước khi mở reference      | `509 / 190 / 8`                                                    |

|               |   HOTA |   DetA |   AssA |   LocA |   IDF1 |   MOTA |   MOTP | FP | FN | IDSW |
| ------------- | -----: | -----: | -----: | -----: | -----: | -----: | -----: | -: | -: | ---: |
| Bản pre-gold | 0.7633 | 0.7432 | 0.7847 | 0.8865 | 0.9168 | 0.8429 | 0.8768 | 13 | 77 |    0 |
| Sau rework    | 0.7925 | 0.7735 | 0.8126 | 0.8916 | 0.9333 | 0.8726 | 0.8833 | 11 | 62 |    0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi                               | Frame  | ID | Đã sửa thế nào                                                                |
| ---------------------------------------- | ------ | -- | ---------------------------------------------------------------------------------- |
| Bổ sung bbox cho track đang bị thiếu | 55–67 | 4  | Thêm 13 row cho ID4 ở các frame này.                                           |
| Bbox lỏng                               | 78     | 4  | Chỉnh lại bbox; diagnostics sau rework không còn lỗi loose box ở frame này. |
| Bbox lỏng                               | 167    | 8  | Chỉnh lại bbox; diagnostics sau rework không còn lỗi loose box ở frame này. |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục                               | Giá trị                                                                                          |
| ---------------------------------- | -------------------------------------------------------------------------------------------------- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13`                                                      |
| weights / hai tracker              | `yolo26n.pt / ByteTrack (bytetrack.yaml) / BoT-SORT + ReID (configs/trackers/botsort-reid.yaml)` |
| conf / IoU / imgsz / classes       | `0.25 / 0.70 / 960 / COCO classes 2, 5, 7`                                                       |
| device                             | `0`                                                                                              |

| So sánh                  |   HOTA |   DetA |   AssA |   LocA |   IDF1 |   MOTA |   MOTP |  FP | FN | IDSW |
| ------------------------- | -----: | -----: | -----: | -----: | -----: | -----: | -----: | --: | -: | ---: |
| bạn vs gold              | 0.7925 | 0.7735 | 0.8126 | 0.8916 | 0.9333 | 0.8726 | 0.8833 |  11 | 62 |    0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 |  88 | 54 |    2 |
| BoT-SORT + ReID vs gold   | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 |  91 | 26 |    2 |
| ReID vs bạn              | 0.7619 | 0.6981 | 0.8353 | 0.8873 | 0.8862 | 0.7471 | 0.8772 | 124 |  8 |    0 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của nhãn sau rework (0.8726) thấp hơn IDF1 (0.9333). Nhãn giữ identity khá tốt, nhưng MOTA còn trừ FP và FN; ở đây có 11 FP và 62 FN, còn IDSW bằng 0. Vì MOTA chỉ tính mỗi ID switch một lần trong tổng FP + FN + IDSW, một lỗi identity kéo dài có thể làm MOTA giảm ít hơn IDF1/AssA.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

BoT-SORT + ReID có AssA 0.8204 và IDF1 0.9001, cao hơn ByteTrack lần lượt 0.0443 và 0.0255; IDSW của cả hai đều là 2. ByteTrack có switch ở frame 59 (GT ID4) và 94 (GT ID5), còn ReID ở frame 87 (GT ID5) và 113 (GT ID6). Ở đoạn 87–113, ReID giữ association tốt hơn theo điểm tổng thể nhưng vẫn fragment ở các ID này. Đây không cô lập causal effect của ReID vì hai tracker implementation khác nhau.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

So với gold, ReID tăng DetA từ 0.6487 lên 0.7110 và giảm FN từ 54 xuống 26, nhưng FP tăng từ 88 lên 91. Vì vậy ReID bắt được nhiều box hơn nhưng cũng sinh thêm false positive; lỗi còn lại của model nghiêng về detector/threshold và ghost track, bên cạnh fragmentation association. Với annotation sau rework, DetA 0.7735, FP 11, FN 62 và IDSW 0: phần identity ổn, thiếu coverage và bbox/biên track còn đáng chú ý hơn association.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Frame 87: annotation chỉ có track 3 và 4; ReID thêm track 18 tại vùng xe khoảng x=690, trong khi diagnostics ghi track này xuất hiện trước khi GT track 5 xuất hiện. Đây là false positive/ghost của ReID, còn annotation không tạo track sớm ở frame này.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Frame 113, ReID có các track 18, 27, 29 và 31; diagnostics ghi track 31 là fragment của GT ID6 và track 27/29 là ghost hoặc xuất hiện trước reference tương ứng. Vì ReID-vs-bạn có 124 FP và MOTA chỉ 0.7471, evidence hiện có ủng hộ model sai hơn là buộc phải sửa annotation.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Mình sẽ điền rõ trong `GUIDELINE_MINI.md`: giữ ID khi che dưới 25 frame, ra khỏi khung rồi quay lại thì mở ID mới, và ghi ngưỡng bắt đầu khi xe đủ rõ là xe bốn bánh. Quy trình sẽ có ba lượt tua riêng cho ID, biên track và frame giữa keyframe; sau mỗi sprint sẽ Save bằng nút CVAT, reload kiểm tra số bbox, rồi mới export và khóa pre-gold.

## 7. Tệp đã nộp

- [X] `annotations/clip_01/gt.txt`
- [X] `annotations/clip_02/gt.txt`
- [X] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [X] `GUIDELINE_MINI.md` đã điền
- [X] `outputs/eval_vs_gold.json`
- [X] `outputs/model_bytetrack_clip_01.txt`
- [X] `outputs/model_reid_clip_01.txt`
- [X] `outputs/model_run_config.json`
- [X] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [X] `reports/REPORT.md` (file này)
