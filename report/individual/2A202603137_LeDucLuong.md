# Báo cáo thực hành Day 3

- Mã học viên theo lớp: 2A202603137
- Ngày / CVAT local: 12/11/2026 / CVAT v2.74.1 local (Docker on WSL2)
- Công cụ đã dùng: CVAT local (Brush, Polygon), MobileSAM (Nuclio interactor)

## 1. Bài đã nộp

Ghi tên ZIP đúng như file trong `submissions/` và số ảnh đã vẽ, Save. Chưa làm hoặc export lỗi thì ghi `chưa có`, không tạo ZIP rỗng. Cột điểm là điểm tối đa của task, **không phải điểm tự chấm**.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | cp1_holes.zip | 1 / 1 | 3 |
| cp2_slice | cp2_slice.zip | 1 / 1 | 3 |
| cp5_occlusion | cp5_occlusion.zip | 1 / 1 | 3 |
| cp3_thin | chưa có | 0 / 1 | 3 |
| cp4_curb | chưa có | 0 / 1 | 3 |
| cp6_coverage | chưa có | 0 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Nếu export lỗi, ghi task, dữ liệu đã Save đến đâu và lỗi đã báo coach.  
*Ghi chú:* Đã hoàn thành trọn vẹn 3 tier chính (easy_semantic, medium_instance, hard_panoptic) và 3 checkpoint instance (cp1_holes, cp2_slice, cp5_occlusion). Ba checkpoint semantic còn lại (cp3, cp4, cp6) chưa kịp hoàn thiện do tập trung thời gian kiểm tra QC và độ chính xác mask cho các tier chính.

## 2. Một quyết định trước khi dùng gợi ý

Chọn object đầu tiên bạn tự vẽ ở `medium_instance`, trước khi xem bất kỳ đề xuất tự động nào cho object đó. Ghi ảnh/vị trí đủ để tìm lại; “quy tắc biên” là lý do bạn chọn hoặc dừng mask ở ranh đó.

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: Ảnh `000000181542.jpg`, vị trí chính giữa ảnh (đang đi bộ trên vạch kẻ đường qua đường), đối tượng là cô gái mặc áo dài màu sáng.
- Class và quy tắc tôi dùng để chọn biên: Class `person`. Quy tắc biên: Chỉ lấy vùng pixel nhìn thấy thực tế của người và trang phục/phụ kiện gắn liền (tóc, khuôn mặt, tà áo dài, túi xách đeo bên người chạm sát cơ thể và giày dép). Dừng biên chính xác tại mép tà áo tiếp xúc với mặt đường và bánh xe máy phía sau, không vẽ lấn ra bóng đổ trên mặt đường hay vẽ phỏng đoán phần bánh xe bị tà áo che.
- Nếu dùng gợi ý sau đó: Khi dùng gợi ý tự động (SAM) cho chiếc xe máy của người đàn ông bên trái cô gái, model có xu hướng lan rộng ăn vào chân người lái xe và bóng đổ dưới gầm bánh xe. Hành động: Đặt thêm điểm Negative point vào chân người lái và dùng công cụ Eraser của Brush gọt sạch phần bóng đổ dưới lốp xe chạm mặt đường để tách biệt rõ ràng giữa `person` và `motorcycle`.
- Nếu không dùng gợi ý: ghi “không dùng”; vẫn giải thích một quyết định gán nhãn của mình.

## 3. Một lỗi tôi tìm thấy và sửa

Chọn một lỗi **có thật** trong bài. Nếu công cụ lỗi khiến bạn chưa sửa được, ghi rõ đã thử gì và cần coach hỗ trợ gì; không ghi “đã sửa” khi chưa sửa.

- Task/ảnh/vùng: Task `medium_instance` / ảnh `000000181542.jpg` / khu vực chiếc xe máy và người lái xe bên trái.
- Lỗi thuộc loại: Gộp-tách vật (Merging error).
- Bằng chứng tôi nhìn thấy: Đề xuất tự động của AI ban đầu gộp cả người lái xe máy (đầu, thân, chân) và thân chiếc xe máy thành một khối mask duy nhất có nhãn `motorcycle`.
- Quy tắc và hành động sửa: Quy tắc instance segmentation yêu cầu tách riêng người và phương tiện (`person` và `motorcycle` là 2 instance độc lập). Tôi đã dùng Brush/Eraser tách phần thân người lái ra khỏi mask xe máy, tạo một mask mới gán nhãn `person` cho người lái, đồng thời giữ nguyên chiếc xe máy thành một instance `motorcycle` duy nhất dù bị chân người lái che khuất chia thành hai mảng nhìn thấy tách rời.
- Sau sửa đã Save và export lại chưa? Đã Save và export lại thành công file `medium_instance.zip` (định dạng COCO 1.0).

Nếu bạn **đã xem Summary tự đánh giá trên GitHub Actions hoặc tự chạy script**, ghi ngắn một kết quả liên quan lỗi vừa sửa (ví dụ task, metric trước/sau nếu có): Chạy `python scripts/inspect_submissions.py --dir submissions` kiểm tra xác nhận `[OK] medium_instance: medium_instance.zip` với 41 annotations dạng RLE hợp lệ.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

Mỗi ca là một **vùng cụ thể** khiến bạn phải cân nhắc hai cách hiểu. Ghi dấu hiệu nhìn thấy hoặc quy tắc đã dùng, rồi nêu quyết định hoặc câu hỏi cho coach. Không cần ba lỗi; ca đã quyết định được cũng hợp lệ.

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| 1. Ảnh `000000181542.jpg`: Hình ảnh người trên biển quảng cáo tường cửa hiệu Chanel (hậu cảnh). | Đánh nhãn `person` vì có hình dạng người rõ ràng VS Không đánh nhãn vì chỉ là tranh vẽ trên tường. | Quy tắc COCO và quy tắc task chỉ gán nhãn người thật trong bối cảnh giao thông, không gán nhãn hình ảnh trên áp phích, biển quảng cáo hay ma-nơ-canh. | Quyết định: Không đánh nhãn `person` cho biển quảng cáo này để tránh lỗi False Positive. |
| 2. Ảnh `000000181542.jpg`: Khu vực nan hoa / căm xe của bánh xe máy. | Đục/khoét từng khe hở nhỏ li ti giữa các nan hoa căm xe để lộ nền đường VS Tô kín toàn bộ hình tròn bánh xe. | Quy tắc tại `guideline-mini-sheet.md` và tiêu chí trạm `cp1_holes`: *"Holes: windows/gaps stay inside the mask — do NOT cut them out"*. | Quyết định: Tô kín toàn bộ diện tích bánh xe máy, chỉ cạo bỏ bóng đổ dưới lốp xe chạm mặt đường, không khoét lỗ nan hoa để tránh làm rách nát mask. |
| 3. Ảnh `000000181542.jpg`: Chiếc xe bán tải (pickup truck) màu bạc ở hậu cảnh giữa ngã tư. | Đánh nhãn `car` VS Đánh nhãn `truck`. | Xe có buồng lái phía trước và thùng chở hàng hở phía sau đặc trưng của xe bán tải; schema task có nhãn `truck` riêng biệt với `car`. | Quyết định: Đánh nhãn `truck` theo đúng công năng chở hàng; đối chiếu với schema của task để phân loại nhất quán. |
