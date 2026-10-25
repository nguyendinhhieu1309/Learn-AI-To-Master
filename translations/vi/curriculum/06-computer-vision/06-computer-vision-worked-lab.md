# Lab computer vision có hướng dẫn: ảnh, nhãn, metric và phân tích lỗi

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/06-computer-vision/06-computer-vision-worked-lab.md)

Computer vision không chỉ là vấn đề architecture. Decode ảnh, color space, resize, biến đổi label, leakage giữa patient/scene và evaluation theo tác vụ có thể thay đổi kết quả nhiều hơn backbone sâu hơn.

## Mục tiêu học tập

- Thiết kế image-data pipeline có thể tái tạo.
- Giữ ảnh và geometric label đồng bộ qua transform.
- Chọn metric cho classification, detection và segmentation.
- Phân tích lỗi hình ảnh và so với baseline đơn giản.

## Điều kiện tiên quyết

Hoàn thành [image pipeline](05-image-pipelines-and-vision-training-workflow.md) và [detection, segmentation](04-detection-segmentation-and-error-analysis.md).

## 1. Viết image contract

Ghi trước khi training:

- một ảnh đại diện đơn vị gì: frame, patient, sản phẩm, camera hay scene;
- channel order, numeric range, color space và orientation;
- quy ước width/height của ảnh;
- annotation format và tọa độ absolute hay normalized;
- event/entity dùng để group train/validation/test;
- target metric và operating point deployment.

File nạp được chưa chắc đúng: EXIF rotation, nhầm BGR/RGB, đơn vị không nhất quán hoặc resize mask bằng interpolation làm mượt có thể làm hỏng label mà không báo lỗi.

## 2. Biến đổi ảnh và label cùng nhau

Nếu flip ảnh ngang, bounding box và segmentation mask cũng phải flip. Crop bỏ pixel thì box cần clip hoặc xóa theo quy tắc được ghi lại. Transform ảnh mà không đổi label tạo supervision sai.

Với horizontal flip ảnh rộng W và tọa độ continuous theo pixel-edge, box $(x_{\min},y_{\min},x_{\max},y_{\max})$ chuyển thành:

$$
(W-x_{\max},\,y_{\min},\,W-x_{\min},\,y_{\max}).
$$

Quy ước pixel-index có thể dùng $W-1$. Điều quan trọng là nhất quán với convention annotation.

Dùng interpolation mượt cho intensity ảnh tự nhiên nếu phù hợp. Dùng nearest-neighbor cho categorical mask để class ID không bị thay bằng giá trị phân số.

## 3. Split theo đơn vị độc lập thật sự

Nếu frame đến từ cùng video, split theo video hoặc capture session. Nếu ảnh cùng patient, split theo patient. Nếu mục tiêu là performance tương lai, split theo thời gian. Chia ngẫu nhiên frame kề nhau có thể tạo cảnh gần như trùng trong cả train lẫn test.

Kiểm tra:

- duplicate theo filename/hash;
- ảnh trùng gần đúng;
- class count theo split;
- camera, site, patient, video hoặc scene chồng lấn giữa source;
- annotation density và tần suất label rỗng.

## 4. Metric trả lời các câu hỏi khác nhau

### Classification

Accuracy hữu ích khi class và chi phí lỗi cân bằng. Với dữ liệu mất cân bằng, thêm precision/recall từng class, confusion matrix và threshold đã dùng. Nếu score dùng ra quyết định, đánh giá probability calibration.

### Object detection

Với box dự đoán A và tham chiếu B:

$$
\operatorname{IoU}(A,B)=\frac{|A\cap B|}{|A\cup B|}.
$$

Nếu box A phủ $[0,10]\times[0,10]$, B phủ $[5,15]\times[0,10]$, diện tích intersection là 50, union là $100+100-50=150$, nên IoU bằng $1/3$.

Average precision tóm tắt precision-recall sau khi ghép prediction với ground-truth object. Nêu IoU matching rule, cách average class, xử lý confidence và evaluation dùng một threshold hay một dải. Detection count không có matching rule thì không thể tái lập.

Non-maximum suppression bỏ box có score thấp hơn khi overlap vượt threshold đã chọn. Nó giảm detection trùng nhưng có thể suppression object hợp lệ nằm sát nhau. Threshold là một phần của hệ thống được đánh giá.

### Segmentation

Với một class, Dice và IoU là:

$$
\operatorname{Dice}=\frac{2|P\cap G|}{|P|+|G|},
\qquad
\operatorname{IoU}=\frac{|P\cap G|}{|P\cup G|}.
$$

Dice nhấn overlap và có thể cao với vùng lớn dù cấu trúc nhỏ bị bỏ sót. Báo điểm theo class và giải thích cách xử lý mask rỗng.

## 5. Baseline và transfer learning

Bắt đầu với baseline đơn giản, tái lập được:

1. xác minh label alignment bằng overlay trực quan;
2. train model nhỏ hoặc dùng pretrained representation đã đóng băng;
3. thiết lập validation metric và nhóm lỗi;
4. chỉ fine-tune sau khi đã tin pipeline input và split.

Pretrained weight mang theo giả định về image domain, color channel, resolution và normalization. Ghi checkpoint source và preprocessing contract. Không so pretrained model với baseline nhận ảnh evaluation khác.

## 6. Phiếu error analysis

Với mỗi lỗi, lưu:

- image identifier và split;
- ground truth cùng prediction overlay;
- confidence hoặc score;
- loại lỗi: bỏ object, duplicate, sai class, boundary, domain shift, label hỏng;
- human reviewer có đồng ý với annotation không;
- vấn đề có thể sửa bằng data, model, threshold hay product design không.

Review ví dụ trên toàn bộ validation set và theo slice quan trọng. Chỉ xem lỗi gây ấn tượng nhất tạo selection bias.

## Thực hành

1. Segmentation mask dùng ID 0,1,2. Vì sao nên tránh smooth resize cho mask?
2. Ảnh lấy mỗi frame thứ mười từ 20 video. Target là performance trên video tương lai. Cần split thế nào?
3. Precision detector tăng sau khi nâng confidence threshold nhưng recall giảm. Model có nhất thiết tốt hơn không?
4. Classifier accuracy tổng cao nhưng gần như bỏ mọi positive. Metric nào phát hiện vấn đề?
5. Pretrained model và baseline dùng normalization ảnh khác nhau. Vì sao so sánh không hợp lệ?

### Gợi ý đáp án

1. Interpolation có thể tạo class ID phân số hoặc không có thật; dùng phương pháp giữ nguyên nhãn.
2. Split theo video, mọi frame của video nằm trong một partition; dùng protocol theo thời gian nếu kết luận nhắm tới giai đoạn tương lai.
3. Không. Operating point đã đổi. So toàn bộ precision-recall và chọn threshold theo chi phí dự định.
4. Positive-class recall, precision, confusion matrix và class support.
5. Input distribution khác nên performance trộn lẫn chất lượng model với preprocessing.

## Mini-project

Train/adapt image classifier trên dataset nhỏ có license. Nộp group-aware split, baseline đơn giản, visual label audit, validation protocol cố định, metric theo class, 12 ví dụ lỗi đã review và inference contract. Nếu chọn detection/segmentation, giải thích matching rule hoặc cách xử lý mask rỗng.

## Lỗi thường gặp

- Chia ngẫu nhiên frame video kề nhau.
- Biến đổi ảnh nhưng không biến đổi box/mask.
- Chỉ báo accuracy hoặc mAP tổng mà thiếu protocol.
- Coi pretrained normalization là chi tiết implementation.
- Sửa label khó sau khi xem test performance.
