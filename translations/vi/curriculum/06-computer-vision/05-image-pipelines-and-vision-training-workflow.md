# Image pipeline và workflow training vision

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/06-computer-vision/05-image-pipelines-and-vision-training-workflow.md)

Vision model có thể thất bại do decode ảnh, tọa độ, channel màu hay thiết kế split từ rất lâu trước khi architecture là giới hạn.

## 1. Biểu diễn ảnh

Image tensor thường có trục height, width và channel. Library khác nhau giữa layout HWC và CHW, thứ tự channel RGB và BGR. Pixel có thể là integer trong $[0,255]$ hoặc float trong $[0,1]$. Normalization constant cần khớp pretrained model hoặc cấu hình training.

Kiểm tra ảnh đã decode sau mỗi transform. Xem orientation metadata, alpha channel, grayscale, file hỏng và mapping label-tên file. Lưu tập audit trực quan nhỏ, vẽ box/mask/keypoint lên ảnh.

## 2. Resize, crop và tọa độ

Resize làm đổi pixel coordinate và scale object. Crop đổi gốc tọa độ và có thể cắt object. Letterboxing giữ aspect ratio nhưng thêm padding. Với box, áp dụng cùng geometric transform cho cả bốn góc rồi tính lại bounds. Với mask, dùng nearest-neighbor interpolation.

Ghi rõ tọa độ chuẩn hóa dùng image width/height hay width-minus-one/height-minus-one; box max inclusive hay exclusive. Lệch quy ước chỉ một pixel có thể ảnh hưởng đáng kể evaluation object nhỏ.

## 3. Chia data cho vision

Split theo source độc lập: patient, video, scene, camera, acquisition session hoặc geographic site. Frame video kề nhau và nhiều crop từ cùng nguồn có tương quan. Giữ mọi dữ liệu liên quan trong cùng partition. Với deployment theo thời gian, train giai đoạn sớm, validate/test giai đoạn muộn.

## 4. Augmentation có ý nghĩa vật lý

Flip ngang, crop, color jitter, blur và noise có thể tăng robustness nếu chúng phản ánh variation deployment hợp lý. Tránh transform đổi nghĩa semantic: lật biển báo, laterality y tế, text hoặc hướng robot có thể tạo training example sai. Với detection, theo dõi box trở nên quá nhỏ hay biến mất sau crop.

## 5. Workflow transfer learning

Bắt đầu từ pretrained backbone khi task và image domain có liên quan hợp lý. Thay task head, nếu cần đóng băng backbone thời gian ngắn, sau đó fine-tune với learning rate thấp hơn. Đảm bảo normalization layer ở mode dự định. Chỉ so với training từ đầu nếu data và compute đủ lý do.

## 6. Đánh giá bằng error slice

Báo benchmark metric và cấu hình chính xác, rồi kiểm tra ví dụ theo object size, occlusion, ánh sáng, camera, địa lý và class. Đo end-to-end latency ở resolution đích. Model đạt mAP offline vẫn có thể thất bại trên edge hardware do memory, frame rate hoặc giới hạn nhiệt.

## 7. Thí nghiệm ảnh tái lập được

Lưu dataset version, file hash hoặc ID ổn định, split ID, resize policy, augmentation config, class map, normalization, model checkpoint và evaluation setting. Trực quan hóa cùng ví dụ giữa các model version. Khóa test data trong lúc tune architecture và threshold.

## 8. Bài tập

1. Pretrained model đòi RGB CHW float nhưng nhận BGR HWC byte. Dự kiến triệu chứng và cần kiểm tra gì?
2. Vì sao không chia ngẫu nhiên frame gần nhau từ cùng video vào train và test?
3. Flip ngang cải thiện validation accuracy trên nhận dạng chữ viết tay. Cần kiểm tra vấn đề gì trước khi chấp nhận?
4. Vì sao mask phải resize bằng nearest-neighbor interpolation?

### Gợi ý đáp án

1. Channel/scale sai có thể làm prediction kém; kiểm tra shape, thứ tự màu, range, normalization và trực quan hóa sau transform.
2. Frame gần nhau gần duplicate, khiến validation score lạc quan.
3. Flip có thể đổi nghĩa ký tự/nhãn; kiểm tra label có còn hợp lệ không và dữ liệu deployment có chứa dạng lật đó không.
4. Giữ nguyên class ID rời rạc, tránh tạo nhãn phân số.
