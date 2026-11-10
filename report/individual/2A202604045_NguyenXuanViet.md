# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Xuân Việt
- MSSV: 2A202604045
- Email: viet.ngxuan@gmail.com
- Lớp: K5-L2B
- Link repo bài làm: https://github.com/ngxuanviet/K5-L2B-DAY02-NguyenXuanViet-2A202604045-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/github/ngxuanviet/K5-L2B-DAY02-NguyenXuanViet-2A202604045-ObjectDetection/blob/main/notebooks/day2_annotation_to_yolo.ipynb

## 2. Schema và mini guideline

Chi tiết ở [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Schema bốn lớp `car/truck/bus/van`; không gán mô tô, người, biển báo. Ba ca mơ hồ đã quyết định:

| Ảnh | Tình huống | Quyết định |
|---|---|---|
| `drive_014` | Xe kéo cứu hộ có khung kéo, cabin nhỏ | gán `truck` |
| `drive_015` | Chỉ thấy mặt trước, mờ, không phân biệt `car`/`van` | không gán |
| `drive_008` | Vật thể ở rìa ảnh, không đủ chi tiết | không gán |

Phần cần làm rõ: ranh giới `bus`/`van` (xe khách nhỏ, minivan) và `truck`/`van` (xe tải nhỏ), cùng việc có gán xe đỗ ở khu vực phụ/xe nền rất nhỏ hay không (xem mục 4).

## 3. Annotation và self-QC

- Số file nhãn train: 50/50 (`dataset/labels/train/drive_001.txt` … `drive_050.txt`).
- Kết quả checker: `python tools/check_yolo_labels.py` → "ĐẠT cấu trúc: 50/50 file nhãn hợp lệ".
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: **[CẦN ĐIỀN: liệt kê ≥10 ảnh bạn thực sự đã mở rà]**
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: **[CẦN ĐIỀN theo thực tế]**
- Commit khóa nhãn ban đầu: `be4e046` ("dan nhan xong", 2026-11-10).

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 421 | không rework |
| FP | 440 | không rework |
| FN | 232 | không rework |
| Precision | 0.489 | không rework |
| Recall | 0.6447 | không rework |
| F1 | 0.5561 | không rework |
| mean_iou_of_matches | 0.8477 | không rework |
| passed | false (yêu cầu F1 ≥ 0.80, mean IoU ≥ 0.70) | không rework |

Kết quả nằm ở `outputs/annotation_evaluation.json`. **Không rework**: F1 chưa đạt, nhưng học viên được phép bỏ qua bước sửa nhãn để đi tiếp train/eval; nhãn nộp là nhãn ban đầu (`be4e046`). Vì vậy phần pipeline YOLO bên dưới chịu ảnh hưởng của nhãn chưa đạt chuẩn.

Phân tích sau khi có gold (đối chiếu từng box với gold và vẽ chồng nhãn lên ảnh `drive_001`, `009`, `018`, `041`):

- **Box khít, không phải vấn đề**: mean IoU 0.85 đạt ngưỡng 0.70; trung vị kích thước box so với gold lệch dưới 5%.
- **Nhầm lớp: 113 box** (cùng vị trí, IoU ≥ 0.5, khác ID): `bus` gán cho `van` gold 52; `van` gán cho `truck` gold 21; `truck` gán cho `bus` gold 18; `car` gán cho `truck` gold 14; `van` gán cho `car` gold 5. Tổng `bus` của tôi là 93 so với 20 ở gold; `truck` là 31 so với 46. Nguyên nhân khả dĩ là định nghĩa `bus`/`van`/`truck` trong guideline chưa đủ rõ cho xe cỡ vừa.
- **Box thừa: 327 box** không có box gold nào gần đó, 263 là `car`, diện tích trung vị ≈ 0.003 (khoảng 35×35 px). Qua các ảnh đã xem, đó là xe đỗ ở khu vực phụ hoặc làn bên kia dải phân cách, xe nền rất xa, và vài vật không phải xe (chân cột ở `drive_018`, biển báo ở `drive_041`). Gold dường như chỉ nhãn xe ở các làn chính; đây là suy luận từ 4 ảnh, README không ghi quy tắc này.
- **Bỏ sót: 119 box**, chủ yếu `car` nhỏ (112/119) mà gold có.
- **Ca mơ hồ**: gold có nhãn ở `drive_015` (12 box so với 9 của tôi) và `drive_008`, tức việc bỏ qua xe mờ làm giảm recall.

Hướng sửa nếu làm rework: sửa 113 box nhầm lớp (F1 ước tính ≈ 0.71), rồi bỏ xe ngoài làn chính/vật không phải xe (cần cắt khoảng 200 box thừa để chạm 0.80).

## 5. Kết quả YOLO

- Runtime: Google Colab, GPU (`device: "0"`), train xong 50 epoch trong ≈ 77 giây. Phiên bản Ultralytics/Torch: **[CẦN ĐIỀN: xem dòng đầu log cell cài đặt/train trên Colab]**.
- Tham số: `model=yolo26n.pt`, `epochs=50`, `imgsz=640`, `batch=-1` (auto), `seed=20260909`, `patience=100` (xem `args.yaml`). Không đổi so với notebook gốc.
- Best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt` (không đưa file `.pt` vào repo). Theo `results.csv`, best rơi vào epoch 2 (mAP50 0.272, mAP50–95 0.199); các epoch sau dao động 0.17–0.26 mà không vượt được.

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.409 |
| Recall | 0.234 |
| mAP50 | 0.271 |
| mAP50–95 | 0.200 |

Nguồn: [model_eval_metrics.json](assets/model_eval_metrics.json). Ảnh dự đoán: [eval_predictions/visuals](assets/eval_predictions/visuals/); log và đồ thị: [vehicle_annotation](assets/yolo26_runs/vehicle_annotation/) (có `results.csv`), [vehicle_eval](assets/yolo26_runs/vehicle_eval/) (có [confusion_matrix.png](assets/yolo26_runs/vehicle_eval/confusion_matrix.png) và [val_batch0_labels.jpg](assets/yolo26_runs/vehicle_eval/val_batch0_labels.jpg) / [val_batch0_pred.jpg](assets/yolo26_runs/vehicle_eval/val_batch0_pred.jpg)).

Ghi chú khi đọc confusion matrix: hàng là Predicted, cột là True. Cột/hàng `background` chứa hàng nghìn dự đoán độ tin cậy rất thấp nên bị phóng đại; chỉ đọc đáng tin phần các lớp thật. Theo cột True: `car` 94 box (84 đúng, 3 bị dự đoán `van`, 2 `bus`, 5 bỏ sót), `truck` 5 box (1 đúng, 4 bị dự đoán `car`), `bus` 2 box (1 đúng, 1 thành `car`), `van` 12 box (**0 đúng**, 11 bị dự đoán `bus`, 1 bỏ sót).

Trả lời bốn câu hỏi cuối notebook:

1. **Lớp recall thấp nhất:** `van`, recall 0/12. Ví dụ `drive_052`: chiếc xe buýt hai tầng ở giữa ảnh có nhãn eval là `van` (dòng `3 0.427 0.534 0.207 0.655` trong `dataset/labels/eval/drive_052.txt`) nhưng model dự đoán `bus` 0.49; `drive_053`: xe buýt lớn màu trắng-xanh cũng có nhãn `van`, model dự đoán `bus` 0.85. Bằng mắt, cả hai xe đều là xe buýt nên dự đoán của model hợp lý theo định nghĩa `bus` của schema; nó chỉ bị tính là bỏ sót `van` vì nhãn gold ghi `van`. Tiếp theo là `truck` (1/5, 4 box bị gọi là `car`, ví dụ `drive_059` phía trên bên trái). `bus` chỉ có 2 box trong eval nên con số này ít tin cậy.
2. **mAP50 cao nhưng mAP50–95 thấp** nghĩa là model tìm đúng vật nhưng box không khít ở IoU cao, nên cần kiểm tra độ chặt cạnh box, box lấn nền hoặc gộp xe, xe bị che/cắt rìa, và nhãn eval có cùng chuẩn với nhãn train hay không. Trường hợp của tôi khác: mAP50 0.271 và mAP50–95 0.200 chỉ chênh khoảng 26%, và mean IoU của nhãn train với gold là 0.85, nên box không phải điểm nghẽn chính. Mức mAP thấp đến chủ yếu từ lớp (nhầm lớp và recall thấp) và từ việc chỉ train 50 ảnh.
3. **Lỗi class liên quan guideline:** toàn bộ 12 box `van` của eval bị dự đoán thành `bus` (11) hoặc bỏ sót (1). Dữ kiện hỗ trợ: khi chấm nhãn train với gold (mục 4), tôi đã gán `bus` cho 52 box mà gold ghi `van`; model học đúng thói quen đó của tôi, nên lặp lại lỗi trên eval. Cần phân biệt hai trường hợp. Với xe buýt lớn như `drive_052`, `drive_053`, định nghĩa `bus` trong `GUIDELINE_MINI.md` ("xe buýt thành phố, xe khách") khớp schema của README, nhưng gold lại ghi `van`; đây là chỗ nhãn gold và schema lệch nhau, guideline của tôi không sai theo chữ nhưng chưa ghi lại ca mơ hồ `bus`/`van` này để hỏi giảng viên (như chính guideline đã tự đặt ra). Với xe con bị dự đoán `van` ở `drive_056`, `drive_060`, ranh giới `car`/`van` mới thực sự chưa rõ. Tương tự, 14 box tôi gán `car` mà gold là `truck` giải thích vì sao 4/5 `truck` eval bị gọi là `car`.
4. **Vì sao không train bằng 10 ảnh eval:** nếu train trên eval thì chỉ số đo khả năng ghi nhớ chứ không phải tổng quát hóa, và bị thổi phồng. Trong notebook, 10 ảnh này đóng vai validation (chọn best checkpoint theo mAP mỗi epoch) và đánh giá cuối. Hệ quả: vì val và eval là cùng một tập, chỉ số cuối không hoàn toàn độc lập, và với 10 ảnh thì chỉ số dao động lớn giữa các epoch (mAP50 0.17–0.27), nên best epoch 2 chủ yếu là may rủi.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Gold QC | F1 0.556 < 0.80 | Phân tích FP/FN theo lớp và kích thước, vẽ chồng ảnh | Nguyên nhân chính: nhầm lớp và box thừa xe nhỏ; không rework do được phép bỏ qua |
| Colab train/eval | mAP thấp (mAP50 0.27, recall 0.23) | Rà confusion matrix và ảnh dự đoán | Nguyên nhân chính: `van` bị gọi `bus`, `truck` bị gọi `car`, và nhãn train chưa đạt gold; chưa train lại do không rework |

## 7. Tự đánh giá theo rubric

- Định dạng YOLO: có, checker đạt 50/50.
- Đầy đủ vật thể / phân lớp: chưa đạt theo gold (recall 0.64, 113 box nhầm lớp).
- Độ bám sát box: tốt (mean IoU 0.85).
- Mini guideline: đã điền định nghĩa và 3 ca mơ hồ.
- Gold QC: **chưa đạt**, không rework (được phép bỏ qua).
- Pipeline YOLO: Colab chạy xong, có metrics, ảnh eval và nhận xét lỗi (mục 5); không dùng `eval` để train. mAP thấp, đã phân tích nguyên nhân. Chưa ghi được phiên bản Ultralytics/Torch.

## 8. Công cụ AI và tài liệu tham khảo

- Dùng Claude Code để đối chiếu nhãn với gold (script phân tích lỗi lớp/kích thước) và soạn báo cáo này; các số liệu mục 4 lấy từ `tools/evaluate_annotations.py` và script đối chiếu, tôi đã đọc lại kết quả. **[Bổ sung nếu bạn dùng AI/nguồn khác ở bước khác]**
- Nguồn ngoài repo: không. Không đính kèm gold train, token hay mật khẩu.
