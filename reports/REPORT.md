# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: Lê Hùng Cường / MSSV 2A202602218
Ngày: 15/09/2026

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT (app.cvat.ai) |
| Thời gian gán `clip_02` (warm-up) | `50` phút |
| Thời gian gán `clip_01` | `50` phút |
| Số track đã vẽ trong `clip_01` | `8` (đếm từ `annotations/clip_01/gt.txt`) |
| Số keyframe trung bình mỗi track | `≈70,5` bbox/track (564 bbox ÷ 8 track, đếm từ `annotations/clip_01/gt.txt`) |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe 4 đi ra khỏi khung ở rìa ảnh (frame ~149–151).** Xe nhỏ dần rồi khuất ở mép, khó xác định chính xác frame xe thực sự rời khung. Tôi bấm `outside` ở frame 151, tức muộn hơn gold (gold kết thúc track 4 ở frame 148) → để lại ghost bbox ở frame 149/151. Đây là lỗi biên track điển hình: khi xe còn lộ một mẩu nhỏ ở rìa, rất dễ tưởng còn trong khung mà giữ box thêm vài frame.
2. **Xe 7 bị che một phần quanh frame 111–114.** Bbox của tôi ôm phần nhìn thấy được nhưng lệch so với gold (IoU chỉ 0.516–0.599), do xe bị chắn nên phần "nhìn thấy" tôi khoanh rộng/nhỏ hơn reference. Tôi xử lý theo luật "bbox ôm phần nhìn thấy", nhưng ranh giới vùng che vẫn là chỗ dễ sai vài pixel.
3. **Xe 7 và xe 8 cắt/gần nhau ở đoạn cuối (frame ~137–167).** Hai xe tiến sát nhau ở vùng rìa, dễ nhầm ID nếu chỉ dựa vào vị trí. Tôi bám từng xe theo quỹ đạo chuyển động liên tục để giữ đúng ID, kết quả không có ID switch nào so với gold (IDSW = 0), chỉ còn sai lệch nhỏ về bbox (track 8, frame 167, IoU 0.582).

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1 (ID): rà soát xe bị che rồi hiện lại — xác nhận không có ID nào bị tách hay đổi nhầm (về sau `eval_vs_gold` cho `IDSW = 0`, `IDF1 = 0.976`).
- Lượt 2 (frame đầu/cuối): kiểm `outside` đúng frame ra/vào khung — phát hiện track 4 hơi "treo" ở frame 149/151 (ghost), track 8 ở frame 167 khớp chưa sát.
- Lượt 3 (frame giữa): kiểm bbox không bị trôi khi xe đổi hướng/bị che — phát hiện các box lỏng (IoU ≈ 0.52–0.60) ở track 7 (frame 111–114).

Kiểm chéo với: làm cá nhân, chưa có bạn cùng nhóm. Chưa có `reports/review_partner.md` (chờ peer review thật trước khi điền — không tự sinh finding). Số lỗi bạn tìm được trong bản của bạn ấy: chưa áp dụng. Số lỗi bạn ấy tìm được trong bản của bạn: chưa áp dụng.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Làm cá nhân nên chưa có ca kiểm chéo đối lập. Luật còn thiếu cần bổ sung rõ trong `GUIDELINE_MINI.md`: ngưỡng chính xác "xe bị cắt rìa" để bấm `outside` (bao nhiêu pixel/bao nhiêu phần trăm bbox còn lộ thì vẫn giữ track), và cách ôm bbox khi xe bị che một phần (ôm đúng phần nhìn thấy theo mép nào khi bị chắn).

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `d0cd933df5bd60131fd1becab3f1c23df21f77604345c6fc7865ee039a5697e7` |
| Thời điểm khóa | `2026-09-15 10:37:23 UTC` (manifest: `locked_at_utc`) |
| Số row / frame / track trước khi mở reference | `564 / 190 / 8` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.793 | 0.783 | 0.805 | 0.850 | 0.976 | 0.953 | 0.831 | 9 | 18 | 0 |
| Sau rework | 0.793 | 0.783 | 0.805 | 0.850 | 0.976 | 0.953 | 0.831 | 9 | 18 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có** — IDF1 0.976 / MOTA 0.953 / MOTP 0.831 (từ `outputs/eval_vs_gold.json`)

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

Do bản pre-gold đã qua cổng, tôi **giữ nguyên** nhãn (không rework). Dưới đây là các điểm còn lệch giữa nhãn của tôi và gold — nếu cần sửa thêm thì đây là danh sách phải xử lý:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Ghost bbox sau khi xe rời khung | 149, 151 | 4 | Chưa sửa (đã qua cổng); nếu sửa thì bấm `outside` ở frame 148 để khớp gold |
| Bbox lỏng do che một phần | 111–114 | 7 | Chưa sửa; nếu sửa thì thêm keyframe ôm đúng phần nhìn thấy (IoU hiện 0.516–0.599) |
| Bbox lỏng ở rìa khi xe ra khung | 167 | 8 | Chưa sửa; nếu sửa thì khớp bbox theo gold (IoU hiện 0.582) |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Thành phần | Phiên bản / Giá trị | Ghi chú |
| :--- | :--- | :--- |
| **Python** | `3.13.15` | Môi trường lập trình Python |
| **Ultralytics** | `8.4.145` | Framework YOLO |
| **PyTorch** | `2.11.0+cu128` | Hỗ trợ tính toán GPU CUDA 12.8 |
| **lap** | `0.5.13` | Thuật toán Linear Assignment Problem (LAP) |
| **Device** | `0` | GPU CUDA số 0 |

So sánh              HOTA    DetA    AssA    LocA    IDF1    MOTA    MOTP      FP      FN    IDSW
------------------------------------------------------------------------------------------------
ban_vs_gold        0.793   0.783   0.805   0.850   0.976   0.953   0.831       9      18       0
bytetrack_vs_gold  0.709   0.649   0.776   0.846   0.875   0.749   0.823      88      54       2
reid_vs_gold       0.763   0.711   0.820   0.872   0.900   0.792   0.860      91      26       2
reid_vs_ban        0.754   0.704   0.809   0.863   0.902   0.793   0.850      95      21       1

Cổng annotation: ĐẠT {'IDF1': 0.976, 'MOTA': 0.953, 'MOTP': 0.831}

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Trên nhãn của mình cả hai đều cao (MOTA 0.953, IDF1 0.976) vì gần khớp gold và không có lỗi identity (IDSW=0). Ở model thì MOTA **thấp hơn** IDF1: ByteTrack MOTA 0.749 < IDF1 0.875; ReID MOTA 0.792 < IDF1 0.900. Nguyên nhân: MOTA trừ điểm mỗi FP, FN và IDSW với trọng số ngang nhau, nên khi có nhiều FP (88–91) và FN (26–54) thì MOTA bị kéo xuống mạnh, còn chỉ 1–2 lỗi IDSW thì chìm trong đám nhiễu đó — tức MOTA gần như không phạt riêng lỗi identity. Ngược lại, IDF1 chỉ đo chất lượng phân công identity (quan hệ IDTP / IDFP + IDFN) nên rất nhạy với từng lần đổi ID. Kết luận: nếu MOTA cao mà IDF1 thấp thì detection tốt nhưng identity hay bị lộn (nhiều IDSW) — đúng loại lỗi mà bài tracking hôm nay nhắm tới.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

IDF1 tăng 0.875 → 0.900 (+0.025), AssA tăng 0.776 → 0.820 (+0.044), IDSW đều bằng 2 khi so với gold (riêng ReID-vs-bạn là 1). Về fragmented tracks: ByteTrack tách gt4 thành pred track 15/14 và gt5 thành 32/23; ReID chỉ còn tách gt5 (18/17), gt6 (31/24), gt7 (29/39). Về frame sequence của IDSW: ByteTrack đổi ID tại frame 59 (gt4, 14→15) và frame 94 (gt5, 23→32); ReID đổi tại frame 87 (gt5, 17→18) và frame 113 (gt6, 24→31). Nhìn chung treatment giữ identity tốt hơn đôi chút ở các đoạn xe bị che/cắt (bớt một ca tách gt4), nhưng không phải thắng tuyệt đối. Lưu ý quan trọng: ByteTrack ≠ BoT-SORT về implementation, nên con số này là so sánh hai hệ thống, KHÔNG kết luận một mình ReID là nguyên nhân.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA tăng 0.649 → 0.711 (+0.062), FN giảm mạnh 54 → 26 (gần một nửa) trong khi FP gần như không đổi (88 → 91). Điều này cho thấy phần cải thiện chủ yếu đến từ việc detector (YOLO) bắt thêm được xe trước đó bị bỏ sót, chứ không phải từ bước association. Lỗi còn lại là FP — chủ yếu là ghost track không khớp track tham chiếu nào (ByteTrack track 10/41/69/64/70; ReID track 7/27/38/26/28) và các bbox nhỏ/rìa — nên thuộc về detector chứ không phải lỗi gán identity.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Ghost pred track 34 trong `eval_reid_vs_me` với lý do "còn bbox sau khi track tham chiếu 8 đã rời khung", frame 168–170 (length 3). Bạn đã bấm `outside` đúng lúc xe 8 ra khỏi khung nên track 8 của bạn kết thúc, trong khi ReID vẫn vẽ thêm bbox treo sau khi xe biến mất → đây là chỗ bạn đúng, model sai (để box treo sau exit).

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

ReID ghost track 7 (frame 16–116, length 43, "không khớp track tham chiếu nào") cùng các loose boxes quanh frame 104/106/107 (gt6, IoU ≈ 0.52) gợi ý đoạn ~104–116 có xe bị che/cắt mà nhãn của mình có thể bỏ sót hoặc box bị lỏng. Đây không phải bằng chứng chắc chắn mình sai — có thể model là FP — nhưng là tín hiệu đáng soi lại keyframe quanh đó. Nếu nghi ngờ reference thì ghi frame–ID–rule–evidence để Lab Coach đóng closure, không tự sửa reference.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Sửa trong `GUIDELINE_MINI.md`: (1) viết rõ **ngưỡng bấm `outside`** khi xe bị cắt rìa — quy định "khi phần bbox còn lộ nhỏ hơn X pixel ở mép thì coi là rời khung" để hết nhập nhằng kiểu frame 149–151; (2) định nghĩa cụ thể **cách ôm bbox khi bị che một phần** (ôm theo mép nào khi bị chắn ngang/nghiêng) để giảm box lỏng kiểu track 7 frame 111–114; (3) bổ sung luật cho **hai xe cắt/gần nhau** ở vùng rìa (ưu tiên quỹ đạo liên tục để giữ ID).

Đổi quy trình làm việc: ghi lại từng ca khó **ngay lúc gặp** (không chờ xong mới viết guideline); sau mỗi track, tự tua lại 3 lượt (ID → biên → giữa) thay vì để cuối giờ; trước khi chạy model, ghi dự đoán đoạn nào motion/IoU dễ nhầm để đối chiếu với kết quả; khi soi kết quả model, luôn đọc theo thứ tự identity (IDF1/AssA/IDSW) → detector (DetA/FP/FN) → worst frames, và không coi model là đáp án.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json` (SHA-256 `d0cd933d…5697e7`, khóa `2026-09-15 10:37:23 UTC`)
- [ ] `GUIDELINE_MINI.md` đã điền ← đang còn bản mẫu (chứa các ô trống cần điền), chờ hoàn thiện theo các luật nêu ở Mục 6
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md` ← chưa có, chờ peer review (làm cá nhân)
- [x] `reports/REPORT.md` (file này)GUIDELINE_MINI.md